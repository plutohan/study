# EasyRust Chapters 111-120

Source: [EasyRust Playlist](https://www.youtube.com/playlist?list=PLfllocyHVgsSJf1zO6k6o3SX2mbZjAqYE)

## Function Pointers

You can pass functions as arguments to other functions. This is done using the `fn` type (lowercase `f`). This is different from the `Fn` traits (function traits) used for closures, though function pointers implement all three closure traits.

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

There are three traits for closures, strictly ranked by their capabilities:

1.  **`Fn`**: Captures by reference (`&T`). It doesn't modify the environment and can be called multiple times.
2.  **`FnMut`**: Captures by mutable reference (`&mut T`). It can modify the environment and can be called multiple times.
3.  **`FnOnce`**: Captures by value (moves ownership). It consumes the captured variables, so it can be called **only once**.

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
    // The compiler infers this implements Fn
    let print_it = || {
        println!("{}", my_string);
    };
    fn_closures(print_it);

    let mut my_number = 5;
    
    // FnMut closure: modifies my_number
    // The compiler infers this implements FnMut (and FnOnce)
    let mut add_to_it = || {
        my_number += 10;
        println!("Number is now {}", my_number);
    };
    fn_mut_closures(add_to_it);

    let my_owned_string = String::from("I am owned");
    
    // FnOnce closure: takes ownership of my_owned_string (drop consumes it)
    // The compiler infers this implements FnOnce only
    let take_ownership = || {
        drop(my_owned_string); 
    };
    fn_once_closures(take_ownership);
}
```

## impl Trait

`impl Trait` is a cleaner way to express that a function returns "something that implements this trait" or accepts "something that implements this trait", without needing to write out full generic syntax or deal with complex return types like closures or iterators.

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

In return positions, `impl Trait` is very powerful. It allows you to return a concrete type without naming it. This is essential for closures and iterators, which have unnameable types.

```rust
// Returns "something that is a closure that implements Fn()"
fn returns_closure() -> impl Fn() {
    || println!("I am a closure!")
}
```

## Attributes

Attributes are metadata (hints) given to the compiler. They typically start with `#[ ... ]` for the item following them, or `#![ ... ]` for the entire file/crate.

### Common Attributes

-   **`#[derive(...)]`**: Automatically implements traits like `Debug`, `Clone`, `Copy`, `PartialEq`, etc.
-   **`#[cfg(...)]`**: Conditional compilation. It tells the compiler to strictly include/exclude code based on configuration (like OS, tests, features).

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

-   **`#[test]`**: Marks a function as a unit test. The test runner will detect and run this function.

```rust
#[test]
fn tests_a_thing() {
    let my_string = String::from("Hello");
    assert_eq!(my_string, "Hello"); 
}
```

-   **`#[should_panic]`**: Used alongside `#[test]`. It asserts that the test *should* panic. If the code panics, the test passes. If it doesn't panic, the test fails.

```rust
#[test]
#[should_panic]
fn tests_panic() {
    let my_vec = vec![1, 2, 3];
    let _ = my_vec[10]; // This will panic, so the test passes
}
```

-   **`#[deprecated]`**: Marks code as deprecated. The compiler will emit a warning if you use it, often suggesting a replacement.

```rust
#[deprecated(since = "1.2.0", note = "please use `new_function` instead")]
fn old_function() {}

fn new_function() {}
```

-   **`#[no_mangle]`**: Tells the compiler not to "mangle" the function name. Name mangling is used to ensure unique names for all functions, but for FFI (Foreign Function Interface) calls from C or other languages, you need the name to stay exactly as it is written.
-   **`#[repr(C)]`**: Tells Rust to lay out the struct in memory exactly like C does. This is crucial for FFI compatibility.

## Smart Pointers: Deref and DerefMut

The `Deref` trait allows you to customize the behavior of the dereference operator `*`. More importantly, it enables **Deref Coercion**.

### Deref

By implementing `Deref`, you can treat your custom type essentially like a reference to the inner type. This is what makes `String` behave like `&str` and `Vec<T>` behave like `&[T]`.

```rust
use std::ops::Deref;

struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        &self.0
    }
}

fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y); // *y works because we implemented Deref
}
```

### DerefMut

`DerefMut` is the same as `Deref`, but for mutable references (`*` on a mutable reference).

## String vs &str vs str

Understanding the difference between these types is crucial in Rust.

