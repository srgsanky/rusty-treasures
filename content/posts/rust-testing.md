+++
date = '2025-12-19T15:20:25-08:00'
draft = false
title = 'Testing'
toc = true
+++


## Testing async code

Standard unit tests (`#[test]`) cannot test async code. To test async code, you need an async runtime like tokio or async-std.

### Using tokio

```toml
[dev-dependencies]
tokio = { version = "1", features = ["macros", "rt"] }
```

Use `#[tokio::test]` instead of `#[test]`.


### Using async-std

```toml
[dev-dependencies]
async-std = { version = "1", features = ["attributes"] }
```

Use `#[async_std::test]` instead of `#[test]`.


## Mocking dependencies

Use `mockall` crate <https://crates.io/crates/mockall>.


```rust
use mockall::automock;

#[automock]
pub trait Database {
    async fn get_user(&self, id: u32) -> u32;
}
```

`automock` gives you a mock implementation with the prefix `Mock` (in the above example, it is `MockDatabase`). It generates methods on the mock that can be used to setup expectations and verify the expectations

```rust
#[tokio::test]
async fn test_with_mock() {
    let mut mock = MockDatabase::new();

    // Setup expectations
    mock.expect_get_user()
        .with(mockall::predicate::eq(5))
        .returning(|_| 42); // Returns a value, not a future!

    // Actual call that exercises the mock
    let result = mock.get_user(5).await;
    assert_eq!(result, 42);
}
```

### Mocking HTTP calls

Mocking `reqwest` is difficult due to the fact that you cannot create the `reqwest::Response` or `reqwest::Client` objects. There are two options

1. Mock the network server by starting a fake local server if you care about the HTTP details.
1. Mock the logic (hide the reqwest behind a trait) if you don't care about the HTTP details.
    1. Do not return `reqwest::Response` in your trait. Return your domain objects like `User` (e.g., `Result<User, Error>`)

Use `wiremock` to spin up a local server.

Hide the dependency logic behind a trait and mock the trait for tests using `mockall`.

## Code coverage

Generate code coverage data by using cargo tarpaulin <https://crates.io/crates/cargo-tarpaulin>

```bash
# This command takes about 2m
cargo install cargo-tarpaulin

# On-demand
cargo tarpaulin --out Lcov

# Auto-generation
cargo watch -x "tarpaulin --out Lcov"

```

View the coverage data in vscode using [Coverage Gutters by ryanluker](https://marketplace.visualstudio.com/items?itemName=ryanluker.vscode-coverage-gutters). Coverage Gutters requires an LCOV file.

Show coverage by using the command `>Coverage Gutters: Display Coverage`.


### Code coverage using nextest and cargo-llvm-cov

```bash
cargo install cargo-nextest --locked
cargo +stable install cargo-llvm-cov --locked
```

<https://nattrio.medium.com/visualizing-rust-code-coverage-in-vs-code-781aaf334f11>


Generate the coverage report

```bash
cargo llvm-cov nextest
# Generate coverage report in lcov format for vscode
cargo llvm-cov nextest --lcov --output-path ./target/lcov.info
cargo watch -x 'llvm-cov nextest --lcov --output-path ./target/lcov.info' -w src
```

Open the report in HTML format

```bash
cargo llvm-cov nextest --open
```
