+++
date = '2025-05-21T15:03:15-07:00'
draft = false
title = 'Rate Limited Logging'
+++

I want to log messages under exceptional situations. If for some unknown reason, the exceptional situation becomes the common case, I don't
want to flood the logs. So, I want a rate limited logging at the individual log message level.

```rust
#[macro_export]
macro_rules! info_every {
    // Count-based with default 1-minute window
    // Convenience pattern that delegates to the full pattern with a preset window
    // This allows simpler syntax when the standard 1-minute window is sufficient
    ($n:expr, $($arg:tt)+) => {{
        // Delegate to the full pattern with the default 1-minute (60 seconds) duration
        info_every!($n, std::time::Duration::from_secs(60), $($arg)+)
    }};

    // Count-based with counter reset at the end of time window.
    ($n:expr, $duration:expr, $($arg:tt)+) => {{
        // Import the necessary atomic types for thread-safe counters and time tracking
        // Atomics are used to ensure thread-safety without locks, even in potentially single-threaded
        // applications.
        use std::sync::atomic::{AtomicU32, AtomicU64, Ordering};
        use std::time::{Duration, Instant};
        use std::sync::Once;

        // Static atomic variables to persist across macro invocations
        // IMPORTANT: These static variables are unique for each macro invocation site in your code.
        // This means each place where info_every!() is called gets its own independent:
        // - counter (different log counts for different log messages)
        // - time window (separate rate limiting for each log point)
        // - its own set of static variables to track these values
        //
        // COUNTER: Tracks how many times this macro has been called at this specific invocation site
        static COUNTER: AtomicU32 = AtomicU32::new(0);
        // TIME_WINDOW_START: Records when the current time window started for this specific invocation site
        static TIME_WINDOW_START: AtomicU64 = AtomicU64::new(0);
        // MONOTONIC_ANCHOR: Reference point for monotonic timing for this specific invocation site
        static mut MONOTONIC_ANCHOR: Option<Instant> = None;
        // INIT: Ensures one-time initialization of MONOTONIC_ANCHOR for this specific invocation site
        static INIT: Once = Once::new();

        // Advantages of using monotonic time (Instant) over system time (SystemTime):
        // 1. Accuracy: Monotonic clocks are specifically designed for measuring elapsed time
        //    and are not affected by system clock adjustments (NTP updates, daylight savings, etc.)
        // 2. Performance: Accessing monotonic time is generally faster as it doesn't require
        //    the same system calls as getting wall-clock time
        // 3. Immunity to time changes: The code continues working correctly even if the
        //    system time changes while the program is running
        // 4. No time-travel: Unlike system time which can jump backward or forward,
        //    monotonic time always increases, avoiding rate-limiting bugs when system time changes

        // Initialize the monotonic anchor point (thread-safe, one-time initialization)
        // This creates a reference point that persists for the lifetime of this particular
        // invocation site's static variable - each place where info_every!() appears in
        // your code will have its own MONOTONIC_ANCHOR with its own reference time
        INIT.call_once(|| {
            unsafe {
                MONOTONIC_ANCHOR = Some(Instant::now());
            }
        });

        // Get current time in seconds since our monotonic anchor
        // Using monotonic time ensures we get reliable elapsed time measurements
        // even if the system clock is changed while the program is running
        let now = unsafe {
            MONOTONIC_ANCHOR.unwrap().elapsed().as_secs()
        };

        // Note on memory ordering (Ordering::Relaxed):
        // We use Relaxed ordering since:
        // 1. For logging, we can tolerate small inconsistencies that don't affect program correctness
        // 2. We don't need to synchronize other memory operations with these counters
        // 3. Even in single-threaded applications, using atomics provides future-proofing
        //    if the code is later used in a multi-threaded context
        // 4. The performance cost of Relaxed atomic operations is minimal even in single-threaded code

        // Parse input parameters
        let n = $n as u32;               // Count threshold
        let window = $duration.as_secs(); // Time window in seconds
        let window_start = TIME_WINDOW_START.load(Ordering::Relaxed); // Time window start timestamp

        // Check if the time window has elapsed since the window start time
        let time_window_elapsed = now >= window_start + window;

        // Reset the counter and update the window start timestamp if the time window has elapsed
        if time_window_elapsed {
            // Reset counter to 0 since we're starting a new time window
            COUNTER.store(0, Ordering::Relaxed);

            // Update the time window start timestamp to mark the beginning of the new window
            // We use store instead of compare_exchange since we've already determined
            // the time window has elapsed and we want to update unconditionally
            TIME_WINDOW_START.store(now, Ordering::Relaxed);
        }

        // Increment counter atomically and get previous value before the increment
        // fetch_add returns the previous value, then adds 1
        let count = COUNTER.fetch_add(1, Ordering::Relaxed);

        // Log only when count threshold is reached:
        // - First call after counter reset: count=0, so 0%n=0 (logs immediately)
        // - Then logs when count=n, 2n, 3n, etc. within the time window
        if count % n == 0 {
            // Log the message with metadata about the rate limiting
            // Note: We've already updated the time window start if needed,
            // and we don't need to update it again when logging due to count threshold
            info!(
                occurences = %count,
                info_every = %n,
                time_window = %window,
                $($arg)*
            );
        }
    }};
}

// Usage examples:
// The semicolon in the examples is NOT required by this macro, but is one way to use
// the structured logging syntax in Rust logging libraries like tracing or log:
//
// Rust logging macros support two main syntax styles for structured fields:
//
// 1. Semicolon style (used in our examples):
//    info!("Main message"; field1 = value1, field2 = ?value2)
//    - Message comes first, followed by semicolon, then key-value pairs
//
// 2. Direct style (also supported):
//    info!(?variable1, %variable2, "Main message")
//    - Fields can come before the message
//    - ?variable includes both the name and debug-formatted value
//    - %variable includes both the name and display-formatted value
//
// Both styles are valid and supported by this macro. The $($arg:tt)+ in the macro
// definition captures all tokens that follow the count/duration parameters and
// passes them directly to the underlying info! macro.

// With explicit duration:
info_every!(
    100,                        // Count threshold (log every 100 occurrences)
    Duration::from_secs(60),    // Time window (reset counter after 60 seconds)
    "Processing batch";         // Main log message (followed by semicolon)
    batch_id = ?id              // Structured field (key-value pair)
);

// With default 1-minute duration:
info_every!(
    100,                        // Count threshold (log every 100 occurrences)
    "Processing batch";         // Main log message (followed by semicolon)
    batch_id = ?id              // Structured field (key-value pair)
);

// Direct style (without semicolon) - also supported:
info_every!(
    100,                        // Count threshold (log every 100 occurrences)
    ?id,                        // Debug-formatted value (includes variable name)
    %count,                     // Display-formatted value (includes variable name)
    "Processing batch"          // Main log message
);

// Understanding multiple invocation sites:
//
// When you use info_every!() in multiple places in your code, each gets its own set of
// static variables:
//
// // In file1.rs - counts & rate-limits this log point independently
// info_every!(100, "Processing orders"; customer_id = ?id);
//
// // In file2.rs - counts & rate-limits this log point independently
// info_every!(50, "Database query completed"; rows = ?count);
//
// Each of these macro expansions will create its own static COUNTER and TIME_WINDOW_START
// variables with unique symbol names. This ensures each log point is independently rate-limited
// based on its own invocation count and timing.
```
