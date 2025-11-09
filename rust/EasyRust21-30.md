# EasyRust Chapters 21-30
Source: [EasyRust Playlist](https://www.youtube.com/playlist?list=PLfllocyHVgsSJf1zO6k6o3SX2mbZjAqYE)

## Mutable references

Two ways to pass

1. by reference

```rust
fn add_is_great(country_name: &mut String) {
    country_name.push_str(" is great");
    println!("{}", country_name);
}

fn main() {
    let mut country = String::from("Canada");
    add_is_great(&mut country);
    add_is_great(&mut country);
}
```

2. by value

```rust
fn add_is_great(mut country_name: String) { // take by value, declare as mutable
    country_name.push_str(" is great");
    println!("{}", country_name);
}

fn main() {
    let country = String::from("Canada"); // mut not needed
    add_is_great(country); // because ownership is moved here
}
```

## Copy and clone

- **copy** - trivial to copy the bytes
- **clone** - for complex types

```rust
fn prints_number(number: i32) {
    println!("{}", number);
}

fn prints_string(string: String) {
    println!("{}", string);
}

fn main() {
    let my_number = 10;
    prints_number(my_number); // no need to care about ownership
    prints_number(my_number);

    let my_string = "Austria".to_string();
    prints_string(my_string.clone()); // pass the clone so the original string is not moved
    prints_string(my_string.clone());
}
```

## Uninitialized variables and loops

In Rust this is a bad practice, but still possible to do:

```rust
fn loop_then_return(mut counter: i32) -> i32 {
    loop {
        counter += 1;
        if counter % 50 == 0 {
            break;
        }
    }
    counter
}

fn main() {
    let my_number;
    {
        // do complex calculations
        let x = loop_then_return(43);
        my_number = x;
    }
    println!("{}", my_number);
}
```

## Collection types

### Arrays

Simple and fast
Cannot add, delete, and it is hard to compare

Often used for buffers

```rust
fn main() {
    let my_array = ["one", "two", "three"]; // [&str: 3]

    let mut array = [0; 100]; // [0, 0, ..., 0]
}
```

### Slices

Dynamically sized type

```rust
fn main() {
    let seasons = ["spring", "summer", "autumn", "winter"];
    println!("{:?}", seasons[0..2]); // error, size cannot be known at compile time
    println!("{:?}", &seasons[0..2]); // ["spring", "summer"]
    println!("{:?}", &seasons[0..=2]); // ["spring", "summer", "autumn"]
    println!("{:?}", &seasons[..]); // ["spring", "summer", "autumn", "winter"]
}
```

### Vectors

Vectors are slower with more functionality

```rust
fn main() {
    let mut my_vector = Vec::new();
    my_vector.push("one");
    my_vector.push("two");
    my_vector.push("three");
    println!("{:?}", my_vector);

    let mut my_vector = vec![1, 2, 3, 4, 5];
    my_vector.push(6);
    println!("{:?}", my_vector);
}
```

Vectors also have capacity. 

Vec::with_capacity(8) will create a vector with capacity for 8 elements.
Which is faster than Vec::new() because it doesn't have to allocate memory for each element.


#### From, Into

If a type has `From`, then it also has `Into`. (but not vice versa)

```rust
fn main() {
    let my_vec = Vec::from([8, 9, 10]); // [i32; 3]
    let my_vec2: Vec<u8> = [1, 2, 3].into();
    let my_vec3: Vec<_> = [9, 0, 10].into(); // Vec<_> means "choose the Vec type for me"
                                             // Rust will choose Vec<i32>
}
```

### Tuples

```rust
fn main() {
    let random_tuple = ("Here is a name", 8, vec!['a'], 'b', [8, 9, 10], 7.7);
    println!(
        "Inside the tuple is: First item: {:?}
Second item: {:?}
Third item: {:?}
Fourth item: {:?}
Fifth item: {:?}
Sixth item: {:?}",
        random_tuple.0,
        random_tuple.1,
        random_tuple.2,
        random_tuple.3,
        random_tuple.4,
        random_tuple.5,
    )

    // Vector with tuples
    // Vec<(&str, i32)>
    let my_vec = vec![("hey", 1), ("hey2", 2)];

    // Destructuring
    let str_tuple = ("one", "two", "threee"); // Also works with array
    let (a, _, _) = str_tuple;
}
```

## Control Flow

You can use `if` and `else`, but in Rust people love to use match

```rust
fn main() {
    let my_number = 5;
    let second_number = match my_number {
        0 => 0,
        5 => 10,
        _ => 2,
    };
}
```

Also you can use it with tuples:

```rust
fn main() {
    let sky = "cloudy";
    let temperature = "warm";

    match (sky, temperature) {
        ("cloudy", "cold") => println!("It's dark and unpleasant today"),
        ("clear", "warm") => println!("It's a nice day"),
        ("cloudy", _) => println!("Cloudy and something else"),
        _ => println!("Not sure what the weather is."),
    }
}
```

You can put `if` inside. This is called a match guard:

```rust
fn main() {
    let children = 5;
    let married = true;

    match (children, married) {
        (children, married) if married == false => println!("Not married with {} children", children),
        (c, m) if c == 0 && m => println!("Married but no children"),
        _ => println!("Married? {}. Number of children: {}.", married, children),
    }
}
```
