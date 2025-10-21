# EasyRust Chapters 1-10

Source: [EasyRust Playlist](https://www.youtube.com/playlist?list=PLfllocyHVgsSJf1zO6k6o3SX2mbZjAqYE)

## Comments

Rust supports both single-line and multi-line comments:

```rust
// This is a single-line comment

/*
This is a multi-line comment
*/

// Supports Unicode characters
let 내숫자 = 5;

// Use underscore prefix for unused variables
let _x = 5;
```

## Primitive Types

### Integer Types

**Signed integers:** `i8`, `i16`, `i32`, `i64`, `i128`, `isize`
**Unsigned integers:** `u8`, `u16`, `u32`, `u64`, `u128`, `usize`

> **Note:** `isize` and `usize` depend on the architecture:
> - 64-bit architecture: 64 bits
> - 32-bit architecture: 32 bits

**Default integer type:** `i32`

```rust
let x = 100; // i32 (default)
```

```rust
let x: u8 = 100;
let y = 200;
let z = x + y; // Rust compiler infers the type of y as u8
```
