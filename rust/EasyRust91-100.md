# EasyRust Chapters 91-100

Source: [EasyRust Playlist](https://www.youtube.com/playlist?list=PLfllocyHVgsSJf1zO6k6o3SX2mbZjAqYE)

## Lifetimes

Lifetimes are a way to tell the compiler how long references should be valid. They ensure that references don't outlive the data they point to, preventing dangling pointers.

### Understanding Lifetime Annotations

When you write `T: Display`, it means "please only take T if it has Display". It does not mean: "I am giving Display to T".

The same is true for lifetimes. When you write `'a` here:

```rust
#[derive(Debug)]
struct City<'a> {
    name: &'a str,
    date_founded: u32,
}

fn main() {}
```

It means "please only take an input for name if it lives at least as long as City". It does not mean: "I will make the input for name live as long as City".

### Lifetime in impl Blocks

When implementing methods for a struct with lifetime parameters, you must specify the lifetime in the `impl` block as well.

```rust
struct Adventurer<'a> {
    name: &'a str,
    hit_points: u32,
}

impl<'a> Adventurer<'a> {
    fn take_damage(&mut self) {
        self.hit_points -= 20;
        println!("{} has {} hit points left!", self.name, self.hit_points);
    }
}

fn main() {}
```

**Error without lifetime annotation:**
```
error[E0726]: implicit elided lifetime not allowed here
 --> src\main.rs:6:6
  |
6 | impl Adventurer {
  |      ^^^^^^^^^^- help: indicate the anonymous lifetime: `<'_>`
```

The compiler needs to know the lifetime parameter when implementing methods for a struct with lifetimes.

### Implementing Traits with Lifetimes

When implementing traits (like `Debug`) for structs with lifetimes, you must include the lifetime parameter:

```rust
use client::InternetClient;
use std::fmt;

struct Customer<'a> {
    money: u32,
    name: &'a str,
    client: &'a InternetClient,
}

impl<'a> fmt::Debug for Customer<'a> {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        f.debug_struct("Customer")
            .field("name", &self.name)
            .field("money", &self.money)
            .field("client", &self.client)
            .finish()
    }
}

fn main() {}
```

**Key points:**
- The lifetime `'a` must be specified in the `impl` block: `impl<'a> fmt::Debug for Customer<'a>`
- The `Formatter<'_>` uses an anonymous lifetime `'_` because the formatter's lifetime doesn't need to be explicitly named
- This allows you to implement traits for structs that contain references, even when using external modules

### Static Lifetime

The `'static` lifetime means the reference lives for the entire duration of the program. String literals have a `'static` lifetime because they are stored in the program's binary.

```rust
fn main() {
    let s: &'static str = "I have a static lifetime.";
    // String literals are stored in the binary and live for the entire program
}
```

## Interior Mutability

Interior mutability is a design pattern in Rust that allows you to mutate data even when you have an immutable reference to it. This is useful when you need to modify data through a shared reference.

**Key concepts:**
- `&` = shared reference (immutable)
- `&mut` = unique reference (mutable)

### Cell

`Cell<T>` provides interior mutability for types that implement `Copy`. It allows you to mutate the value even through an immutable reference, but it's **not thread-safe**.

```rust
use std::cell::Cell;

struct PhoneModel {
    company_name: String,
    model_name: String,
    screen_size: f32,
    memory: usize,
    date_issued: u32,
    on_sale: Cell<bool>,
}

fn main() {
    let super_phone_3000 = PhoneModel {
        company_name: "YY Electronics".to_string(),
        model_name: "Super Phone 3000".to_string(),
        screen_size: 7.5,
        memory: 4_000_000,
        date_issued: 2020,
        on_sale: Cell::new(true),
    };

    // 10 years later, super_phone_3000 is not on sale anymore
    super_phone_3000.on_sale.set(false);
}
```

**Why use Cell?**
- You want to make just one field mutable without making the entire struct mutable
- If you write `let mut super_phone_3000`, then every field becomes mutable
- `Cell` allows you to have interior mutability for specific fields

**Limitations:**
- Use it for small `Copy` types (like `bool`, `i32`, etc.)
- For non-`Copy` types, use `RefCell` instead
- Not thread-safe (use `Mutex` or `RwLock` for thread safety)

### RefCell

`RefCell<T>` provides interior mutability with **runtime-checked borrowing rules**. Unlike `Cell`, it works with non-`Copy` types and allows you to get references to the inner value.

**Important:** You have to be careful with a `RefCell`, because it checks borrows at runtime, not compilation time. Runtime means when the program is actually running (after compilation). So this will compile, even though it is wrong:

```rust
use std::cell::RefCell;

#[derive(Debug)]
struct User {
    id: u32,
    year_registered: u32,
    username: String,
    active: RefCell<bool>,
    // Many other fields
}

fn main() {
    let user_1 = User {
        id: 1,
        year_registered: 2020,
        username: "User 1".to_string(),
        active: RefCell::new(true),
    };

    let borrow_one = user_1.active.borrow_mut(); // first mutable borrow - okay
    let borrow_two = user_1.active.borrow_mut(); // second mutable borrow - not okay
}
```

But if you run it, it will immediately panic:

```
thread 'main' panicked at 'already borrowed: BorrowMutError', C:\Users\mithr\.rustup\toolchains\stable-x86_64-pc-windows-msvc\lib/rustlib/src/rust\src\libcore\cell.rs:877:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
error: process didn't exit successfully (exit code: 101)
```

`already borrowed: BorrowMutError` is the important part. So when you use a `RefCell`, it is good to compile and run to check.

