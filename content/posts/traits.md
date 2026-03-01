+++
date = '2025-12-19T16:30:33-08:00'
draft = false
title = 'Traits'
+++


# Async traits

Async traits allow the use of `async` functions.

```rust
pub trait UserService {
    async fn fetch_user(&self, id: u32) -> Result<User, String>;
}
```


Rust didn't support async traits until 1.75. `async_trait` crate provided the necessary support for using async crates.

Native async traits (in rust 1.75+) only supports static dispatch.

> Static Dispatch: If you only use your traits with generic type parameters (e.g.,
> `T: MyTrait`), native async traits are preferred as they avoid heap allocation.

Continue using `async_trait` crate if you need dynamic dispatch.

> Dynamic Dispatch (dyn Trait): Native async traits are not yet "object safe"
> (dyn-compatible). If you need to store or pass traits as `Box<dyn MyTrait>`, you
> must use `async_trait`.
