# Generic Types, Traits, and Lifetimes

- Generics are abstract stand-ins for concrete types or other properties
- Generics and traits are two methods of avoiding duplication in our code
- Lifetimes are a particular type of generics which allow us to give the compiler
information about how references relate to each other. By default, Rust's compiler
is incredibly strict in how it enforces shared memory; lifetimes allow us to
borrow values that we would not be allowed by default by providing additional
information needed for trhe compiler to check that those refrences are valid
throughout that lifetime

## Generic Data Types

### In Function Definitions

To define a function using generics, we place the generics in the signature of the
functions in place of the data types in the parameters and return values; this provides
us a lot of additional flexibility when structuring our code. Instead of defining separate
functions for different types, we can define one function using generics. Consider a function
that returns the largest item in a collection. We could write a function that returns the largest
integer in a vector of integers and another function that returns the largest float in a
vector of floats, or we could define one funtion using generic data types.

### In Struct Definitions

Similar methodology can be applied to the definition of structs. This allows us to reuse
custom structures to hold various data types. Consider a structure which represents a coordinate
point and holds number values in its fields. Instead of defining a point structure for i32 values
and another for f64 values, we could define a point structure that uses generic data types.

### In Enum Definitions

Just like structs, we can also use generic data types in enum definitions to allow their
variants to hold various types. An example of this is the Option<T> enum or Result<T,E> enum
from the standard library.

### In Method Definitions

We can also implement methods on structs and enums using generics by using generic types in
their definitions.

### Performance of Code Using Generics

- Rust's implementation of generics allows for the use of generics with no runtime performance
hits!
    - This will impact compile times, however, becuase Rust achieves this by preprocessing the
    generic code and converting into specific code at compile time


## Traits: Defining Shared Behavior

- Traits are used to define shared behavior in an abstract way. They tell the Rust compiler
about functionality a particular type has and can share with other types. Additionally, we
can use trait bounds to specify that a generic can be any type that has a certain behavior

- Traits are similar to interfaces in other OO languages, but have some key differeces

### Defining a Trait

- A types behavior is defined by the methods that we can call on that type. By this methodology,
we can say that different types share the same behavior if we can call the same methods on
those types. Trait definitions provide a way to group method signatures together to define
a set of behaviors necessary to accomplish some purpose. We use the following syntax to define
a trait. Note that there is no function body:

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

### Implementing a Trait on a Type

Once a trait is defined, we can define an implementation of that trait for any type that we
wish to give that behavior. Defining a trait allows us to write better APIs because they allow
for better consistency across our codebase by naming behaviors instead of just operations.
Consider a the following example of a media aggregator in which you want to put together content
from multiple sources. Although the summary of a tweet and the summary of a news article may contain
slightly different information, we can abstract away the behavior of providing that summary and
address those differences in implementation for each individual type while still maintaining
a level of consistency:

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}

pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

It is also possible to provide default implementations for traits; you could override this by
specifying a specific implementation for a particular type later.

### Traits as Parameters

- Traits can be used as parameters in function definitions instead of types. This is helpful when
defining generic functions because it allows us to enforce rules about which types the function
will accept. This is done with the following syntax:

```rust
pub fn notify<T: Summary>(item: T) {
    println!("Breaking news! {}", item.summarize());
}
```

- This can also be done with a shorthand syntax if the signature is less complex:

```rust
pub fn notify(item: impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

- Additionally, we can specify more than one trait bound with the + syntax:

```rust
pub fn notify(item: impl Summary + Display) {}
```

- To enforce complex trait bounds, we can use where clauses. This allows us to define complex trait
bounds in a more readible way:

```rust
fn some_function<T, U>(t: T, u: U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{}
```

### Returning Types that Implement Traits

For the same reasons you may want to enforce trait bounds on functions, you may also want
to specify that return types of functions implement specific traits. This is done with similar
syntax, only now we are replacing the return type with the trait bound instead of the parameters:

```rust
fn returns_summarizable(switch: bool) -> impl Summary {}
```

### Using Trait Bounds to Conditionally Implement Methods

- We can implement methods conditionally for types that implement particular traits by
enforcing a trait bound on an impl block that uses generic type parameters:

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

- Similarly, we can also conditionally implement traits on specific types:

```rust
impl<T: Display> ToString for T {}
```

## Validating References with Lifetimes

Every reference in Rust has a lifetime; however, most of the time lifetimes are implicit
and inferred by the compiler. Sometimes, though, it is necessary to provide further information
to the compiler about the liftimes of references to ensure that they will, with absolute cetainty,
be valid at runtime.

### Preventing Dangling References with Lifetimes

- One of the primary purposes of lifetimes are to prevent dangling references; these will cause
a program to unintentially reference incorrect data at runtime

### The Borrow Checker

- Rust's compiler has a borrow checker that compares scopes to determine whether all borrows
in your program are valid

### Lifetime Annotation Syntax

- Just as functions can accept any type when the signature specifies a generic type parameter,
functions can accept references with any lifteime by specifying a generic type parameter

- Liftime annotations do not change how long any of the references live; they simply describe the
relationships of the lifetimes of multiple references to each other

- The lifetime annotation syntax looks as follows, with an apostrophe followed by a lowercase name
for the lifetime (usually lowercase and very short):

```rust
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```

- It should be noted that a single liftime reference annotation has no significance; the purpose
of the annotation is to give explicit information regarding the relationship between multiple
references. Consider the following example that enforces a constraint such that both of its
parameters as well as its return value must have the same lifetime, 'a:

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

- It should be noted that this in no way changes the lifetimes of any of the values passed in
or returned. We are merely specifying that the compiler should reject any values that do not
conform to the imposed restraints

- Note that liftime annotations are added to the function signature but not the function body.
This is because the compiler is able to deduce the liftimes of any references within the scope
of the function; it is only when outside references are brought into the scope of the function
that annotations are necessary

### Thinking in Terms of Lifetimes

- The way in which you need to specify lifetime parameters depends on what your function is
doing

- When writing functions in Rust, we need to consider the parameter and return types, as well as
the lifetimes

- We use Rust's lifetime syntax to connect the lifetimes of parameters to return values

### Lifetime Annotations in Struct Definitions

- It is possible to define structs to hold references rather than to hold owned types; however,
to do this we must specify the lifetimes of those references by adding a lifetime annotation to
every reference in the struct's definition

- By explicitly defining lifetimes for structs and their fields, we can ensure that referces
held by the struct are not outlived by the struct itself. Without providing this additional
information to the compiler, you would create the possiblity for a failure at runtime. Consider
the following struct definition in which we enforce the constraint that the reference held
in the field "part" lives at least as long as the struct itself:

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}
```

### Liftime Annotations in Method Definitions

- When implementing methods on a struct with liftetimes, we use the same syntax as that of
generic type parameters discussed before

- Where we declare and use the lifetime parameters depends on whether they are related to the
struct fields or the method parameters and return values

- Lifetime names for struct fields are always declared immediately after the impl keyword and
are then used again after the struct's name:

```rust
impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```

### The Static Lifetime

- One special lifetime is the 'static lifetime, which means that the reference can live for the
entire duration of the program

- **NOTE**: All string literals have the 'static lifetime. This is because they are stored
directly in the binary and are therefor always available

- The use of the static lifetime is generally discouraged, as there are few instances where
the reference you are thinking about marking as static actually lives through the duration
of the program. In other words, do not just attempt to resolve errors thrown by the compiler
by slapping the static lifetime on your references. This will most likely lead to dangling
references and breakage later on
