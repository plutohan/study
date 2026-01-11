# EasyRust Chapters 101-110

Source: [EasyRust Playlist](https://www.youtube.com/playlist?list=PLfllocyHVgsSJf1zO6k6o3SX2mbZjAqYE)

## Type Aliases

Type aliases allow you to give a new name to an existing type. This is particularly useful when you have very long or complex types that you don't want to type out repeatedly.

```rust
// Giving a long iterator type a simpler name
type SkipFourTakeFive<'a> = std::iter::Take<std::iter::Skip<std::slice::Iter<'a, char>>>;

fn returns<'a>(input: &'a Vec<char>) -> SkipFourTakeFive {
    input.iter().skip(4).take(5)
}

fn main() {
    let chars = vec!['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h'];
    let iter = returns(&chars);
    
    for c in iter {
        println!("{}", c);
    }
}
```

It doesn't create a new type, just an alias (another name) for the same type.

## The Newtype Pattern

Unlike type aliases, the "Newtype Pattern" creates a completely distinct type. This involves wrapping a type inside a tuple struct with a single field. This is useful for type safety, ensuring you don't accidentally mix up two different values just because they happen to be the same underlying type (like `String` or `i32`).

```rust
struct File(String); // File is a wrapper around String

fn main() {
    let my_file = File(String::from("I am file contents"));
    let my_string = String::from("I am file contents");
    
    // my_file and my_string are different types and cannot be mixed up.
    // print_file(my_string); // This would cause a compile error
}
```

## Cow (Clone on Write)

`Cow` stands for "Clone on Write". It is a flexible smart pointer enum that can hold either borrowed data or owned data.

- **Borrowed variant**: Holds a reference to data.
- **Owned variant**: Holds the data itself.

It is useful when you usually just want to read data (avoiding allocation), but occasionally need to modify it. When you modify it, `Cow` will "clone" the data into an owned version so you can safely change it.

```rust
use std::borrow::Cow;

fn mask_sensitive_info(log: &str) -> Cow<str> {
    if log.contains("010-") {
        // If sensitive info exists, create a new modified String (Owned)
        let masked = log.replace("010-", "010-****-");
        Cow::Owned(masked)
    } else {
        // If no sensitive info, just use the original reference (Borrowed)
        Cow::Borrowed(log)
    }
}

fn main() {
    // Example 1: Modification needed
    let log1 = "User contact: 010-1234-5678";
    let output1 = mask_sensitive_info(log1); 
    // Here, new memory allocation happens (Owned).

    // Example 2: No modification needed
    let log2 = "System rebooted successfully";
    let output2 = mask_sensitive_info(log2); 
    // No copying happens here. It just points to log2's address. (Performance gain)

    println!("Log 1: {}", output1);
    println!("Log 2: {}", output2);
}
```

## Multithreading

Rust provides basic multithreading capabilities via borrowing and ownership rules that prevent data races.

### Basic Thread Usage

You spawn a thread using `std::thread::spawn`. This returns a `JoinHandle`.

```rust
fn main() {
    for _ in 0..10 {
        let handle = std::thread::spawn(|| {
            println!("I am printing something");
        });

        handle.join(); // Wait for the thread to finish
    }
}
```

`join()` blocks the current thread until the spawned thread finishes. If you don't join, the main thread might exit before the spawned threads get a chance to run.

### Threads and Ownership

Threads can outlive the scope they were created in. Therefore, you cannot simply borrow a value from the main thread into a spawned thread, because the compiler doesn't know if the data will still exist when the thread runs.

**This will imply an error:**

```rust
fn main() {
    let mut my_string = String::from("Can I go inside the thread?");

    let handle = std::thread::spawn(|| {
        println!("{}", my_string); // Error! my_string might be dropped before this runs
    });

    std::mem::drop(my_string); // Proving why it's unsafe
    handle.join();
}
```

**The Solution: `move` closure**

To fix this, use the `move` keyword to force the closure to take ownership of the values it uses.

```rust
fn main() {
    let my_string = String::from("Can I go inside the thread?");

    // 'move' takes ownership of my_string into the thread
    let handle = std::thread::spawn(move || {
        println!("{}", my_string);
    });

    handle.join().unwrap();
}
```

## Mutex (Mutual Exclusion)

A `Mutex` allows you to safely share mutable data between threads. It ensures that only one thread can access the data at a time.

```rust
use std::sync::Mutex;

fn main() {
    let my_mutex = Mutex::new(5);
    
    {
        // .lock() returns a LockResult. unwrap() gives us a MutexGuard.
        let mut mutex_changer = my_mutex.lock().unwrap();
        
        // We can modify the value through the guard (it dereferences to the data)
        *mutex_changer = 6;
        
    } // mutex_changer goes out of scope here. The lock is automatically released.

    println!("{:?}", my_mutex); // Output: Mutex { data: 6 }
}
```

You can also explicitly drop the guard to release the lock:

```rust
let mut mutex_changer = my_mutex.lock().unwrap();
*mutex_changer = 6;
std::mem::drop(mutex_changer); // Lock is released immediately
```

## Arc (Atomic Reference Counting)

`Arc` stands for "Atomic Reference Counting". It is the thread-safe version of `Rc`.

### Understanding Arc vs Mutex

**Arc (Atomic Reference Counted)**
- **Role**: Shared Ownership.
- Allows multiple threads to own references to the same data.
- It atomically manages the reference count. The data is cleaned up only when the last `Arc` is dropped.
- **Limitation**: It does not allow you to modify the internal data (it is immutable).

**Mutex (Mutual Exclusion)**
- **Role**: Interior Mutability.
- Provides a "Lock" mechanism so that only one thread can access the data at a time.
- **Limitation**: It cannot share ownership across threads (traditionally, only one owner holds the value).

**Conclusion**:
You usually combine them: Use `Arc` to give a pointer (copy) to multiple threads, and use `Mutex` inside to let them take turns modifying the value (`Arc<Mutex<T>>`).

### Example

```rust
use std::sync::{Arc, Mutex};

fn main() {
    // Wrap the Mutex in an Arc to share ownership across threads
    let my_number = Arc::new(Mutex::new(0));

    // Clone the Arc to create new handles to the same data
    let my_number1 = Arc::clone(&my_number);
    let my_number2 = Arc::clone(&my_number);

    let thread1 = std::thread::spawn(move || {
        for _ in 0..10 {
            // Lock the mutex and change the value
            *my_number1.lock().unwrap() += 1;
        }
    });

    let thread2 = std::thread::spawn(move || {
        for _ in 0..10 {
            *my_number2.lock().unwrap() += 1;
        }
    });

    thread1.join().unwrap();
    thread2.join().unwrap();
    
    println!("Value is: {:?}", my_number);
    println!("Exiting the program");
}
```

**Output:**
```
Value is: Mutex { data: 20 }
Exiting the program
```

## RwLock (Read-Write Lock)

`RwLock` stands for "Read-Write Lock". It is similar to a `Mutex` but distinguishes between reading and writing.

- **Multiple readers** are allowed at the same time.
- **Only one writer** is allowed at a time.
- Readers and writers cannot access it simultaneously.

It's similar to `RefCell` rules but for threads:
- `.read().unwrap()` gives read access.
- `.write().unwrap()` gives write access.

This is useful when you have data that is read often but written to rarely.

## Clippy

Clippy is the official linter for Rust. It catches common mistakes and improves your code.

To use it, run:
```bash
cargo clippy
```

It acts like a helper that suggests more idiomatic or efficient ways to write your code.

## Box (Smart Pointer)

`Box<T>` allows you to store data on the heap rather than the stack. `Box` is a pointer to the data on the heap.

```rust
fn main() {
    let my_box = Box::new(1); // This stores the integer 1 on the heap
    let an_integer = *my_box; // We can dereference it to get the value
    
    println!("{:?}", my_box);
    println!("{:?}", an_integer);
}
```

### Recursive Types

`Box` is essential for recursive types (types that contain themselves), because the compiler needs to know the size of a type at compile time. A `Box` has a known size (it's just a pointer), wheras a recursive struct without a Box would be infinitely large.

```rust
struct Node {
    value: i32,
    next: Option<Box<Node>>, // Without Box, this would be infinite size
}
```

## Generics vs Trait Objects

When working with traits, you have two main choices: Generics (Static Dispatch) and Trait Objects (Dynamic Dispatch).

### Impl Trait / Generics (Static Dispatch)

The compiler generates a specific version of the function for each concrete type used. This is faster (no runtime overhead) but leads to larger binary size if used with many types.

```rust
use std::fmt::Display;

// Concrete / Generics
fn print<T: Display>(input: T) {
    println!("{}", input);
}

// Syntactic sugar for the above
fn print_2(input: impl Display) {
    println!("{}", input);
}
```

### Box<dyn Trait> (Dynamic Dispatch)

This keeps a single version of the function. The specific type is resolved at runtime (dynamic dispatch). This is more flexible (e.g., you can have a `Vec<Box<dyn Animal>>` with different animals) but slightly slower due to the lookup overhead.

```rust
// Dynamic Dispatch
fn print_3(input: Box<dyn Display>) {
    println!("{}", input);
}
```