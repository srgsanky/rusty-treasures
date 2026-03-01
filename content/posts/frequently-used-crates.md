+++
date = '2025-12-19T13:27:01-08:00'
draft = false
title = 'Frequently Used Crates'
toc = true
+++

## By name

NOTE: The crates are ordered in alphabetic order.

### anyhow

Create error types on the fly without having to define them. Use this when you
only care about the presence or absence of an error without worrying about the
actual error.

<https://crates.io/crates/anyhow>

### clap

Program command line arguments easily.

<https://crates.io/crates/clap>

### futures

Utilities for async programming.

<https://crates.io/crates/futures>

### hashbrown

Rust port of Google's SwissTable

<https://crates.io/crates/hashbrown>

Swisstable links

1. <https://abseil.io/blog/20180927-swisstables>
1. <https://github.com/abseil/abseil-cpp/blob/master/absl/container/internal/raw_hash_set.h>
1. <https://www.youtube.com/watch?v=ncHmEUmJZf4>

### mockall

Mock object library.

<https://crates.io/crates/mockall>


### quote

Turns rust syntax tree into tokens of rust source code. This is useful in writing proc macros.

<https://crates.io/crates/quote>

### rand

Generate random numbers.

<https://crates.io/crates/rand>

### reqwest

HTTP client library.

<https://crates.io/crates/reqwest>

### syn

Parser for rust source code. This is useful in writing proc macros.

<https://crates.io/crates/syn>

### tokio

Async runtime that drives async tasks to completion.

<https://crates.io/crates/tokio>


## By category

### Async programming

1. futures
1. reqwest
1. tokio

### Command line arguments

1. clap

### Data structures

1. hashbrown (port of swisstable)

### Testing

1. mockall

### Writing macros

1. syn and quote