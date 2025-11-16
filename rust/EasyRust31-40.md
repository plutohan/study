# EasyRust Chapters 31-40

Source: [EasyRust Playlist](https://www.youtube.com/playlist?list=PLfllocyHVgsSJf1zO6k6o3SX2mbZjAqYE)

## More Control Flow

### Match error cases

All branches in a `match` expression must return the same type. Rust enforces type consistency, so you cannot return different types from different match arms.

```rust
// match has to return the same type
fn main() {
    let my_number = 10;
    let some_variable = match my_number {
        10 => 8,
        _ => "Not ten", // ERROR
    };
}

// This doesn't work either!
fn main() {
    let my_number = 10;
    let some_variable = if my_number == 10 { 8 } else { "something else" }; // ERROR
    // Note: Rust doesn't have a ternary operator, so use if-else expressions like above
}
```

### Match with @ (binding)

The `@` operator allows you to bind a value to a variable while also testing it against a pattern. This is useful when you want to capture the matched value for use in the match arm.

```rust
fn match_number(input: i32) {
    match input {
        number @ 1..=10 => println!("{} is between 1 and 10", number),
        _ => println!("It's greater than 10"),
    }
}
```

## Structs

Structs are custom data types that let you group related data together. Rust has three types of structs: unit structs, tuple structs, and named structs.

### Unit Struct

A unit struct is a struct with no fields, taking up 0 bytes of memory. They're useful for implementing traits or as marker types.

```rust
struct FileDirectory;
```

### Tuple Struct

A tuple struct is a struct with unnamed fields. You access the fields using tuple syntax with indices (`.0`, `.1`, etc.).

```rust
struct Colour(u8, u8, u8);

fn main() {
    let my_colour = Colour(50, 0, 50);
    println!("The second part of the colour is: {}", my_colour.1); // Use tuple syntax to access fields
}
```

### Named Struct

A named struct has fields with explicit names. Rust allows field shorthand when variable names match field names.

```rust
struct Country {
    population: u32,
    capital: String,
    leader_name: String
}

fn main() {
    let population = 50_000_000;
    let capital = String::from("Seoul");
    let leader_name = String::from("Lee");

    let korea = Country {
        population, // Field shorthand: same as `population: population`
        capital,    // Field shorthand
        leader_name,
    };
}
```

## Enums

Enums (enumerations) allow you to define a type by enumerating its possible variants. Each variant can optionally hold data of different types.

### Basic Enum

Here's a simple enum with variants that don't hold any data:

```rust
enum ThingsInTheSky {
    Sun,   // These are called variants
    Stars,
}
```

### Enums with Data

You can add data to enum variants. Each variant can hold different types and amounts of data.

```rust
enum ThingsInTheSky {
    Sun(String), // Now each variant has a string
    Stars(String),
}

fn create_skystate(time: i32) -> ThingsInTheSky {
    match time {
        6..=18 => ThingsInTheSky::Sun(String::from("I can see the sun!")), // Daytime hours
        _ => ThingsInTheSky::Stars(String::from("I can see the stars!")), // Nighttime
    }
}

fn check_skystate(state: &ThingsInTheSky) {
    match state {
        ThingsInTheSky::Sun(description) => println!("{}", description),
        ThingsInTheSky::Stars(n) => println!("{}", n),
    }
}

fn main() {
    let time = 8; 
    check_skystate(&create_skystate(time));
}
```

### Enums as Integers

Enums without data can be cast to integers. Each variant gets a number starting from 0 by default.

```rust
enum Season {
    Spring, // If this was Spring(String), casting wouldn't work! Only works with enums without data
    Summer,
    Autumn,
    Winter,
}

fn main() {
    use Season::*; // Import all variants so we can use them without the Season:: prefix
    let four_seasons = vec![Spring, Summer, Autumn, Winter];
    for season in four_seasons {
        println!("{}", season as u32);
    }
}
```

### Custom Enum Values

You can assign custom integer values to enum variants. Subsequent variants without explicit values will increment from the last assigned value.

```rust
enum Star {
    BrownDwarf = 10,
    RedDwarf = 50,
    YellowStar = 100,
    RedGiant = 1000,
    DeadStar, // Automatically becomes 1001 (previous value + 1)
}
```

### Enums with Different Types

Enum variants can hold different types of data, making them very flexible for representing values that can be one of several types.

```rust
enum Number {
    U32(u32),
    I32(i32),
}

fn get_number(input: i32) -> Number {
    let number = match input.is_positive() {
        true => Number::U32(input as u32), // Change it to u32 if it's positive
        false => Number::I32(input), // Otherwise just give the number because it's already i32
    };
    number
}

fn main() {
    let my_vec = vec![get_number(-800), get_number(8)];

    for item in my_vec {
        match item {
            Number::U32(number) => println!("It's a u32 with the value {}", number),
            Number::I32(number) => println!("It's an i32 with the value {}", number),
        }
    }
}
```

## Loops

Rust provides several loop constructs: `loop` (infinite loop), `while` (conditional loop), and `for` (iterator loop). Loops can be named, broken out of, and can even return values.

### Named Loops

In Rust, you can give loops labels (names) to control nested loops more precisely. This is especially useful when you need to break out of an outer loop from within an inner loop.

```rust
fn main() {
    let mut counter = 0;
    let mut counter2 = 0;
    println!("Now entering the first loop.");

    'first_loop: loop {
        // The loop label starts with a single quote '
        counter += 1;
        println!("The counter is now: {}", counter);
        if counter > 9 {
            // Starts a second loop inside this loop
            println!("Now entering the second loop.");

            'second_loop: loop {
                // Now we are inside 'second_loop
                println!("The second counter is now: {}", counter2);
                counter2 += 1;
                if counter2 == 3 {
                    break 'first_loop; // Break out of the outer 'first_loop, skipping 'second_loop's label
                }
            }
        }
    }
}
```

### While and For Loops

While `loop` creates infinite loops, `while` and `for` loops are more commonly used for iteration with conditions or over collections.

```rust
// while example
fn main() {
    let mut counter = 0;

    while counter < 5 {
        counter += 1;
        println!("The counter is now: {}", counter);
    }
}

// for example
fn main() {
    // Exclusive range (0..3 means 0, 1, 2)
    for number in 0..3 {
        println!("The number is: {}", number);
    }

    // Inclusive range (0..=3 means 0, 1, 2, 3)
    for number in 0..=3 {
        println!("The next number is: {}", number);
    }

    // Using _ when you don't need the loop variable
    for _ in 0..3 {
        println!("Printing the same thing three times");
    }
}
```

### Returning Values from Loops

The `break` keyword can return a value from a loop, making loops expressions that evaluate to a value.

```rust
fn main() {
    let mut counter = 5;
    let my_number = loop {
        counter += 1;
        if counter % 53 == 3 {
            break counter; // Return the counter value from the loop
        }
    };
    println!("{}", my_number); // Prints: 56
}
```
## Implementation Blocks (impl)

The `impl` keyword allows you to define methods and associated functions for structs and enums. This is how you add behavior to your custom types.

**Two types of functions in impl blocks:**
- **Associated functions** (known as "static" methods in some languages): Don't take `self` as a parameter. Most often used to create new instances (constructors).
- **Methods**: Take `self` (or `&self` or `&mut self`) as the first parameter. Used to operate on instances of the type.

```rust
#[derive(Debug)]
struct Animal {
    age: u8,
    animal_type: AnimalType,
}

#[derive(Debug)]
enum AnimalType {
    Cat,
    Dog,
}

impl Animal {
    // Associated function (constructor) - called as Animal::new()
    fn new() -> Self { // Self is an alias for Animal
        Self {
            // When we write Animal::new(), we always get a 10-year-old cat
            age: 10,
            animal_type: AnimalType::Cat,
        }
    }

    // Method that modifies the instance - requires &mut self
    fn change_to_dog(&mut self) {
        println!("Changing animal to dog!");
        self.animal_type = AnimalType::Dog;
    }

    fn change_to_cat(&mut self) {
        println!("Changing animal to cat!");
        self.animal_type = AnimalType::Cat;
    }

    // Method that only reads the instance - requires &self
    fn check_type(&self) {
        match self.animal_type {
            AnimalType::Dog => println!("The animal is a dog"),
            AnimalType::Cat => println!("The animal is a cat"),
        }
    }
}

// You can have multiple impl blocks for the same type
// This is useful for organizing code, though this example just demonstrates the syntax
impl Animal {
    fn check_type_again(&self) {
        match self.animal_type {
            AnimalType::Dog => println!("The animal is a dog"),
            AnimalType::Cat => println!("The animal is a cat"),
        }
    }
}


fn main() {
    let mut new_animal = Animal::new(); // Associated function to create a new animal

    // Calling methods on the instance
    new_animal.check_type(); // Syntactic sugar for Animal::check_type(&new_animal)
    new_animal.change_to_dog();
    new_animal.check_type();
    new_animal.change_to_cat();
    new_animal.check_type();
}
```