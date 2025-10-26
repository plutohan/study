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

### Type Casting

Rust uses the `as` keyword for simple type conversions:

```rust
let a = 'a'; // single quote for characters!

// Casting = simple type change using 'as'
let my_number = 'a' as u8;

// 97 is the ASCII value of 'a'
println!("{}", my_number); // 97

let my_number = 100; // 100 is the ASCII value of 'd'
println!("{}", my_number as u8 as char); // d
```

### Type Inference for Numbers

Rust provides several ways to specify number types:

```rust
let small_number: u8 = 10;
let small_number = 10u8; // 10u8 = 10 of type u8
let small_number = 10_u8; // This is easier to read
let big_number = 100_000_000_i32; // 100 million is easy to read with _
let number = 0________u8; // _ does not change the number. And it doesn't matter how many _ you use
```


### Characters

**Important:** Single quotes for characters, double quotes for strings

**Character sizes:**
- Size of a `char` is always 4 bytes
- Size of string varies depending on the characters:
  - Size of string containing 'a': 1 byte
  - Size of string containing 'ß': 2 bytes
  - Size of string containing '国': 3 bytes
  - Size of string containing '𓅱': 4 bytes

```rust
let slice = "Hello!";
println!("Slice is {} bytes and also {} characters.", slice.len(), slice.chars().count());
// Slice is 6 bytes and also 6 characters.

let slice2 = "안녕!"; // 3 bytes for '안' + 3 bytes for '녕' + 1 byte for '!' = 7 bytes
println!("Slice2 is {} bytes but only {} characters.", slice2.len(), slice2.chars().count());
// Slice2 is 7 bytes but only 3 characters.
```

### Floating-Point Numbers

**Floating-point types:** `f32` and `f64`
**Default:** `f64`

```rust
let my_float = 5.0; // f64 (default)
let my_other_float: f32 = 8.5;

let third_float = my_float + my_other_float as f64;
```

## println! Macro

Rust's `println!` macro supports different formatting styles:

```rust
fn give_age() -> i32 {
    42
}

fn main() {
    let my_name = "david";
    
    // Traditional positional arguments
    println!("MY NAME: {}, MY AGE: {}", my_name, give_age()); // Good
    
    // Named arguments (available from Rust 1.58.0)
    println!("MY NAME: {my_name}, MY AGE: {}", give_age()); // Good from 1.58.0
    
    // This won't work - can't call functions directly in format strings
    // println!("MY NAME: {my_name}, MY AGE: {give_age()}"); // Error
    
    let my_age = 1;
    // This won't work - can't use expressions in curly braces
    // println!("MY NAME: {my_name}, MY AGE: {my_age + 1}"); // Error
}
```

### Debug Printing

In Rust, we call empty tuple instead of void:

```rust
// In Rust we call empty tuple instead of void
fn empty_tuple() {
    // This function returns () (empty tuple)
}

fn main() {
    let tuple = empty_tuple();
    
    // This doesn't work since () doesn't implement Display trait
    // println!("{}", tuple); // Error
    
    // This works! {:?} is debug print
    println!("{:?}", tuple); // ()
}
```

## Code Blocks

Code blocks in Rust can return values. The last expression without a semicolon becomes the return value:

```rust
fn give_number(one: i16, two: i16) -> i16 {
    let multiplied_by_ten = {
        let ten = 10;
        ten * one * two // <- By ending without ;, this value is assigned to multiplied_by_ten
    };
    multiplied_by_ten
}

fn main() {
    let number = give_number(9, 2);
    println!("{number}"); // 180
}
```