# Enums and Pattern Matching

## Defining an Enum

- Enums ar useful when there are multiple possibilities, but only one
can be true at any given time

- The following code block could be used to describe an IP address,
which has two possible types:

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

- Instances of each variant can be created as follows:

```rust
let four = IpAddrKind::V4;
let six = IpAddrKind::V6;
```

- Variants of enums are namespaced under its identifier and accessed via
the :: syntax

- The use of enums in function definitions because the same function can then
be used for any variant of that enum. For example, we may define a route function
as follows:

```rust
fn route(ip_kind: IpAddrKind) { }
```

- Enums are even more useful for defining struct fields, as demonstrated by the
following code block:

```rust
struct IpAddr {
    kind: IpAddrKind,
    address: String,
}
```

- This code can be made even more efficient by modifying the enum variants to include
type names. If written this way, we can eliminate the need for a struct all together:

```rust
enum IpAddr {
    V4(String),
    V6(String),
}

let home = IpAddr::V4(String::from("127.0.0.1"));
```

- Furthermore, each variant can have different types and amounts of associated data.
This allows us to also write our IP Address enum in the following way:

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));
```

- As it turns out, Rust's standard library already has structs for IP Adresses; however,
enums still are useful here as you can add the address type to the enum definition:

```rust
enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

- Enums can hold any kind of data, including other enums. Take for example this Message enum:

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32},
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

- This message eum has four variants with four different types:
    1. Quit has no associated data
    2. Move includes an asynchronus struct inside it
    3. Write includes a single String
    4. ChangeColor includes three i32 values

- Using an enum here instead of multiple structs allows for more flexibility through
the use of implementations. Instead of definining an implementation for each struct
individually, one implentation can be defined for the entire enum.

### The Option Enum

- The option type is an enum defined by Rust's standard library used when a particular
value could either be something or nothing

- Unlike many other languages, Rust has no Null value; instead it has enum that can
encode the concept of a value being absent

- The Option<T> enum is defined by the standard library as follows:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

- The following are some examples of using Option values to hold number types
and String types:

```rust
let some_number = Some(5);
let some_string = Some("a string");

let absent_number: Option<i32> = None;
```

- **NOTE:** Rust does not know how to add types and option types. In order for these
to work nicely together, option types must first be converted to types

- When using Option<T>, you generally want to have code to handle each variant; this is often
done via matching


## The Match Control Flow Operator

- Match allows you to compare a value against a series of patterns and then execute code based
on which pattern matches. This is similar to a switch statement in other languages.

- The following code block is an example of a match statement:

```rust
enum Coin {
    Penny,
    Nickle,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickle => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25, }
}
```

- The variants of a match statment in Rust are called arms and are comprised of two parts:
    1. A pattern
    2. Some code that executes if that pattern is matched


### Patterns That Bind to Values

- Match arms can also bind to the parts of the values that match the pattern. Values can be
extracted out of enum variants as demonstrated in the following code block:

```rust
enum UsState {
    Alabama,
    Alaska,
    ...
}

enum Coin {
    Penny,
    Nickle,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
         Coin::Penny => 1,
        Coin::Nickle => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {}!", state);
            25
        }
    }
}
```

- By adding a variable called state to the pattern that matches quarters, when a quarter
matches, the state variable will bind to the value of that quater's state. This variable
can then be used in the code for that arm

### Matching with Option<T>

- Match is commonly used to compare the variants of Option<T>, returning one value if there
is a value and another if the value is absent

- The following code block demonstrates what this may look like:

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

### Matches Are Exhaustive

- In Rust matches are exhaustive, meaning that every case must be handled in order to be valid

- The _ pattern will match any value, and can therefore be used in match statments as a default
case:

```rust
let value = 0u8;
match value {
    1 => println!("One!"),
    2 => println!("Two!"),
    _ => (),
}
```

## Concise Control Flow with If Let

- Rust allows for the matching of single patterns to be handled with the if let keyword. For
example, the code block following the if let statement will only trigger if the pattern is
matched:

```rust
if let Some(3) = some_value {
    println!("Three!");
}
```

- If let statements can also be used in conjuction with else staements. This is more concise
than a match statement if you are only matching on one pattern and a default
