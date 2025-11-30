# EasyRust Chapters 41-50

Source: [EasyRust Playlist](https://www.youtube.com/playlist?list=PLfllocyHVgsSJf1zO6k6o3SX2mbZjAqYE)


## Enum match


`match` allows you to compare a value against a series of patterns and execute code based on which pattern matches. When used with Enums, it can destructure the variant to access the data inside.

```rust
enum AnimalType {
    Cat(String),
    Dog(String),
}

fn main() {
    let my_animal = AnimalType::Cat(String::from("Fluffy"));
    match my_animal {
        AnimalType::Cat(name) => println!("The cat's name is {}", name),
        _ => println!("It's not a cat"),
    }
}
```


### Destructuring
```rust
struct Person { // make a simple struct for a person
    name: String,
    real_name: String,
    height: u8,
    happiness: bool
}

struct Person2 {
    name: String,
    height: u8,
}

impl Person2 {
    fn from_persion(input: Person) -> Self {
        let Person {name, height, ..} = input; // destructure input
        Self {
            name,
            height,
        }
    }
}

fn main() {
    let papa_doc = Person { // create variable papa_doc
        name: "Papa Doc".to_string(),
        real_name: "Clarence".to_string(),
        height: 170,
        happiness: false
    };

    let Person { // destructure papa_doc
        name: a,
        real_name: b,
        height: c,
        happiness: d
    } = papa_doc;

    println!("They call him {} but his real name is {}. He is {} cm tall and is he happy? {}", a, b, c, d);

    let person2 = Person2::from_persion(papa_doc);
}
```

## Deref 

The dot operator (.) performs auto-referencing, dereferencing and coercion until types match! 

```rust
struct Item {
    number: u8,
}

impl Item {
    fn compare_number(&self, other_number: u8) { // takes a reference to self
        println!("Are {} and {} equal? {}", self.number, other_number, self.number == other_number);
            // We don't need to write *self.number
    }
}

fn main() {
    let item = Item {
        number: 8,
    };

    let reference_item = &item; // This is type &Item
    let reference_item_two = &reference_item; // This is type &&Item

    item.compare_number(8); // the method works
    reference_item.compare_number(8); // it works here too
    reference_item_two.compare_number(8); // and here

}
```

## Generics


Generics allow you to write flexible definitions for functions and structs that can work with any type. You use a placeholder (like `T`) instead of a specific type.

```rust
fn return_number<T>(number: T) -> T { // <T> is a must
    println!("Here is your number.");
    number
}

fn return_number(number: MyType) -> MyType { // Error
    println!("Here is your number.");
    number
}

fn print_number<T: Debug>(number: T) { // <T: Debug> is the important part
    println!("Here is your number: {:?}", number);
}

```

### Trait Bounds
You can restrict generics to types that implement specific traits. This ensures the type has the functionality you need (like printing or comparing).

```rust
use std::fmt::Display;
use std::cmp::PartialOrd;

fn compare_and_display<T: Display, U: Display + PartialOrd>(statement: T, num_1: U, num_2: U) {
    println!("{}! Is {} greater than {}? {}", statement, num_1, num_2, num_1 > num_2);
}

// the same as above
fn compare_and_display<T, U>(statement: T, num_1: U, num_2: U)
where
    T: Display,
    U: Display + PartialOrd,
{
    println!("{}! Is {} greater than {}? {}", statement, num_1, num_2, num_1 > num_2);
}


fn main() {
    compare_and_display("Listen up!", 9, 8);
}
```

## Option

The `Option` enum is used when a value might be something or nothing. It replaces `null` in other languages. It has two variants: `Some(T)` (contains a value) and `None` (no value).

```rust 

    // ⚠️
fn take_fifth(value: Vec<i32>) -> i32 {
    value[4]
}

fn main() {
    let new_vec = vec![1, 2];
    let index = take_fifth(new_vec); // panic!
}


```
Panic means that the program stops before the problem happens. Rust sees that the function wants something impossible, and stops. It "unwinds the stack" (takes the values off the stack) and tells you "sorry, I can't do that".


```rust
// ⚠️
fn take_fifth(value: Vec<i32>) -> Option<i32> {
    if value.len() < 5 {
        None
    } else {
        Some(value[4])
    }
}

fn main() {
    let new_vec = vec![1, 2];
    let bigger_vec = vec![1, 2, 3, 4, 5];
    println!("{:?}, {:?}",
        take_fifth(new_vec).unwrap(), // this one is None. .unwrap() will panic!
        take_fifth(bigger_vec).unwrap()
    );


// better way to handle None
    match take_fifth(new_vec) {
        Some(number) => println!("Here is your number: {}", number),
        None => println!("There is no number here!"),
    }

// another way
        let inside_number = take_fifth(vec);
        inside_number.is_some() {
            println!("Here is your number: {}", inside_number.unwrap());
        }


}
```

## Result
Result is similar to Option, but here is the difference:

Option is about Some or None (value or no value),
Result is about Ok or Err (okay result, or error result).
So Option is if you are thinking: "Maybe there will be something, and maybe there won't." But Result is if you are thinking: "Maybe it will fail."


```rust

fn check_if_five(number: i32) -> Result<i32, String> {
    match number {
        5 => Ok(number),
        _ => Err("Sorry, the number wasn't five.".to_string()), // This is our error message
    }
}

fn main() {
    let mut result_vec = Vec::new(); // Create a new vec for the results

    for number in 2..7 {
        result_vec.push(check_if_five(number)); // push each result into the vec
    }

    println!("{:?}", result_vec);
}
```

### if let and while let
`if let` is syntax sugar for a `match` that runs code only if the value matches a specific pattern.
`while let` works like `if let` but for loops: it runs the loop body as long as the pattern matches.

```rust
fn main() {
    let weather_vec = vec![
        vec!["Berlin", "cloudy", "5", "-7", "78"],
        vec!["Athens", "sunny", "not humid", "20", "10", "50"],
    ];
    for mut city in weather_vec {
        println!("For the city of {}:", city[0]); // In our data, every first item is the city name
        while let Some(information) = city.pop() {
            // This means: keep going until you can't pop anymore
            // When the vector reaches 0 items, it will return None
            // and it will stop.
            if let Ok(number) = information.parse::<i32>() {
                // Try to parse the variable we called information
                // This returns a result. If it's Ok(number), it will print it
                println!("The number is: {}", number);
            }  // We don't write anything here because we do nothing if we get an error. Throw them all away
        }
    }
}
```