**Key differences from Cell:**
- Works with non-`Copy` types
- Returns references (`borrow()` and `borrow_mut()`) instead of copying values
- Borrowing rules are checked at runtime, not compile time
- Panics if borrowing rules are violated

## Rc (Reference Counting)

`Rc<T>` stands for "Reference Counted". It allows multiple ownership of the same data by keeping track of how many references exist. When the last reference is dropped, the data is automatically cleaned up.

### Why Rc is Better Than .clone()

Using `Rc::clone()` is much more efficient than using `.clone()` on the data itself:

- **`.clone()` on data**: Creates a complete copy of the data (expensive for large data)
- **`Rc::clone()`**: Only increments a reference counter (cheap, just copying a pointer)

```rust
use std::rc::Rc;

fn takes_a_string(input: String) {
    println!("It is: {}", input)
}

fn also_takes_a_string(input: String) {
    println!("It is: {}", input)
}

fn main() {
    let user_name = Rc::new("Hello there".to_string());

    // When you clone an Rc, it only increments the reference count (cheap)
    // The actual string data is not copied
    let name1 = user_name.clone(); // Rc::clone() - cheap, just increments counter
    let name2 = user_name.clone(); // Rc::clone() - cheap, just increments counter
    
    // If you need to extract the String (which requires ownership):
    takes_a_string((*user_name).clone()); // This copies the String (expensive)
    also_takes_a_string((*name1).clone()); // This copies again (expensive)
    
    // But sharing the Rc itself is cheap - multiple owners, same data
}
```

**Why Rc::clone() is better than cloning the data:**
- `Rc::clone(&rc)` only increments a reference counter (very cheap - just an integer increment)
- Cloning the actual data (like `(*rc).clone()`) copies all the data (expensive for large data)
- With `Rc`, you can share the same data across multiple owners without copying until you actually need to extract the value
- The data is automatically freed when the last `Rc` is dropped

### More Complex Example with Rc

Here's a more realistic example showing how `Rc` can be used to share data:

```rust
use std::rc::Rc;

#[derive(Debug)]
struct City {
    name: String,
    population: u32,
    city_history: Rc<String>, // String inside an Rc
}

#[derive(Debug)]
struct CityData {
    names: Vec<String>,
    histories: Vec<Rc<String>>, // A Vec of Strings inside Rcs
}

fn main() {
    let calgary = City {
        name: "Calgary".to_string(),
        population: 1_200_000,
        // Pretend that this string is very very long
        city_history: Rc::new("Calgary began as a ...".to_string()), // Rc::new() to make the Rc
    };

    let canada_cities = CityData {
        names: vec![calgary.name],
        histories: vec![calgary.city_history.clone()], // .clone() to increase the count
    };

    println!("Calgary's history is: {}", calgary.city_history);
    println!("Reference count: {}", Rc::strong_count(&calgary.city_history));
    let new_owner = calgary.city_history.clone();
    println!("Reference count after clone: {}", Rc::strong_count(&calgary.city_history));
}
```

**Key points:**
- `Rc::new()` creates a new reference-counted value
- `Rc::clone()` increments the reference count (doesn't copy the data)
- `Rc::strong_count()` shows how many references exist
- The data is automatically freed when the last `Rc` is dropped
- `Rc` is **not thread-safe** (use `Arc` for thread-safe reference counting)

### Combining Rc and RefCell

`Rc<RefCell<T>>` is a common pattern that combines multiple ownership (`Rc`) with interior mutability (`RefCell`). This allows you to:
- Share the same data across multiple owners (via `Rc`)
- Mutate the data even through shared references (via `RefCell`)

**Why combine them?**
- `Rc` allows multiple ownership, but only provides immutable access
- `RefCell` allows interior mutability, but doesn't support multiple owners
- Together, they give you both: multiple owners AND the ability to mutate

```rust
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug)]
struct DataContainer {
    data: Rc<RefCell<String>>,
}

fn main() {
    // Create shared mutable data
    let important_data = Rc::new(RefCell::new("Super secret data".to_string()));

    // Multiple containers can share the same data
    let container_1 = DataContainer {
        data: Rc::clone(&important_data), // Reference count: 2
    };

    let container_2 = DataContainer {
        data: Rc::clone(&important_data), // Reference count: 3
    };

    // All containers see the same initial data
    println!("container_1: {:?}", container_1.data.borrow());
    println!("container_2: {:?}", container_2.data.borrow());

    // Mutate through container_1 - affects the shared data
    container_1.data.borrow_mut().push_str(" and more data");
    
    // Mutate through container_2 - also affects the same shared data
    container_2.data.borrow_mut().push_str(" and even more data");

    // Both containers now see the same updated data
    println!("container_1: {:?}", container_1.data.borrow());
    println!("container_2: {:?}", container_2.data.borrow());
    
    // The original reference also sees the changes
    println!("important_data: {:?}", important_data.borrow());
}
```

**Output:**
```
container_1: RefCell { value: "Super secret data" }
container_2: RefCell { value: "Super secret data" }
container_1: RefCell { value: "Super secret data and more data and even more data" }
container_2: RefCell { value: "Super secret data and more data and even more data" }
important_data: RefCell { value: "Super secret data and more data and even more data" }
```

**Key points:**
- `Rc<RefCell<T>>` = shared ownership + interior mutability
- All owners see the same data and can mutate it
- Changes made through one reference are visible to all other references
- Remember: `RefCell` checks borrowing rules at runtime, so you must be careful not to create multiple mutable borrows simultaneously
- This pattern is useful for shared state that needs to be mutated by multiple parts of your program