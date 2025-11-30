# EasyRust Chapters 51-60

Source: [EasyRust Playlist](https://www.youtube.com/playlist?list=PLfllocyHVgsSJf1zO6k6o3SX2mbZjAqYE)

## Hashmap

Collection of key and values
Hash map no ordering

```rust
struct City {
    name: String,
    population: HashMap<u32, u32>, // This will have the year and the population for the year
}

fn main() {

    let mut tallinn = City {
        name: "Tallinn".to_string(),
        population: HashMap::new(), 
    };

    tallinn.population.insert(1372, 3_250); // insert three dates
    tallinn.population.insert(1851, 24_000);
    tallinn.population.insert(2020, 437_619);


    for (year, population) in tallinn.population { // The HashMap is HashMap<u32, u32> so it returns a two items each time
        println!("In the year {} the city of {} had a population of {}.", year, tallinn.name, population); // random order
    }
}

```

BTreeMap is a little slower than Hashmap but has order.
Else is almost the same.

2 Ways to get values from HashMap:

```rust
    println!("{:?}", city_hashmap["Bielefeld"]); // Will panic if key doesn't exist
    println!("{:?}", city_hashmap.get("Bielefeld")); // Returns Some or None
```


Hashmap  overwrites existing keys
```rust
use std::collections::HashMap;

fn main() {
    let mut book_hashmap = HashMap::new();

    book_hashmap.insert(1, "L'Allemagne Moderne");
    book_hashmap.insert(1, "Le Petit Prince");
    book_hashmap.insert(1, "섀도우 오브 유어 스마일");
    book_hashmap.insert(1, "Eye of the World");

    println!("{:?}", book_hashmap.get(&1)); // Pass ref. You don't want to pass ownership to hashmap
    // Some("Eye of the World")
}
```

### Why insert takes value but get takes reference?

*   **`insert`**: Takes ownership (or a copy) because the HashMap needs to store the key.
*   **`get`**: Takes a reference because it only needs to look up the value. Taking ownership would consume the key.

**Note**: `i32` is `Copy`, so passing `1` to `insert` copies it. For `String`, `insert` moves the value.

```rust
// Example with non-Copy type (String)
let mut map = HashMap::new();
let key = String::from("My Key");

map.insert(key, 10); // 'key' is moved into the map

// println!("{}", key); // ⚠️ ERROR: value borrowed here after move
```

### Entry

### Entry Method

The `entry` method is a convenient way to check if a key exists in a `HashMap`. It returns an `Entry` enum, which is either `Occupied` or `Vacant`.

```rust
// Simplified view of the Entry enum
enum Entry<K, V> {
    Occupied(OccupiedEntry<K, V>),
    Vacant(VacantEntry<K, V>),
}
```

The `or_insert` method is often used with `entry`. It inserts a default value if the entry is `Vacant` and returns a mutable reference to the value (whether it was just inserted or already existed).

```rust
// Simplified view of or_insert
fn or_insert(self, default: V) -> &mut V {
    match self {
        Occupied(entry) => entry.into_mut(),
        Vacant(entry) => entry.insert(default),
    }
}
```

Here is how you can use it to count occurrences of items:

```rust
use std::collections::HashMap;

fn main() {
    let book_collection = vec!["L'Allemagne Moderne", "Le Petit Prince", "Eye of the World", "Eye of the World"];

    let mut book_hashmap = HashMap::new();

    for book in book_collection {
        let return_value = book_hashmap.entry(book).or_insert(0); // return_value is a mutable reference. If nothing is there, it will be 0
        *return_value +=1; // Now return_value is at least 1. And if there was another book, it will go up by 1
    }

    for (book, number) in book_hashmap {
        println!("{}, {}", book, number);
        /* 
            L'Allemagne Moderne, 1
            Le Petit Prince, 1
            Eye of the World, 2
        */
    }
}
```

```rust
use std::collections::HashMap;

fn main() {
    let data = vec![ // This is the raw data
        ("male", 9),
        ("female", 5),
        ("male", 0),
        ("female", 6),
        ("female", 5),
        ("male", 10),
    ];

    let mut survey_hash = HashMap::new();

    for item in data { // This gives a tuple of (&str, i32)
    // or you can destructure it like this: (gender, number)
        survey_hash.entry(item.0).or_insert(Vec::new()).push(item.1); 
        // creates a new Vec if there is nothing there and pushes the number into it
    }

    for (male_or_female, numbers) in survey_hash {
        println!("{:?}: {:?}", male_or_female, numbers);
    }
}

```

## HashSet

## HashSet

A `HashSet` is actually a `HashMap` that only has keys (no values). It is used when you want to know if something exists or not, and you don't want any duplicates.

Example:

```rust
use std::collections::HashSet;

fn main() {
    let many_numbers = vec![
        94, 42, 59, 64, 32, 22, 38, 5, 59, 49, 15, 89, 74, 29, 14, 68, 82, 80, 56, 41, 36, 81, 66,
        51, 58, 34, 59, 44, 19, 93, 28, 33, 18, 46, 61, 76, 14, 87, 84, 73, 71, 29, 94, 10, 35, 20,
        35, 80, 8, 43, 79, 25, 60, 26, 11, 37, 94, 32, 90, 51, 11, 28, 76, 16, 63, 95, 13, 60, 59,
        96, 95, 55, 92, 28, 3, 17, 91, 36, 20, 24, 0, 86, 82, 58, 93, 68, 54, 80, 56, 22, 67, 82,
        58, 64, 80, 16, 61, 57, 14, 11];

    let mut number_hashset = HashSet::new();

    for number in many_numbers {
        number_hashset.insert(number);
    }

    let hashset_length = number_hashset.len(); // The length tells us how many numbers are in it
    println!("There are {} unique numbers, so we are missing {}.", hashset_length, 100 - hashset_length);

    // Let's see what numbers we are missing
    let mut missing_vec = vec![];
    for number in 0..100 {
        if number_hashset.get(&number).is_none() { // If .get() returns None,
            missing_vec.push(number);
        }
    }

    print!("It does not contain: ");
    for number in missing_vec {
        print!("{} ", number);
    }
}
```

Use `BTreeSet` if you need the items to be ordered.

## BinaryHeap

A `BinaryHeap` is a collection that keeps the largest item at the front. It is not fully sorted, but it guarantees that when you `pop` an item, you always get the largest one.

It is good for a **priority queue**.

Example:

```rust
use std::collections::BinaryHeap;

fn main() {
    let mut jobs = BinaryHeap::new();

    // Add jobs to do throughout the day
    jobs.push((100, "Write back to email from the CEO"));
    jobs.push((80, "Finish the report today"));
    jobs.push((5, "Watch some YouTube"));
    jobs.push((70, "Tell your team members thanks for always working hard"));
    jobs.push((30, "Plan who to hire next for the team"));

    while let Some(job) = jobs.pop() {
        println!("You need to: {}", job.1);
        // You need to: Write back to email from the CEO
        // You need to: Finish the report today
        // You need to: Tell your team members thanks for always working hard
        // You need to: Plan who to hire next for the team
        // You need to: Watch some YouTube
    }

}

```

## VecDeque

If using Vec, removing the first item shifts all the other items to the left, which is slow.


```rust
use std::collections::VecDeque;

fn main() {
    let mut my_vec = VecDeque::from(vec![0; 600000]);
    for i in 0..600000 {
        my_vec.pop_front(); // pop_front is like .pop but for the front
    }
}
```

If you try to use a `Vec` for the above example, it would be very slow because removing from the front requires shifting all elements.

## Question Mark Operator (?)

The `?` operator is a shorter way to handle `Result` (and `Option`). It propagates the error (or `None`) if one occurs.

If the value is `Ok`, it unwraps it and gives you the value. If it is `Err`, it returns the error from the function immediately.

Example:

```rust
fn parse_str(input: &str) -> Result<i32, std::num::ParseIntError> { // Return type must be Result
    let parsed_number = input.parse::<i32>()?; // If it fails, it will stop here and return the error
    Ok(parsed_number)
}

fn parse_str(input: &str) -> Result<i32, ParseIntError> {
    let parsed_number = input.parse::<u16>()?.to_string().parse::<u32>()?.to_string().parse::<i32>()?; // Add a ? each time to check and pass it on
    Ok(parsed_number)
}


fn main() {
    let str_vec = vec!["Seven", "8", "9.0", "nice", "6060"];
    for item in str_vec {
        let parsed = parse_str(item);
        println!("{:?}", parsed);
    }
}
```

## Traits

Traits are like interfaces in other languages. They define functionality that a type must provide.

Example:

```rust

struct Animal { 
    name: String,
}

trait Canine { // Like an interface or abstract class
    fn bark(&self) ;

    fn run(&self) { // Default implementation
        println!("The dog is running!");
    }
}

impl Canine for Animal {
    // implement the trait
    fn bark(&self) {
        println!("Woof!");
    }

    // override the default implementation
    fn run(&self) {
        println!("My dog {} is running!", self.name);
    }
}

fn main() {
    let rover = Animal {
        name: "Pappi".to_string(),
    };

    rover.bark(); // Now Animal can use bark()
    rover.run();  // and it can use run()
}
```