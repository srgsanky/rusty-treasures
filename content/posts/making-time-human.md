+++
date = '2025-05-04T12:50:10-07:00'
draft = false
title = 'Making Time Human'
+++


I can't read epochs and instantly know what year they represent. I can't look at raw seconds and tell how far they are from now.

|                     | Input                 | Output                           |
| ------------------- | --------------------- | -------------------------------- |
| Seconds since epoch | `1746388621`          | `2025-05-04T19:57:01.000000000Z` |
| Duration            | `std::time::Duration` | `1h 5s`                          |
| Time delta from now | `chrono::TimeDelta`   | `1h 5s`                          |

# Convert seconds since epoch to human readable format

Prerequisite: Use chrono crate <https://crates.io/crates/chrono>

```bash
cargo add chrono
```

```rs
use chrono::{DateTime, Utc, Local, TimeZone};

let seconds_since_epoch: i64 = 1746388621;
let utc_time = Utc.timestamp_opt(seconds_since_epoch, 0).unwrap();
let local_time: DateTime<Local> = DateTime::from(utc_time);

println!(
    "{}: {}",
    seconds_since_epoch,
    local_time.format("%Y-%m-%dT%H:%M:%S.%fZ")
);
```

[See it in rust playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2024&gist=a358a4953d05536e3fba6c0cbdee5de9).

References:
* <https://docs.rs/chrono/latest/chrono/trait.TimeZone.html#method.timestamp_opt>
* <https://docs.rs/chrono/latest/chrono/format/strftime/index.html>

# Convert `std::time::Duration` to human readable format


```rs
use std::time::Duration;

fn format_duration(duration: Duration) -> String {
    let total_seconds = duration.as_secs();

    if total_seconds == 0 {
        return "0s".to_string();
    }

    // Define thresholds for different time units in seconds
    const SECOND: u64 = 1;
    const MINUTE: u64 = 60;
    const HOUR: u64 = 60 * MINUTE;
    const DAY: u64 = 24 * HOUR;
    const WEEK: u64 = 7 * DAY;
    const MONTH: u64 = 30 * DAY; // Approximation
    const YEAR: u64 = 365 * DAY; // Approximation

    let mut result = String::new();
    let mut remainder = total_seconds;

    // Process each unit from largest to smallest
    if remainder >= YEAR {
        let years = remainder / YEAR;
        remainder %= YEAR;
        result.push_str(&format!("{}y ", years));
    }

    if remainder >= MONTH {
        let months = remainder / MONTH;
        remainder %= MONTH;
        result.push_str(&format!("{}mo ", months));
    }

    if remainder >= WEEK {
        let weeks = remainder / WEEK;
        remainder %= WEEK;
        result.push_str(&format!("{}w ", weeks));
    }

    if remainder >= DAY {
        let days = remainder / DAY;
        remainder %= DAY;
        result.push_str(&format!("{}d ", days));
    }

    if remainder >= HOUR {
        let hours = remainder / HOUR;
        remainder %= HOUR;
        result.push_str(&format!("{}h ", hours));
    }

    if remainder >= MINUTE {
        let minutes = remainder / MINUTE;
        remainder %= MINUTE;
        result.push_str(&format!("{}m ", minutes));
    }

    if remainder >= SECOND {
        result.push_str(&format!("{}s", remainder));
    }

    result.trim().to_string()
}
```

[See it in rust playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2024&gist=9afa5a66830620e2bbf3c64d2e5d8425).

References:
* <https://doc.rust-lang.org/std/time/struct.Duration.html>

# Convert `chrono::TimeDelta` to human readable format

```rs
use chrono::{DateTime, TimeDelta, Utc};

fn format_time_delta(delta: TimeDelta) -> String {
    if delta.num_seconds() == 0 {
        return "0s".to_string();
    }

    let mut result = String::new();
    let mut remaining = delta;

    let days_in_year = 365;
    if remaining.num_days().abs() >= days_in_year {
        let years = remaining.num_days() / days_in_year;
        remaining -= TimeDelta::days(years * days_in_year);
        result.push_str(&format!("{}y ", years));
    }

    let days_in_month = 30;
    if remaining.num_days().abs() >= days_in_month {
        let months = remaining.num_days() / days_in_month;
        remaining -= TimeDelta::days(months * days_in_month);
        result.push_str(&format!("{}mo ", months));
    }

    if remaining.num_days().abs() >= 7 {
        let weeks = remaining.num_days() / 7;
        remaining -= TimeDelta::days(weeks * 7);
        result.push_str(&format!("{}w ", weeks));
    }

    if remaining.num_days().abs() > 0 {
        result.push_str(&format!("{}d ", remaining.num_days()));
        remaining -= TimeDelta::days(remaining.num_days());
    }

    if remaining.num_hours().abs() > 0 {
        result.push_str(&format!("{}h ", remaining.num_hours()));
        remaining -= TimeDelta::hours(remaining.num_hours());
    }

    if remaining.num_minutes().abs() > 0 {
        result.push_str(&format!("{}m ", remaining.num_minutes()));
        remaining -= TimeDelta::minutes(remaining.num_minutes());
    }

    if remaining.num_seconds().abs() > 0 {
        result.push_str(&format!("{}s", remaining.num_seconds()));
    }

    result.trim().to_string()
}

fn get_time_delta(date_time: DateTime<Utc>) -> String {
    let now = Utc::now();
    let formatted_now = now.format("%Y-%m-%dT%H:%M:%SZ");
    if now < date_time {
        format!(
            "{} after {}",
            format_time_delta(date_time - now),
            formatted_now
        )
    } else {
        format!(
            "{} before {}",
            format_time_delta(now - date_time),
            formatted_now
        )
    }
}
```

[See it in rust playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2024&gist=204a596597133080cf1df79329c55760).

References:
* <https://docs.rs/chrono/latest/chrono/struct.TimeDelta.html>

