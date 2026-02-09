# EasyRust Chapters 151-160

Source: [EasyRust Playlist](https://www.youtube.com/playlist?list=PLfllocyHVgsSJf1zO6k6o3SX2mbZjAqYE)

## Vec Methods

Rust's `Vec` provides powerful methods for sorting and filtering data.

### Sorting and Deduplicating

-   **`sort_unstable()`**: Sorts the vector. It is faster than standard `sort()` but does not preserve the order of equal elements. If you have two `10`s, their relative order might swap. Usually, this is what you want for primitive types.
-   **`dedup()`**: Removes **consecutive** repeated elements. For it to remove all duplicates, the vector must be sorted first.

```rust
fn main() {
    let mut my_vec = vec![1, 5, 2, 5, 1, 10];
    
    // 1. Sort it first (unstable is faster)
    my_vec.sort_unstable(); // now: [1, 1, 2, 5, 5, 10]
    
    // 2. Remove consecutive duplicates
    my_vec.dedup(); // now: [1, 2, 5, 10]
    
    println!("{:?}", my_vec);
}
```

### Other Useful Methods

-   **`shrink_to_fit()`**: Tries to free unused memory. If a Vec has capability 100 but only length 10, this asks the allocator to reduce capacity to 10.
-   **`retain()`**: Filters elements in place. It keeps only the elements for which the closure returns `true`.

```rust
fn main() {
    let mut my_vec = vec![1, 2, 3, 4, 5, 6];
    
    // Keep only even numbers
    my_vec.retain(|&x| x % 2 == 0);
    
    println!("{:?}", my_vec); // [2, 4, 6]
}
```

## String Methods

-   **`pop()`**: Removes the last character and returns `Some(char)` or `None`. Note that `String` is a wrapper around `Vec<u8>`.

### Converting Bytes to String

-   **`String::from_utf8(vec)`**: Converts a `Vec<u8>` into a `String`. Returns `Result` because not all byte sequences are valid UTF-8.
-   **`String::from_utf8_lossy(slice)`**: Converts bytes to a string, including invalid UTF-8. Invalid sequences are replaced with the replacement character . This returns a `Cow<str>` (Clone on Write), meaning it only allocates a new string if it *needs* to replace characters.

```rust
fn main() {
    let valid_utf8 = vec![72, 101, 108, 108, 111]; // "Hello"
    let s1 = String::from_utf8(valid_utf8).unwrap();
    println!("{}", s1);

    let invalid_utf8 = vec![72, 101, 108, 108, 111, 255]; // 255 is invalid
    let s2 = String::from_utf8_lossy(&invalid_utf8);
    println!("{}", s2); // "Hello"
}
```

-   **`split_off(n)`**: Splits the string at index `n`. Returns a new `String` with the second part, leaving the original string with the first part.

## Other String Types

Rust has string types for interacting with the OS and C libraries.

-   **`OsString` / `OsStr`**: Owned/Borrowed strings that match the operating system's format (e.g., UTF-16 on Windows, arbitrary bytes on Linux). Used for file paths, environment variables, and command-line arguments.
-   **`CString` / `CStr`**: Used for FFI (Foreign Function Interface) to talk to C code. C strings end with a null byte (`\0`) and don't store their length.

## Attributes

-   **`#![no_std]`**: A crate-level attribute that tells Rust **not** to link the Rust standard library (`std`). This is used for embedded systems (microcontrollers) or writing operating systems where you don't have an OS allocation/threading support. You only get `core`.
-   **`#![no_implicit_prelude]`**: Tells Rust not to import the standard prelude (like `Vec`, `String`, `Option`, `Result` are no longer in scope by default). This is rarely used.

## Memory and std::mem

-   **`std::mem::size_of::<T>()`**: Returns the size of type `T` in bytes.
-   **`std::mem::size_of_val(&T)`**: Returns the size of the *value* pointed to (useful for unsized types like slices or trait objects).
-   **`std::mem::align_of::<T>()`**: Returns the alignment requirement of type `T` (memory address must be a multiple of this).

### Managing Ownership Manually

-   **`drop(x)`**: Manually runs the destructor of `x` immediately.
-   **`std::mem::forget(x)`**: Takes ownership of `x` and **forgets** it without running its destructor. This leaks memory intentionaly. It is safe (memory leaks are memory-safe in Rust), but often used in FFI to give ownership of data to C.

### Swapping Values

-   **`std::mem::swap(&mut x, &mut y)`**: Swaps the values of `x` and `y`.
-   **`std::mem::replace(&mut dest, src)`**: Moves `src` into `dest` and returns the *old* value of `dest`.
-   **`std::mem::take(&mut dest)`**: Replaces `dest` with its default value and returns the old value. Only works if the type implements `Default`. This is extremely useful for moving out of a mutable reference (like `Option`).

```rust
use std::mem;

fn main() {
    let mut s1 = String::from("Hello");
    let mut s2 = String::from("World");
    
    mem::swap(&mut s1, &mut s2);
    println!("s1: {}, s2: {}", s1, s2); // s1: World, s2: Hello
    
    let old_s1 = mem::replace(&mut s1, String::from("New"));
    println!("Old: {}, New s1: {}", old_s1, s1);

    let mut my_option = Some("Content".to_string());
    // Safe to move out of reference because we leave 'None' behind
    let taken = my_option.take(); 
    println!("Taken: {:?}, Left: {:?}", taken, my_option);
}
```

-   **`std::mem::transmute`**: Reinterprets the bits of a value of one type as another type. **This is incredibly `unsafe`**. Don't use it unless you know exactly what you are doing (e.g. `u8` array to `u32`).

## Macros

Macros are code that writes other code.

### Inspection Macros

These perform string substitution based on where they are called.
-   **`file!()`**: Current filename.
-   **`line!()`**: Current line number.
-   **`column!()`**: Current column number.
-   **`module_path!()`**: Current module path.

### Source Code Macros

-   **`include_str!("path")`**: Reads a file at compile time and embeds it as a `&'static str`.
-   **`include_bytes!("path")`**: Embeds file as `&'static [u8; N]`.
-   **`concat!("a", "b")`**: Concatenates literals at compile time.

### Declarative Macros (macro_rules!)

Macros match patterns in the source code.

#### 1. Simple Pattern Matching

```rust
macro_rules! six_or_print {
    (6) => {
        6
    };
    () => {
        println!("You didn't give me six.");
    };
}

fn main() {
    six_or_print!(); 
    let x = six_or_print!(6);
    println!("{}", x);
}
```

#### 2. Matching Tokens

You can match arbitrary text.

```rust
macro_rules! might_print {
    (Hey there user) => {
        println!("You guessed the secret message!");
    };
    () => {
        println!("You didn't guess it");
    };
}

fn main() {
    might_print!(Hey there user); // Prints secret message
    might_print!();               // Prints didn't guess it
}
```

#### 3. Capturing Arguments

You pass arguments with types like `expr` (expression), `ident` (identifier), `ty` (type), `tt` (token tree).

```rust
macro_rules! print_this {
    ($input:expr) => {
        println!("{:?}", $input);
    }
}

fn main() {
    print_this!(9);
    print_this!("Hi there");
    let my_vec = vec![8, 9, 10];
    print_this!(my_vec);
}
```

#### 4. Repetition

You can accept variable arguments using `$()*`. `+` means "one or more", `*` means "zero or more".

```rust
macro_rules! print_anything {
    ($($input:tt),+) => { // Match one or more token trees, separated by comma (implied here)
        let output = stringify!($($input),+); // Convert tokens to string
        println!("{}", output);
    }
}

fn main() {
    print_anything!(thothe, 9, thdoefgoe, 7652);
}
```

#### 5. Generating Functions

Macros can even write functions for you!

```rust
macro_rules! make_a_function {
    ($name:ident, $($input:tt),*) => {
        fn $name() {
            let output = stringify!($($input),*);
            println!("{}", output);
        }
    };
}

fn main() {
    make_a_function!(print_it, 6, 7, 8, "I");
    print_it(); // This function was created by the macro!
}
```