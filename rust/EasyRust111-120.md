# EasyRust Chapters 111-120

Source: [EasyRust Playlist](https://www.youtube.com/playlist?list=PLfllocyHVgsSJf1zO6k6o3SX2mbZjAqYE)

## Function Pointers

You can pass functions as arguments to other functions. This is done using the `fn` type.

```rust
fn gives_five() -> u8 {
    5
}

fn gives_six() -> u8 {
    6 
}

fn add_to_function_output(my_function: fn() -> u8, some_number: u8) {
    let my_number = my_function();
    let next_number = my_number + some_number;
    println!("We got {}", next_number);
}

fn main() {
    add_to_function_output(gives_five, 8);
    add_to_function_output(gives_six, 8);
}
```

## Closures

Closures are anonymous functions that can capture their environment. They are very useful when you want to pass a function to another function, especially if it needs to use variables from the surrounding scope.

### Closure Traits

There are three traits for closures, depending on how they capture variables:

1.  **`Fn`**: Captures by reference (`&T`). Can be called multiple times without changing the environment.
2.  **`FnMut`**: Captures by mutable reference (`&mut T`). Can modify the environment. Can be called multiple times.
3.  **`FnOnce`**: Captures by value (moves ownership). Can be called only once because it consumes the variables.

```rust
fn fn_closures<F>(f: F)
where
    F: Fn(),
{
    f();
}

fn fn_mut_closures<F>(mut f: F)
where
    F: FnMut(),
{
    f();
}

fn fn_once_closures<F>(f: F)
where
    F: FnOnce(),
{
    f();
}

fn main() {
    let my_string = String::from("Hello there");

    // Fn closure: just reads my_string
    let print_it = || {
        println!("{}", my_string);
    };
    fn_closures(print_it);

    let mut my_number = 5;
    
    // FnMut closure: modifies my_number
    let mut add_to_it = || {
        my_number += 10;
        println!("Number is now {}", my_number);
    };
    fn_mut_closures(add_to_it);

    let my_owned_string = String::from("I am owned");
    
    // FnOnce closure: takes ownership of my_owned_string (drop consumes it)
    let take_ownership = || {
        drop(my_owned_string); 
    };
    fn_once_closures(take_ownership);
}
```

## impl Trait

`impl Trait` is a cleaner way to express that a function returns "something that implements this trait" or accepts "something that implements this trait", without needing to write out full generic syntax.

### impl Trait vs Generics

Using `impl Trait` in arguments is syntactic sugar for generics.

**Generic Syntax:**
```rust
use std::fmt::Display;

fn generic_function<T: Display>(input: T) {
    println!("{}", input);
}
```

**impl Trait Syntax (Syntactic Sugar):**
```rust
fn impl_function(input: impl Display) {
    println!("{}", input);
}
```

In return positions, `impl Trait` is very useful because it lets you return a concrete type (like a complex closure or iterator) without naming it explicitly. In the past, you often had to use `Box<dyn Trait>` for this, but `impl Trait` is more efficient (static dispatch).

## Attributes

Attributes are metadata (hints) given to the compiler. They typically start with `#[ ... ]` for the item following them, or `#![ ... ]` for the entire file/crate.

### Common Attributes

-   **`#[derive(...)]`**: Automatically implements traits like `Debug`, `Clone`, `Copy`, etc.
-   **`#[cfg(...)]`**: Conditional compilation. It tells the compiler to check a configuration (like the OS) before compiling the code.

```rust
#[cfg(target_os = "linux")]
fn do_something() {
    println!("I am running in Linux");
}

#[cfg(target_os = "windows")]
fn do_something() {
    println!("I am running in Windows");
}
```

-   **`#[test]`**: Marks a function as a test function. The test runner will convert this into a test.

```rust
#[test]
fn tests_a_thing() {
    // If this function panics, the test fails
    let my_string = String::from("Hello");
    assert_eq!(my_string, "Hello"); // Passes
}

#[test]
fn tests_another_thing() {
    // ...
}
```

-   **`#[deprecated]`**: Marks code as deprecated. The compiler will emit a warning if you use it.
-   **`#[repr(C)]`**: Tells Rust to lay out the struct in memory exactly like C does. This is important for FFI (Foreign Function Interface) when talking to C code.
