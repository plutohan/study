# EasyRust Chapters 61-70

Source: [EasyRust Playlist](https://www.youtube.com/playlist?list=PLfllocyHVgsSJf1zO6k6o3SX2mbZjAqYE)

## Traits

Traits are like interfaces in other languages. They define functionality that a type must provide.

Example:

```rust
use std::fmt;

struct Cat {
    name: String,
    age: u8,
}

impl fmt::Display for Cat {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        let name = &self.name;
        let age = self.age;
        write!(f, "{name} is a cat who is {age} years old.")

        // You can also write it like this:
        // write!(f, "{} is a cat who is {} years old.", self.name, self.age)
    }
}

fn main() {
    let mr_mantle = Cat {
        name: "Reggie Mantle".to_string(),
        age: 4,
    };

    println!("{}", mr_mantle);
}
```


### Trait Bounds

Trait bounds specify that a generic type must implement certain traits. You can combine multiple trait bounds using `+`.
```rust
use std::fmt::Debug;
#[derive(Debug)]
struct Monster {
    health: i32,
}
#[derive(Debug)]
struct Wizard {
    health: i32,
}
#[derive(Debug)]
struct Ranger {
    health: i32,
}

trait Magic{}
trait FightClose {}
trait FightFromDistance {}

impl FightClose for Ranger{} 
impl FightClose for Wizard {}
impl FightFromDistance for Ranger{} 
impl Magic for Wizard{}  


// You can also use `where` clause: fn attack_with_bow<T>(character: &T, ...) where T: FightFromDistance + Debug
fn attack_with_bow<T: FightFromDistance + Debug>(character: &T, opponent: &mut Monster, distance: u32) {
    if distance < 10 {
        opponent.health -= 10;
        println!(
            "You attack with your bow. Your opponent now has {} health left.  You are now at: {:?}",
            opponent.health, character
        );
    }
}

fn attack_with_sword<T: FightClose + Debug>(character: &T, opponent: &mut Monster) {
    opponent.health -= 10;
    println!(
        "You attack with your sword. Your opponent now has {} health left. You are now at: {:?}",
        opponent.health, character
    );
}

fn fireball<T: Magic + Debug>(character: &T, opponent: &mut Monster, distance: u32) {
    if distance < 15 {
        opponent.health -= 20;
        println!("You raise your hands and cast a fireball! Your opponent now has {} health left. You are now at: {:?}",
    opponent.health, character);
    }
}

fn main() {
    let radagast = Wizard { health: 60 };
    let aragorn = Ranger { health: 80 };

    let mut uruk_hai = Monster { health: 40 };

    attack_with_sword(&radagast, &mut uruk_hai);
    attack_with_bow(&aragorn, &mut uruk_hai, 8);
    fireball(&radagast, &mut uruk_hai, 8);
}

```

### From Trait

The `From` trait allows you to define how to create one type from another. When you implement `From<T>` for your type, you automatically get the `Into<T>` trait for free (this is called a blanket implementation).

Example of implementing `From`: 

```rust
#[derive(Debug)]
struct City {
    name: String,
    population: u32,
}

impl City {
    fn new(name: &str, population: u32) -> Self {
        Self {
            name: name.to_string(),
            population,
        }
    }
}
#[derive(Debug)] 
struct Country {
    cities: Vec<City>, 
}

// Note: We can implement From<Vec<City>> (not just From<City>).
// This works because Country is our local type (see Orphan Rule below).
impl From<Vec<City>> for Country {
    fn from(cities: Vec<City>) -> Self {
        Self { cities }
    }
}

impl Country {
    fn print_cities(&self) { // function to print the cities in Country
        for city in &self.cities {
            // & because Vec<City> isn't Copy
            println!("{:?} has a population of {:?}.", city.name, city.population);
        }
    }
}

fn main() {
    let helsinki = City::new("Helsinki", 631_695);
    let turku = City::new("Turku", 186_756);

    let finland_cities = vec![helsinki, turku];
    let finland = Country::from(finland_cities);

    // Or use .into() - this works because implementing From gives you Into for free:
    // let finland: Country = finland_cities.into();

    finland.print_cities();
}
```

### About The Orphan Rule

The comment `So we can also implement on a type that we didn't create` refers to Rust's **Orphan Rule**.

**Definition**: You can implement a trait for a type only if **at least one of them (the trait or the type) is defined in your current crate**.

In the code above:
*   `From` (the trait) is from the standard library (**External**).
*   `Vec` (the type used in the trait parameter) is from the standard library (**External**).
*   **However**, `Country` (the type we are implementing the trait *for*) is defined in our code (**Local**).

Because `Country` is local, we are allowed to implement the external `From` trait for it. If both were external (e.g., trying to implement `Display` for `Vec<String>`), the compiler would forbid it to prevent conflicts.

**Why does this rule exist?**

1.  **Coherence (Avoiding Conflicts):** It prevents multiple crates from implementing the same external trait for the same external type. Without this rule, the compiler wouldn't know which implementation to use if two libraries defined conflicting behaviors.
2.  **Future Compatibility:** It protects your code from breaking when dependencies update. It ensures that if an upstream crate (like the standard library) adds a trait implementation in the future, it won't clash with an implementation you defined locally.

### Blanket Trait Implementations

A blanket implementation lets you implement a trait for all types that satisfy certain bounds. The syntax `impl<T> Trait for T` means "implement this trait for every type T".

```rust
use std::fmt::{Debug, Display};

trait Prints {
    fn debug_print(&self) where Self: Debug{
        println!("I am: {:?}", self);
    }
    fn display_print(&self) where Self: Display{
        println!("I am: {}", self);
    }
}

#[derive(Debug)]
struct Person;

impl<T> Prints for T {

}

fn main() {
    let my_person = Person;
    my_person.debug_print();

    let my_string = String::from("Hello");
    my_string.debug_print();
    my_string.display_print();
}
```

### AsRef Trait

The `AsRef<T>` trait allows you to write functions that accept any type that can be cheaply converted to a reference of type `T`. This is useful for writing flexible APIs that accept both `&str` and `String`.

```rust
use std::fmt::Display;

fn print_it<T: AsRef<str> + Display>(input: T) {
    println!("{}", input)
}

fn main() {
    print_it("Please print me");
    print_it("Also, please print me".to_string());
    // print_it(7); <- This will not print
}
```

## Chaining methods

Chaining methods is a common pattern in Rust, where you can call multiple methods together in a single statement.

Imperative style (non-functional)

```rust
fn main() {
    let mut new_vec = Vec::new();
    let mut counter = 1;

    while counter < 11 {
        new_vec.push(counter);
        counter += 1;
    }

    println!("{:?}", new_vec);
}
```


Functional style

```rust
fn main() {
    let new_vec = (1..=10).collect::<Vec<i32>>();
    // Or you can write it like this:
    // let new_vec: Vec<i32> = (1..=10).collect();
    println!("{:?}", new_vec);
}
```

## Iterator

An iterator is a type that implements the `Iterator` trait, which requires a `.next()` method that returns `Option<Item>`. Iterators are lazy - they don't do anything until you consume them.

There are three ways to create an iterator from a collection:
- `.iter()` - iterates over `&T` (borrows)
- `.iter_mut()` - iterates over `&mut T` (mutable borrows)
- `.into_iter()` - iterates over `T` (takes ownership)

```rust
fn main() {
    let vector1 = vec![1, 2, 3];
    let vector1_a = vector1.iter().map(|x| x + 1).collect::<Vec<i32>>();
    // vector1 is still usable here because .iter() only borrows
    let vector1_b = vector1.into_iter().map(|x| x * 10).collect::<Vec<i32>>();
    // vector1 is now consumed and can't be used anymore

    let mut vector2 = vec![10, 20, 30];
    vector2.iter_mut().for_each(|x| *x += 100); // modifies in place

    println!("{:?}", vector1_a);
    println!("{:?}", vector2);
    println!("{:?}", vector1_b);
}
```

### How .next() Works

An iterator works by using the `.next()` method, which returns `Some(item)` for each element, and `None` when exhausted.

```rust
fn main() {
    let my_vec = vec!['a', 'b', '거', '柳'];

    let mut my_vec_iter = my_vec.iter(); // Creates an iterator (lazy, nothing happens yet)

    assert_eq!(my_vec_iter.next(), Some(&'a'));  // First item
    assert_eq!(my_vec_iter.next(), Some(&'b'));  // Second item
    assert_eq!(my_vec_iter.next(), Some(&'거')); // Third item
    assert_eq!(my_vec_iter.next(), Some(&'柳')); // Fourth item
    assert_eq!(my_vec_iter.next(), None);        // No more items
    assert_eq!(my_vec_iter.next(), None);        // Calling .next() after exhaustion always returns None
}
```