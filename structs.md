# Using Structs to Structure Related Data

## Defining and Instantiating Structs

- Pieces of data within structs are called fields

- The following code block demonstrates the definition of a struct:

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool
}
```

- Particular values of a struct can be accesesed via dot notation (e.g. some_user.username)

- If the struct instance is mutable, fields can be changed via dot notation as examplifed by the
following code block:

```rust
some_user.email = String::from("newemail@service.com");
```

- **NOTE** - The entire instance must be marked mutable if any values of the struct are to be
changed. Rust will not allow only certain field of the struct to be declared as mutable.

- Shorthand can be used to create new struct instances when field names are the same as variable
names. For example, a declaration within a function body may be written as follows:

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}
```

- New struct instances can be created from other instances with the following syntax:
    - This is cool!

```rust
let new_user = User {
    email: String::from("some_email@service.com"),
    username: String::from("some_username@service.com"),
    ..existing_user
};
```

### Tuple Structs

- Tuple structs look similar to tuples but behave like structs. They have no field names,
just type names. The following code block demonstrates the definition and then initialization
of a tuple struct:

```rust
struct Color(i32, i32, i32);

let black = Color(0, 0, 0);
```

### Unit-Like Structs Without Any Fields

- You can also define structs that have no fields

- This type of struct can be useful when you need to implement a trait on some type
but don't have any data that you want to store in the type itself.


### Ownership of Struct Data

- Owned types are used in struct definitions (unless lifetimes are specifically defined). This
is because we want each instance of the struct to own all of its data, thus ensuring that
the data bound to it is valid as long as the entire struct is valid


## An Example Program Using Structs

### Takeaways

- Both methods and associated functions are added to structures with impl (short for implementation) blocks

- Impl blocks are used to define an implementation of a certain function for a particular struct

- **NOTE** Rust has automatic referencing and dereferencing. This means no arrow syntax like C

- Methods take self as a parameter while associated functions do not; Associated functions are most
often used for constructors to return a new instance of the struct. Associated functions are called
via the :: syntax

- Mutiple impl blocks can be used, but are not required. One large impl block is equivalent to
multiple smaller impl blocks. This is purely a design choice.
    - I am thinking that I will most likely use multiple impl blocks to separate methods and
associated functions in my own code for the sake of clarity and readility.
