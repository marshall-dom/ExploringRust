# Managing Growing Projects with Packages, Crates, and Modules

- Rust's _module system_ includes:
    - **Packages**: A Cargo feature that lets you build, test, and share crates
    - **Crates**: A tree of modules that produces a library or executable
    - **Modules** and **Use**: Let you   control the organization, scope, and privacy of paths
    - **Paths**: A way of naming an item, such as a struct, function, or module

## Packages and Crates

- A crate is a binary or library

- The create root is a source file that the Rust compiler starts from and makes up the
root module of your crate
 
- A package is one or more crates that provides a set of functionality, and contains a
Cargo.toml file that describes how to build those crates

- Crate roots of binary crates are named main.rs by convention; crate roots of library
crates are named lib.rs by convention

- It is recommended that a crate's functionality is kept within its own scope. This prevents
potential conflicts (i.e. namespaces, etc.)


## Defining Modules to Control Scope and Privacy

- Modules allow you to organize code within a crate into groups for readability and easy reuse.
Modules also control the privacy of items

- Modules can also contain nested modules

- Modules are defined by the mod keyword in src/lib.rs as follows:

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}
```

- The tree structure for this project would look something like the following:

```
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

## Paths for Referring to an Item in the Module Tree

- A path can take two forms:
    - An absolute path (starting from the crate root)
        - Uses a crate name or literal
    - A relative path (starting from the current module)
        - Uses self, super, or an identifier

- Both path types are followed by one or more identifiers separated by ::

- We can expose a public api for our crate as follows. Note that hosting AND its functions must
be marked as public so it can be accessed from the other function:

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

- The keyword super can also be used to construct relative paths. The super keyword
points to the parent of the current module. This would like like the following:

```rust
fn serve_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::serve_order();
    }
    
    fn cook_order() {}
}
```

### Making Structs and Enums Public

- The pub keyword can be used to make structs public; however, this will only make the
struct itself public. The structs fields are made public individually. This means you
could write something like the following:

```rust
mod back_of_house {
    pub struct Breakfast {
        pub toast: String.
        seasonal_fruit: String,
    }
}
```

- In contrast, if enums are marked as public, then all of its variants are marked as public

## Bringing Paths into Scope with the Use Keyword

- Instead of writing the absolute or relative path each time we want to call a function
defined in a particular module, we can bring that module into scope by using the use keyword:

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
```

- When bringing a function into scope it is idiomatic to specify its parent module; however,
when bringing in structs, enums, and other items with use, it is idiomatic to specify the entire
path

- The as keyword can be used to rename something from another module in the current scope

- Names can be re-exported with the keywords pub use

- Nested paths are helpful for readability when bringing multiple items from the same module into
scope:

```rust
use std::{cmp::Ordering, io};
```

- The glob operator is used to bring in all items from a particular path:

```rust
use std::collections::*;
```

## Separating Modules into Files

- If we wish to separate modules into separate files, we can use a semicolon rather than
a block tells Rust to load the contents of the module from another file with the same
name as the module:

```rust
pub mod hosting;
```

- We can continue this by mimicking the module tree structure with the directory structure
of the project. For example, the definitions for the hosting module could be placed in a
file called "src/front_of_house/hosting.rs"
