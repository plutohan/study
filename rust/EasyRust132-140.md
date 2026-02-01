# EasyRust Chapters 132-140

Source: [EasyRust Playlist](https://www.youtube.com/playlist?list=PLfllocyHVgsSJf1zO6k6o3SX2mbZjAqYE)

## The Any Trait

`std::any::Any` is a trait that enables dynamic typing in Rust. It allows you to emulate some behavior of dynamic languages by letting you check a type at runtime.

Most types satisfy `Any` automatically as long as they represent `'static` types (which means they don't contain any non-static references).

### Getting Type Names

You can use `std::any::type_name` to get the string name of a type.

```rust
use std::any::{Any, type_name};

struct MyType;

fn get_type_names<T: Any, U: Any>(_: T, _: U) -> (String, String) {
    (type_name::<T>().to_string(), type_name::<U>().to_string())
}

fn main() {
    let (t1, t2) = get_type_names(MyType, 42);
    println!("Type 1: {}, Type 2: {}", t1, t2);
    // Output: Type 1: rust_playground::MyType, Type 2: i32
}
```

## Dynamic Typing Methods

The `Any` trait provides methods to check types at runtime and "downcast" them back to their original concrete type.

### Key Methods

1.  **`is::<T>()`**: Returns `true` if the boxed value is of type `T`.
2.  **`downcast_ref::<T>()`**: Attempts to cast the value to a reference `&T`. Returns `Some(&T)` if the types match, or `None` if they don't.

### Example: Identifying Types

```rust
use std::any::Any;

fn print_if_string(s: &dyn Any) {
    if s.is::<String>() {
        println!("It's a String!");
    } else {
        println!("Not a String...");
    }
}

fn main() {
    let my_string = "Hello World".to_string();
    let my_number = 10;

    print_if_string(&my_string); // It's a String!
    print_if_string(&my_number); // Not a String...
}
```

### Example: Downcasting

You often use `downcast_ref` when you want to actually use the data if it matches a specific type.

```rust
use std::any::Any;

fn print_any(value: &dyn Any) {
    if let Some(num) = value.downcast_ref::<i32>() {
        println!("It's an integer: {}", num);
    } else if let Some(string) = value.downcast_ref::<String>() {
        println!("It's a string length: {}", string.len());
    } else {
        println!("Got something else!");
    }
}

fn main() {
    print_any(&10); 
    print_any(&String::from("Rust"));
}
```

## Use Cases for Any

### Universal Logger

The `Any` trait is helpful when building a system that needs to accept "anything" and log it. For example, a logging function can take `Box<dyn Any>` (or `&dyn Any`) and try to downcast it to known types (`String`, `i32`, `Error`, etc.) to print specific details, falling back to a generic message if the type is unknown.

### Panic Hooks

When a Rust program panics, you can define a custom behavior called a "panic hook". The panic hook receives a `PanicInfo` struct, which contains the arguments passed to the panic (the "payload").

Because a user can panic with any value (e.g., `panic!("error")`, `panic!(404)`), the payload is stored as `&dyn Any`. You can use `downcast_ref` to inspect this payload.

**Why downcast_ref?**
The panic payload is hidden behind the `Any` trait object. To get the useful error message out, we need to guess its type. Most panics use `&str` (string literals) or `String`. `downcast_ref::<&str>()` allows us to check "Is this error message a string slice?" and if so, gives us a reference to print it.

```rust
use std::panic::{set_hook, take_hook};

fn main() {
    // 1. Set a custom panic hook
    set_hook(Box::new(|panic_info| {
        println!("Custom Panic Hook Detected!");
        
        // Try to downcast the payload to a string slice (&str)
        // We use downcast_ref because we only have a reference (&dyn Any) 
        // and we want to see if it matches the type &str.
        if let Some(s) = panic_info.payload().downcast_ref::<&str>() {
            println!("Panic message detected: {:?}", s);
        } else {
            println!("Panic occurred with unknown payload type.");
        }
    }));

    // 2. Trigger a panic with a safe operation that fails (like parsing bad int)
    // Note: unwrap() panics if the Result is Err
    let _ = "not-a-number".parse::<i32>().unwrap();
}
```

## External Libraries: Time and Channels

### Time

Rust's standard library provides the `std::time` module to handle time-related operations.

-   **`Instant`**: Represents a specific moment in time. Useful for measuring duration.
-   **`Duration`**: Represents a span of time.

```rust
use std::time::{Instant, Duration};
use std::thread::sleep;

fn main() {
    let start = Instant::now();
    
    // Do some work...
    sleep(Duration::from_secs(2));
    
    let duration = start.elapsed();
    println!("Time elapsed: {} ms", duration.as_millis());
}
```

### Channels

A channel is a way to send data between threads. Rust provides `std::sync::mpsc`, which stands for **Multiple Producer, Single Consumer**. This means many threads can send to the channel, but only one thread can receive from it.

```rust
use std::sync::mpsc::channel;
use std::thread;

fn main() {
    let (sender, receiver) = channel();
    
    let sender_clone = sender.clone(); // Clone sender to allow another thread to own it

    thread::spawn(move || {
        sender.send("Send a &str this time").unwrap();
    });

    thread::spawn(move || {
        sender_clone.send("And here is another &str").unwrap();
    });

    // recv() blocks the current thread until a message is received
    println!("{}", receiver.recv().unwrap());
    println!("{}", receiver.recv().unwrap());
} 
```

### Channels with Any (Sending Mixed Types)

Normally, channels are typed (e.g., `Sender<String>`), meaning you can only send one type of data. However, by using `Box<dyn Any>`, you can send "any" type over the channel.

Note that we need `Send` trait bound (`Box<dyn Any + Send>`) because we are sending these values across threads.

**Using downcast_ref in Channels**
Since the receiver gets a generic `Box<dyn Any>`, it doesn't know what concrete type is inside (Is it a `Book`? A `Magazine`?). We must iterate through possible types using `downcast_ref`. If `downcast_ref::<Book>()` returns `Some`, we know it's a Book and can safely access its fields.

```rust
use std::sync::mpsc::channel;
use std::thread;
use std::thread::sleep;
use std::time::Duration;
use std::any::Any;

fn sleepy(time: u64) {
    sleep(Duration::from_millis(time));
}

#[derive(Debug)]
struct Book {
    name: String,
}

#[derive(Debug)]
struct Magazine {
    name: String,
}

fn book() -> Box<dyn Any + Send> {
    Box::new(Book {
        name: "My Book".to_string(),
    })
}

fn magazine() -> Box<dyn Any + Send> {
    Box::new(Magazine {
        name: "My Magazine".to_string(),
    })
}

fn main() {
    // Channel that sends Box<dyn Any + Send>
    let (sender, receiver) = channel();
    
    let sender_clone = sender.clone();

    // Thread 1: Sends Books
    thread::spawn(move || {
        for _ in 0..5 {
            sleepy(100);
            sender.send(book()).unwrap();
        }
    });

    // Thread 2: Sends Magazines
    thread::spawn(move || {
        for _ in 0..5 {
            sleepy(100);
            sender_clone.send(magazine()).unwrap();
        }
    });

    // Main Thread: Receives and Downcasts
    for _ in 0..10 {
        if let Ok(any_type) = receiver.recv() {
            // Check if the received message is a Book
            if let Some(book) = any_type.downcast_ref::<Book>() {
                println!("Got a book: {:?}", book);
            } 
            // Check if the received message is a Magazine
            else if let Some(magazine) = any_type.downcast_ref::<Magazine>() {
                println!("Got a magazine: {:?}", magazine);
            } else {
                panic!("Expected a magazine or a book, what's going on?");
            }
        }
    }
}
```

## Handling Multiple Error Types with Any (downcast_ref)

Just like `dyn Any`, `dyn Error` can also be downcasted if you need to recover the original concrete error type. This is useful when you have a function that returns a dynamic error (`Box<dyn Error>`) but the caller needs to handle specific errors differently.

```rust
use std::fmt;
use std::error::Error;

#[derive(Debug)]
struct BaseError;

impl fmt::Display for BaseError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "This is a base error")
    }
}

impl Error for BaseError {}

#[derive(Debug)]
struct CompanyError;

impl CompanyError {
    fn print_extra_detail(&self) {
        println!("Here is all the extra detail: blah blah blah");
    }
}

impl fmt::Display for CompanyError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "This is a company error")
    }
}

impl Error for CompanyError {}

// This function returns a Box<dyn Error>, erasing the concrete type
fn give_error(is_company_error: bool) -> Box<dyn Error> {
    if is_company_error {
        Box::new(CompanyError)
    } else {
        Box::new(BaseError)
    }
}

fn main() {
    let error_1 = give_error(true);
    let error_2 = give_error(false);

    // We can downcast generic dyn Error back to concrete CompanyError
    // downcast_ref checks: "Is this dyn Error actually a CompanyError?"
    if let Some(company_error) = error_1.downcast_ref::<CompanyError>() {
        println!("Error 1 is a CompanyError!");
        company_error.print_extra_detail();
    } else {
        println!("Error 1 is just a generic error: {}", error_1);
    }
    
    // This will fail the downcast check because error_2 is BaseError
    if let Some(company_error) = error_2.downcast_ref::<CompanyError>() {
        company_error.print_extra_detail();
    } else {
        println!("Error 2 is just a generic error: {}", error_2);
    }
}
```

This works because `std::error::Error` automatically implements `Any` (as long as it's `'static`), so you can use `downcast_ref` on it directly.
