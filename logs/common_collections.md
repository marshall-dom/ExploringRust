#Common Collections

## Overview

Some basic, commonly used Rust collection types:

- A vector allows you to store a variable number of values next to each other

- A string is a collection of characters

- A hash map allows you to associate a value with a particular key; it is a
particular implementation of the more general data structure called a map


## Storing Lists of Values with Vectors

- Vectors can only store values of the same type

- When initializing a new, empty vector, we must explicity define its type; however,
Rust can infer its type if a vector is initialized with values:

```rust
let v: Vec<i32> = Vec::new();

let v = vec![1, 2, 3];
```

### Reading Elements of Vectors

- Data can be accessed from vectors either by indexing or the get method
    - Indexing will return a reference to the element at that index of the vector
        - This will panic if we try to access a nonexistent element
    - Get() will return an Option type
        - This will allow us to handle a potential error more gracefully

- Rust compiler enforces strict ownership rules on collections, so we must be
thoughtful in our management of references. This means we are unable to modify
a collection while there are valid references to any of its elements, not just
the vector itself

- Vectors are iterable by default and can be iterated over with simple loops

- Mutable vectors can be modified through loops, but require we first dereference
to obtain the value at that index. This would look something like the following:

```rust
let  mut v = vec![1, 2, 3, 4];
for i in &mut v {
    *i += 50;
}
```

### Using an Enum to Store Multiple Types

- While it is true that vectors can only store one type, enums are defined as a single type
despite being able to store different types in their variants. This means we can use enums
to create vectors of multiple types. Very cool!

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec! [
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```

- Rust needs to know what types will be in the vector at compile time so it knows exactly how
much memory will be needed to store each element on the heap. Using enums in combination with a
match expression means that the compiler can ensure each possible case is handled


## Storing UTF-8 Enconded Text with Strings

- Strings are often overlooked by programmers as being more simple than they actually are.
Don't do this! They are actually pretty complex collections

- Strings are implemented as a collection of bytes, with methods to interpret those bytes as
text

### What is a String?

- The only string type in Rust's core language is the string slice "str"; string literals
are also considered string slices here.

- The String type provided by Rust's standard library is a "growable, mutable, ownded, UTF-8
encoded string type"

- There are also other string types included in Rust's standard library, such as:
    - OsString
    - OsStr
    - CString
    - CStr

### Updating a String

- Like vectors, Strings can grow in size and its contents can change

- Data can be added to the string via the push method. The method push_str takes a str
slice because we don't want to take ownership of the parameter. The push method will assume
ownership of whatever it consumes. For example:

```rust
let mut s1 = String::from("foo");
let s2 = "bar";
s1.push_str(s2); // s2 is consumed here
println!("s2 is {}", s2); // therefore this is invalid
```

- Concatonation can be done via the + operator or the format! macro. When concatonating strings,
we always use the following syntax. Note that the add method for strings takes a String and
a string reference, and that s1 will no longer be valid as it will be moved to s3. Also note that
we pass an &String even though the method actually receives &str as a parameter. This is because
the Rust compiler can coerce the &String argument into a &str (i.e. it uses a deref coercion to
turn &s2 into &s2[..]):

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2;
```

- The behavior of the + operator becomes more difficult to use when concatonating more than two
string objects. When joining more than two strings, it is recommended to use the format! macro

### Indexing into Strings

- **NOTE**: Rust Strings do not support indexing!

- It is important to understand that String objects in Rust are really just wrappers over a
Vec<u8> object where each character is encoded as a UTF-8 character. This means a few things:
    1. Strings are manipulated in a similar way to vectors
    2. String objects behave differently than one may expect, as the elements of the elemnts in
    the vector are UTF-8 values, not the characters themselves (e.g. "3" is really encoded as
    208 in the vector)

- There are three relevant ways to look at strings from Rust's perspective:
    1. bytes
    2. scalar values
    3. grapheme clusters (closest thing to "letters")

- Another reason that Rust does not allow string indexing is that indexing operations are
expected to always take constant time; this is not guaranteed with strings because Rust would
first have to walk through the contents of the string from the beginning to the index in order
to determine how many valid characters there are

- While you can not index strings using a single number, you can, however, index a string using
a range. This is still not recommended for most use cases, though, as it could cause a panic
at runtime if an attempt was made access an invalid index

- The recommended way to access the elements of a string is through the chars method, which
separates and returns the char values of the string. You can then iterate over these values
like so:

```rust
for c in some_string.chars() {
    println!("{}", c);
}
```

- A similar method exists for accessing the raw bytes of a string:

```rust
for b in some_string.bytes() {
    println!("{}", b);
}
```

## Storing Keys with Associated Values in Hash Maps

### Creating a New Hash Map

- Hash maps can be created via the new function, and elemtents can be added via the
insert function. Note that hash maps are not included in the prelude and therefore must
be brought into scope via use:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

- Hash Maps are homogenoues, meaning all keys must have the same type and all values must
have the same type

- Hash maps can also be constructed by using the collect method on a vector of tuples,
where each tuple consists of a key and its value. The collect method is often used in
tandom with the zip method, which will create a vector of tuple pairs (i.e. "zipping
them together"). The following code block demonstrates a possible use case of this.
Note the type annotation HashMap<_, _>. This is needed beacuse it is possible to collect
into a number of different data structures and Rust requires you to specify type; however,
when we use underscores Rust will infer the types contained in the map based on the data
in the vectors:

```rust
use std::collections::HashMap;

let teams  = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];

let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();
```

### Hash Maps and Ownership

- For types that implement the Copy trait, the values will be copied into the hash map;
however, for owned values (like Strings), the values will be moved and the hash map
will assume ownership of that value. This means that the variable previously bound to that
data will become invalid upon insertion

- It is possible to insert references to values into a hash map rather than the value itself,
but this presents its own set of challenges. If this route is chosen, we must guarantee that
the values to which the references point are valid for the entire lifetime of the hash map


### Accessing Values in a Hash Map

- Values can be retreived from a Hash Map by providing its key to the get method:

```rust
let some_key = String::from("Key");
let some_value = some_map.get(&some_key);
```

- Note that the get method returns an Option type; this is done to guard against a panic
in the event that no value is stored in the hash map for that key. This means that a valid
result will be wrapped in Some, and if there's no value for that key the get method will return
None. Any code written to access the hash map will need to handle the Option

- We can also iterate over each key/value pair in a similarly to vectors:

```rust
for (key, value) in &some_map {
    println!("{}: {}", key, value);
}
```

### Updating a Hash Map

- While the number of keys and values is growable, each key can only have one associated value
at a time. This means that when you wish to update the hash map, you must handle the case in which
a key already has a value assigned to it

- By default, an attempt to insert a value into a key with an existing value will simply overwrite
the old value with the new one

- There is a built in method for hash maps called entry, which checks whether a
particular key has a value. The return type of entry is an enum called Entry that represents
a value that may or may not exist. We can chain insert commands to the entry method like so:

```rust
some_map.entry(String::from("Key")).or_insert(50);
```

- The or_insert method on Entry returns a mutable referfence to the corresponding key passed into
entry if that key exists, or inserts the parameter as the new value for the key if it does not exist

- The entry method can also be used to update a key based on its old value. The following code block
demonstrates how this strategy may be used to update a map acting as a counter:

```rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

// if the word already has a count, increment the count
// else, insert 0 for the word
for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
```

- **NOTE**: Rust allows you to specify wish hashing algorithm is used for a hash map. Multiple
hashers are built into the standard library or you can implement your own hasher
