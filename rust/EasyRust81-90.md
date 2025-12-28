# EasyRust Chapters 81-90

Source: [EasyRust Playlist](https://www.youtube.com/playlist?list=PLfllocyHVgsSJf1zO6k6o3SX2mbZjAqYE)

## Iterator Methods: any() and all()

The `any()` method returns `true` if any element in the iterator satisfies the predicate. The `all()` method returns `true` if all elements satisfy the predicate.

```rust
fn in_char_vec(char_vec: &Vec<char>, check: char) {
    println!("Is {} inside? {}", check, char_vec.iter().any(|&char| char == check));
}

fn main() {
    let char_vec = ('a'..'働').collect::<Vec<char>>();
    in_char_vec(&char_vec, 'i');
    in_char_vec(&char_vec, '뷁');
    in_char_vec(&char_vec, '鑿');

    let smaller_vec = ('A'..'z').collect::<Vec<char>>();
    println!("All alphabetic? {}", smaller_vec.iter().all(|&x| x.is_alphabetic()));
    println!("All less than the character 행? {}", smaller_vec.iter().all(|&x| x < '행'));
}
```

## Reverse Iterator

The `rev()` method reverses an iterator, allowing you to iterate through a collection from the end to the beginning.

```rust
fn main() {
    let num_vec = vec![1, 2, 3, 4, 5];
    
    for number in num_vec.iter().rev() {
        println!("{}", number);
    }
    // Output: 5, 4, 3, 2, 1
}
```

## find() and position()

The `find()` method returns the first element that matches a predicate, wrapped in `Some`, or `None` if no element matches. The `position()` method returns the index of the first matching element, wrapped in `Some`, or `None` if no element matches.

**Difference:**
- `find()` returns the **value** itself
- `position()` returns the **index** (position) of the value

```rust
fn main() {
    let num_vec = vec![10, 20, 30, 40, 50, 60, 70, 80, 90, 100];

    println!("{:?}", num_vec.iter().find(|&number| number % 3 == 0)); // find takes a reference, so we give it &number
    println!("{:?}", num_vec.iter().find(|&number| number * 2 == 30));

    println!("{:?}", num_vec.iter().position(|&number| number % 3 == 0));
    println!("{:?}", num_vec.iter().position(|&number| number * 2 == 30));
}
```

**Output:**
```
Some(30)  // This is the number itself
None      // No number inside times 2 == 30
Some(2)   // This is the position (index)
None
```

## String And &str

In Rust, `String` and `&str` are two different string types with distinct characteristics:

### String
- **Owned**: `String` owns its data and is stored on the heap
- **Growable**: Can be modified (append, remove, etc.)
- **Mutable**: Can change its contents
- **Size**: Has a known size at compile time (pointer, length, capacity)

### &str
- **Borrowed**: `&str` is a string slice - a reference to a string
- **Immutable view**: Cannot modify the string it points to
- **Flexible**: Can point to a `String`, a string literal, or part of a string
- **No ownership**: Does not own the data

### Deref Coercion

Functions that accept `&str` can also accept `&String` because `String` implements `Deref` to `str`. This automatic conversion is called **deref coercion**.

```rust
fn prints_str(my_str: &str) { // it can use &String like a &str
    println!("{}", my_str);
}

fn main() {
    let my_string = String::from("I am a string");
    let string_literal = "I am a string slice";
    
    prints_str(&my_string);      // &String is automatically converted to &str
    prints_str(string_literal);  // string literal is already &str
    prints_str(&my_string[0..5]); // string slice from String
}
```

### When to Use Each

**Use `String` when:**
- You need to own the string data
- You need to modify or grow the string
- You're building a string dynamically

**Use `&str` when:**
- You only need to read the string
- You want to accept both `String` and string literals
- You're writing function parameters (more flexible)

### Memory Layout

```rust
fn main() {
    // String: stored on the heap, owns the data
    let owned = String::from("Hello");
    
    // &str: reference to string data (could be from String or string literal)
    let borrowed: &str = "World";
    let borrowed_from_string: &str = &owned;
    
    // String slice: reference to part of a string
    let slice: &str = &owned[0..3]; // "Hel"
}
```

**Key points:**
- `&String` can be automatically converted to `&str` through deref coercion
- This allows functions to accept both `&str` and `&String` without explicit conversion
- `&str` is more flexible and should be preferred in function signatures when you don't need ownership
- String literals (like `"hello"`) are of type `&str` and are stored in the program's binary

### String vs &str vs &mut str

**When to use `String`:**
- You need to own and modify the string (grow, shrink, append)
- You're building a string dynamically
- You need to return a string from a function
- Example: `let mut s = String::from("hello"); s.push_str(" world");`

**When to use `&str`:**
- You only need to read the string (most common case)
- Function parameters that accept both `String` and string literals
- You want maximum flexibility without ownership
- Example: `fn print_text(text: &str) { ... }`

**When to use `&mut str`:**
- You need to modify characters in-place but **cannot change the length**
- You're working with a fixed-size buffer
- You need to modify part of a string without reallocating
- Example: Converting characters to uppercase in-place: `text.make_ascii_uppercase()`
- **Note**: `&mut str` is less common and more restrictive than `String` - you can only modify existing characters, not add or remove them

### Why can't &mut str change the length?

`&mut str` cannot change the length of a string because:

1. **Memory Layout**: `&mut str` is a reference to a **fixed-size memory region**. It points to a specific slice of bytes in memory, and that slice's size is determined when the reference is created.

2. **No Ownership**: A reference doesn't own the underlying buffer. The actual string data is owned by a `String` (or stored as a string literal). The reference can only modify the bytes within its view, but cannot:
   - Allocate new memory
   - Reallocate the buffer
   - Change the buffer's capacity

3. **Safety**: Allowing length changes through a reference would require reallocation, which could invalidate other references to the same string, violating Rust's memory safety guarantees.

```rust
fn main() {
    let mut s = String::from("hello");
    let slice: &mut str = &mut s;
    
    // ✅ This works: modify existing characters
    slice.make_ascii_uppercase();
    println!("{}", slice); // "HELLO"
    
    // ❌ This doesn't work: cannot change length
    // slice.push_str(" world"); // ERROR: no method named `push_str` found
    
    // To change length, you need the String itself:
    s.push_str(" world"); // ✅ Works because s is a String
    println!("{}", s); // "HELLO world"
}
```

**Summary:**
- `&mut str` = mutable reference to a **fixed-size** string slice
- `String` = owned, growable string that can change length
- To modify length, you need ownership (`String`), not just a reference (`&mut str`)

```rust
fn modify_in_place(text: &mut str) {
    // Can modify characters but not length
    // Using safe method to convert to uppercase
    text.make_ascii_uppercase();
}

fn main() {
    let mut s = String::from("hello world");
    modify_in_place(&mut s); // Can pass &mut String as &mut str
    println!("{}", s); // "HELLO WORLD"
    
    // If you need to grow/shrink, use String instead:
    let mut s2 = String::from("hello");
    s2.push_str(" world"); // This requires String, not &mut str
    println!("{}", s2); // "hello world"
}
```
