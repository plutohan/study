# EasyRust Chapters 141-150

Source: [EasyRust Playlist](https://www.youtube.com/playlist?list=PLfllocyHVgsSJf1zO6k6o3SX2mbZjAqYE)

## The Anyhow Crate

`anyhow` is a library that provides an easy-to-use `Error` type for applications. It works well when you don't care exactly *what* the error is, but you want to propagate it easily and possibly add context.

### Before Anyhow

Without `anyhow`, returning dynamic errors often involves `Box<dyn Error>`.

```rust
use std::error::Error;

fn try_to_make_numbers(int: &str, float: &str) -> Result<(), Box<dyn Error>> {
    let _my_integer = int.parse::<i32>()?; 
    let _my_float = float.parse::<f64>()?;
    Ok(())
}

fn main() {
    let first_try = try_to_make_numbers("8", "tnohentho");
    println!("{:?}", first_try);
}
```

### With Anyhow

`anyhow::Error` handles the underlying error types for you.

```rust
use anyhow::Result; // Anyhow has its own Result alias: Result<T, anyhow::Error>

fn try_to_make_numbers(int: &str, float: &str) -> Result<()> {
    let _my_integer = int.parse::<i32>()?; 
    let _my_float = float.parse::<f64>()?;
    Ok(())
}

fn main() {
    let first_try = try_to_make_numbers("8", "tnohentho");
    println!("{:?}", first_try);
}
```

### Adding Context

You can use `.context()` or `.with_context()` to add more details to an error chain. This helps in debugging to know *where* and *why* an error happened.

```rust
use anyhow::{Context, Result};

fn try_to_make_numbers(int: &str, float: &str) -> Result<()> {
    let _my_integer = int.parse::<i32>()
        .with_context(|| format!("Could not parse integer from '{}'", int))?;
        
    let _my_float = float.parse::<f64>()
        .with_context(|| format!("Could not parse float from '{}'", float))?;
        
    Ok(())
}
```

### Creating Ad-hoc Errors

You can create errors on the fly using the `anyhow!` macro (similar to `panic!`, but returns a Result).

```rust
use anyhow::{anyhow, Result};

fn check_number(x: i32) -> Result<()> {
    if x == 9 {
        return Err(anyhow!("Uh oh, x shouldn't be 9!"));
    }
    Ok(())
}
```

## The Thiserror Crate

While `anyhow` is for *applications* (consuming errors), `thiserror` is for *libraries* (creating errors). It allows you to easily derive `std::error::Error` for your own enums.

```rust
use thiserror::Error;

#[derive(Error, Debug)]
enum CompanyError {
    #[error("Not enough data")]
    NotEnoughData,
    
    #[error("Too old: {0} can't be over 120")]
    TooOld(u8),
    
    #[error("Got {0}, should be under 10,000")]
    TooBig(u32),
}

fn main() {
    let some_error = CompanyError::TooBig(20000);
    let second_error = CompanyError::NotEnoughData;
    
    println!("{}", some_error); // Output: Got 20000, should be under 10,000
    println!("{}", second_error); // Output: Not enough data
}
```

### Anyhow + Thiserror

The typical pattern in Rust is:
1.  Use `thiserror` to define your specific error types (domain errors).
2.  Use `anyhow` in your `main` or top-level functions to handle those errors easily.

```rust
use thiserror::Error;
use anyhow::{Result, Context};

#[derive(Error, Debug)]
enum DataError {
    #[error("Invalid ID: {0}")]
    InvalidId(i32),
}

fn get_data(id: i32) -> Result<String, DataError> {
    if id < 0 {
        return Err(DataError::InvalidId(id));
    }
    Ok("Some Data".to_string())
}

fn main() -> Result<()> {
    // We convert DataError into anyhow::Error here automatically (thanks to ?)
    // And we add extra context
    let data = get_data(-5).context("Failed to fetch user data")?;
    
    println!("{}", data);
    Ok(())
}
```

## Serde (Serialize + Deserialize)

`serde` is the standard framework for serializing and deserializing Rust data structures. It usually requires `serde_json` (or other format crates like `serde_yaml`, `toml`).

```rust
use serde::{Serialize, Deserialize};
use serde_json;

#[derive(Serialize, Deserialize, Debug)]
struct UserRequest {
    points: u32,
    age: u8,
}

fn main() {
    let request_json = r#"
        {
            "points": 100,
            "age": 120
        }
    "#;

    // Deserialize: JSON String -> Rust Struct
    let user_request: UserRequest = serde_json::from_str(request_json).unwrap();
    println!("Got struct: {:?}", user_request);
    
    // Serialize: Rust Struct -> JSON String
    let json_output = serde_json::to_string(&user_request).unwrap();
    println!("Back to JSON: {}", json_output);
}
```

## Rand

The `rand` crate provides random number generation. `fastrand` is a lighter alternative.

### Using ThreadRng

`thread_rng()` gives you a random number generator that is local to the current thread.

```rust
// In Cargo.toml: rand = "0.8"
use rand::prelude::*;

fn main() {
    let mut rng = thread_rng();
    
    // Generate a random boolean
    let coin_flip: bool = rng.gen(); 
    println!("Coin flip: {}", coin_flip);
    
    // Generate a number in a range
    let die_roll = rng.gen_range(1..=6);
    println!("Die roll: {}", die_roll);
    
    // Generate a random u8
    let random_u8: u8 = rng.gen();
    println!("Random u8: {}", random_u8);
}
```
