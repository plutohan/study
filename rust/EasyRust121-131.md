# EasyRust Chapters 121-131

Source: [EasyRust Playlist](https://www.youtube.com/playlist?list=PLfllocyHVgsSJf1zO6k6o3SX2mbZjAqYE)

## The Default Trait

The `Default` trait allows you to define a default value for a type. This is very useful when you want to create a struct with sensible default values, or when you need to initialize a value before you know what to put in it.

### Deriving Default

For many simple types, you can just derive `Default`. This will use the default value for each field (e.g., 0 for numbers, "" for Strings, false for bools).

```rust
#[derive(Debug, Default)]
struct MyStruct {
    number: i32,
    text: String,
    is_active: bool,
}

fn main() {
    let s = MyStruct::default();
    println!("{:?}", s); 
    // Output: MyStruct { number: 0, text: "", is_active: false }
}
```

### Implementing Default Manually

If you want custom default values, you can implement the trait manually.

```rust
struct Character {
    name: String,
    age: u8,
    height: u32,
    weight: u32,
    lifestate: LifeState,
}

#[derive(Debug)]
enum LifeState {
    Alive,
    Dead,
    NeverAlive,
    Uncertain,
}

impl Default for Character {
    fn default() -> Self {
        Self {
            name: "Billy".to_string(),
            age: 15,
            height: 170,
            weight: 70,
            lifestate: LifeState::Alive,
        }
    }
}

fn main() {
    let billy = Character::default();
    println!("Name is {}", billy.name); // Name is Billy
}
```

## Operator Overloading (The Add Trait)

In Rust, you can overload operators like `+`, `-`, `*`, etc., by implementing traits from `std::ops`. For example, implementing the `Add` trait allows you to use the `+` operator on your custom types.

```rust
use std::ops::Add;

#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

// Implement Add for Point + Point
impl Add for Point {
    type Output = Point; // Result type of the addition

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 1, y: 2 };
    let p2 = Point { x: 3, y: 4 };
    
    let p3 = p1 + p2; // Uses Point::add
    println!("{:?}", p3); // Point { x: 4, y: 6 }
}
```

## The Builder Pattern

The Builder Pattern is a common design pattern in Rust used to construct complex objects. It's especially useful when:
1.  You have a struct with many fields.
2.  Most fields have sensible default values.
3.  You want to enforce validation during construction.

It typically involves a separate `Builder` struct that holds the configuration, and a `build()` method that creates the final object.

```rust
#[derive(Debug)]
struct Character {
    name: String,
    age: u8,
    height: u32,
    weight: u32,
    lifestate: LifeState,
}

#[derive(Debug)]
enum LifeState {
    Alive,
    Dead,
    NeverAlive,
    Uncertain,
}

#[derive(Debug)]
struct CharacterBuilder {
    name: String,
    age: u8,
    height: u32,
    weight: u32,
    lifestate: LifeState,
}

impl Default for CharacterBuilder {
    fn default() -> Self {
        Self {
            name: "Billy".to_string(),
            age: 15,
            height: 170,
            weight: 70,
            lifestate: LifeState::Alive,
        }
    }
}

impl CharacterBuilder {
    // Methods consume 'self' and return 'Self' to allow method chaining
    fn with_name(mut self, name: &str) -> Self {
        self.name = name.to_string();
        self
    }
    
    fn with_age(mut self, age: u8) -> Self {
        self.age = age;
        self
    }
    
    fn with_height(mut self, height: u32) -> Self {
        self.height = height;
        self
    }
    
    fn with_weight(mut self, weight: u32) -> Self {
        self.weight = weight;
        self
    }
    
    fn build(self) -> Result<Character, &'static str> {
        if self.age < 0 { 
            // Note: u8 is never < 0, but serves as a validation example
            Err("Age cannot be negative")
        } else if self.height > 300 {
             Err("Character is too tall!")
        } else {
            Ok(Character {
                name: self.name,
                age: self.age,
                height: self.height,
                weight: self.weight,
                lifestate: self.lifestate,
            })
        }
    }
}

fn main() {
    // Usage: Chain methods to configure the builder, then call build()
    let character_1 = CharacterBuilder::default()
        .with_name("Sloth")
        .with_age(20)
        .with_height(180)
        .build();

    println!("{:?}", character_1);
}
```