-   **`String`**: An **owned**, growable, UTF-8 string. It stores its data on the heap. You have full control over it (you can modify, grow, or shrink it).
-   **`str`**: The fundamental string type (Dynamically Sized Type or DST). It represents a sequence of valid UTF-8 bytes stored somewhere in memory. **You almost never see `str` used directly** (like `let x: str`) because its size is unknown at compile time.
-   **`&str` (String Slice)**: A **borrowed reference** to a `str`. It is a "fat pointer" containing:
    1.  A pointer to the start of the `str` data.
    2.  The length of the slice.
    
    Since `&str` has a known size (pointer + length), it can be stored on the stack.

### Comparison

| Feature | `String` | `str` | `&str` |
| :--- | :--- | :--- | :--- |
| **What is it?** | A resizable buffer | Theoretical string data | A view/window into string data |
| **Ownership** | Owns the data | N/A (usually owned by someone else) | Borrows the data |
| **Size (Stack)** | Sized (ptr + cap + len) | **Unsized** (DST) | Sized (ptr + len) |
| **Where does it live?** | Heap | Heap, Stack, or Binary | Stack (implied) |
| **Can be variable?**| Yes (`let s: String`) | **No** (`let s: str` ❌) | Yes (`let s: &str`) |

> **Analogy**:
> - `str` is like an infinite scroll of text. You can't hold "infinite text" in your hands.
> - `&str` is a picture frame that shows a specific part of that text. You can hold the frame.
> - `String` is a notebook where you can write (and store) your own text.
## ToOwned

`ToOwned` is a trait that generalizes the clone-like conversion from borrowing to ownership.

-   **`Clone`**: Creates `T` from `&T`. It assumes you already have a reference to the same type you want to create.
-   **`ToOwned`**: Creates `T` from `&U`. It allows converting a borrowed type (like `&str` or `&[T]`) into an owned type (like `String` or `Vec<T>`).

This is why `&str` doesn't implement `Clone` to produce a `String` (it can't, `Clone` on `&str` just creates another `&str` reference/pointer). Instead, `&str` implements `ToOwned` which produces `String`.

```rust
// Difference between clone() and to_owned() for &str
let my_str: &str = "hello";

let a: &str = my_str.clone();   // clone() on &str returns &str (just copies the pointer + length)
let b: String = my_str.to_owned(); // to_owned() uses ToOwned trait to create a String from &str
```

## Cow (Clone on Write)

`Cow` stands for **Clone on Write**. It is a smart pointer (an enum) that holds either **borrowed** data or **owned** data.

It is used for optimization. If you essentially just want to read data, you keep it borrowed. But if you *might* need to modify it, you can lazily clone it only when that modification happens.

The definition looks like this:
```rust
pub enum Cow<'a, B> 
where B: 'a + ToOwned + ?Sized 
{
    Borrowed(&'a B),
    Owned(<B as ToOwned>::Owned),
}
```

### Example: Cow in Action

```rust
use std::borrow::Cow;

struct User<'a> {
    // Cow might contain a &str (Borrowed) or a String (Owned)
    name: Cow<'a, str>,
}

impl<'a> User<'a> {
    fn is_borrowed(&self) {
        match &self.name {
            Cow::Borrowed(name) => println!("'{}' is borrowed", name),
            Cow::Owned(name) => println!("'{}' is owned", name),
        }
    }
}

fn main() {
    // Case 1: Created from a string literal (borrowed)
    let user_1 = User {
        name: "User 1".into(), // .into() converts &str to Cow::Borrowed
    };
    user_1.is_borrowed(); // Prints: 'User 1' is borrowed

    // Case 2: Created from an owned String
    let my_name = String::from("User 2");
    let user_2 = User {
        name: my_name.into(), // .into() converts String to Cow::Owned
    };
    user_2.is_borrowed(); // Prints: 'User 2' is owned

    // Case 3: Mutating a Cow
    let mut user_3 = User {
        name: "User 3".into(), // Initially borrowed
    };
    user_3.is_borrowed(); // Prints: 'User 3' is borrowed

    // .to_mut() checks:
    // - If it's Borrowed, it clones the data into Owned, updates the Cow to be Owned, and returns &mut String
    // - If it's already Owned, it just returns &mut String
    user_3.name.to_mut().push('!'); 
    
    user_3.is_borrowed(); // Prints: 'User 3!' is owned
}
```

**Why use Cow?**
It lets you design APIs that are efficient for readers (using references) but flexible enough to support ownership/mutation when needed, without forcing an allocation (clone) upfront.
