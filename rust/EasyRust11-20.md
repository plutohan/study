# EasyRust Chapters 11-20

Source: [EasyRust Playlist](https://www.youtube.com/playlist?list=PLfllocyHVgsSJf1zO6k6o3SX2mbZjAqYE)

## Mutability

Values are immutable by default.
You must use `mut` before mutable variables.

## Shadowing

Shadowing is redefining a variable with the same name.

```rust
fn double(i: i32) -> i32 {
    i * 2
}

fn main() {
    let x = 2;
    let x = double(x);
    println!("{}", x);
}
```

It happens where you want to quickly take a variable, do something to it, and do something else again.
You don't have to come up with a new name every line.

## Stack, Heap, and Pointers

Rust stores data in two main memory areas — the stack and the heap.

The stack is very fast, but requires that the compiler knows the size of each variable at compile time.
Simple fixed-size types like `i32` go directly on the stack.

The heap is slower but can hold dynamically sized data.
For such data, Rust stores a pointer on the stack that points to the heap.

A **reference** in Rust (written as `&T`) is just a pointer — it holds the address of another value, either in the stack or the heap.

```rust
let my_number = 15;        // stored directly on the stack
let my_ref = &my_number;   // stored on the stack, contains my_number's address
```

### Invalid References

**What if you refer to an invalid stack?**

```rust
fn make_ref() -> &i32 {
    let x = 10;
    &x // ❌ x disappears after function call
}

fn main() {
    let r = make_ref();
    println!("{}", r);
    // error[E0515]: cannot return reference to local variable `x`
}
```

## More about printing

You can do more complicated printing.
(This is why `println!` is a **macro**.)

**Example:**

```rust
fn main() {
    let title = "TODAY'S NEWS";
    println!("{:-^30}", title); // no variable name, pad with -, put in centre, 30 characters long
    let bar = "|";
    println!("{: <15}{: >15}", bar, bar); // no variable name, pad with space, 15 characters each, one to the left, one to the right
    let a = "SEOUL";
    let b = "TOKYO";
    println!("{city1:-<15}{city2:->15}", city1 = a, city2 = b); // variable names city1 and city2, pad with -, one to the left, one to the right
}
```

It prints:
```
---------TODAY'S NEWS---------
|                            |
SEOUL--------------------TOKYO
```

Refer to [std::fmt documentation](https://doc.rust-lang.org/stable/std/fmt/) to know more about it.

## Strings

### String Types

- **`String`**: Sized type (pointer)
- **`str`**: Dynamic type
- **`&str`**: Sized type

### Characteristics

**`String`**: growable, owned type, you don't have to worry about other.
A `String` is a pointer, with data on the heap. It is a bit slower, but it has more functions.

```rust
let my_name = "David";              // &str
let my_name2 = "David".to_string(); // String
let my_name3 = String::from("David"); // String

// my_name.push('!');  // no
my_name2.push('!');    // ok
my_name2.push_str(" blablabla");
```

**Reallocation** happens when string grows bigger than its capacity.
To prevent it, we can use `with_capacity`:

```rust
let mut my_string = String::with_capacity(100);
```

## const and static

`const` and `static` are used for declaring **global variables**.

`static` can be used with `mut`, but very rare.

So they are almost the same. Rust programmers almost always use `const`.

Use them with **ALL_CAPITAL_LETTERS**:

```rust
const MAX_POINTS: u32 = 100_000;
static LANGUAGE: &str = "Rust";
```

## Mutable reference

### Reference Types

- **`&`**: immutable reference or shared reference
- **`&mut`**: mutable reference or unique reference

### Borrowing Rules

**Rule 1:** If you have only immutable references, you can have as many as you want. 1 is fine, 3 is fine, 1000 is fine. No problem.

**Rule 2:** If you have a mutable reference, you can only have one. Also, you can't have an immutable reference and a mutable reference together.

> **Note:** But sometimes it is ok.

**Example:**

```rust
fn main() {
    let mut number = 10;
    let number_change = &mut number; // create a mutable reference
    *number_change += 10; // use mutable reference to add 10
    let number_ref = &number; // create an immutable reference
    println!("{}", number_ref); // print the immutable reference
}
```

It knows that we didn't use `number_change` again (after immutable ref).

### Reference and shadowing

Even after shadowing the value still exists:

```rust
fn main() {
    let country = String::from("Austria"); // Now we have a String called country
    let country_ref = &country; // country_ref is a reference to this data. It's not going to change
    let country = 8; // Now we have a variable called country that is an i8. But it has no relation to the other one, or to country_ref
    println!("{}, {}", country_ref, country); // country_ref still refers to the data of String::from("Austria") that we gave it.
}
```

## Ownership

In the below case, ownership of `country` moved to the first `print_country` and disappeared:

```rust
fn print_country(country_name: String) {
    println!("{}", country_name); // country_name disappears here
}

fn main() {
    let country = String::from("Austria");
    print_country(country); // Ownership moved to here
    print_country(country); // ⚠️ Error
}
```

### A way to handle this

```rust
fn print_country(country_name: String) -> String {
    println!("{}", country_name);
    country_name // return it here
}

fn main() {
    let mut country = String::from("Austria");
    country = print_country(country);
    country = print_country(country);
    country = print_country(country);
}
```

### Much better way

```rust
fn print_country(country_name: &String) {
    println!("{}", country_name);
}

fn main() {
    let country = String::from("Austria");
    print_country(&country);
    print_country(&country);
}
```