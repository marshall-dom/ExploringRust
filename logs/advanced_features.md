# Advanced Features

## Unsafe Rust
Rust's memory safety guarantees are one of the languages most attractive features, and are
personally one of the main reasons I decided to delve into it. There is a way, however, to
escape these compile-time guarantees--Unsafe Rust. The static anlysis that Rust's compiler
perfoms is conservative by nature, and this makes sense in context of what it looks to acheive;
if you are going to make blanket guarantees about memory safety, it is better to reject some
reject code as invalid than let incorrect code slip through as valid. In cases where Rust's
static analysis may conservatively reject valid code, such as with low-level systems programming.
we can use Unsafe blocks to alert the compiler of what we are doing

### Unsafe Superpowers
- To switch to unsafe Rust, we preface a code block with the *unsafe* keyword
- Unsafe Rust allows us to perform key actions that we would otherwise be unable to
perform; these actions are referred to as *unsafe superpowers*:
    - Dereference a raw pointer
    - Call an unsafe function or method
    - Access or modify a mutable static variable
    - Implement an unsafe trait
    - Access fields of union S
- **NOTE**: The unsafe keyword does not turn off the borrow checker or disable any of Rust's
other saftey checks; it only gives you access to these additional features which will not
be checked for memory safety. For example, the use of a reference in an unsafe block will
still be checked by the compiler; this allows for some degree of safety to be maintained
even inside of unsafe blocks
- It is best practice to wrap any unsafe code within a safe abstraction and provide a safe
API

### Dereferencing a Raw Pointer
- Unsafe Rust has two raw pointer types
- Just as with traditional references, raw pointers can be either mutable or mutable. Note
that the asterisk is not the dereference operator; it is part of the type name
    - Immutable -> \*const T
    - Mutable -> \*mut T
- Unlike references and smart pointers, raw pointers:
    - Are allowed to ignore the borrowing rules by having both immutable and mutable pointers
    or multiple pointers to the same location
    - Aren't guaranteed to point to valid memory
    - Are allowed to be null
    - Do not implement any automatic cleanup
- By opting out of memory safety guarantees, we are exchanging guaranteed safety for greater
performance or the ability to interface with another language or hardware where Rust's guarantees
do not apply
- We can create raw pointers in the following way. Note that we can create raw pointers in safe
code; we just can't dereference them outside of an unsafe block. Also note that we use the *as*
keyword to cast an immutable and mutable reference into their corresponding raw pointer types;
this guarantees that they will be valid:

```rust
let mut num = 5;

let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;
```

- To access the data which the raw pointers point, we simply use an unsafe block and dereference
the raw pointers just as we would a regular pointer:

```rust
let mut num = 5;

let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;

unsafe {
    println!("r1 is: {}", *r1);
    println!("r2 is: {}", *r2);
}
```

### Calling an Unsafe Function or Method
- The second type of operation that requires an unsafe block is calls to unsafe functions
- By inserting the unsafe block around a call to an unsafe function, we are asserting to Rust
that we are aware of the potential safety issues the use of the function may present

#### Creating a Safe Abstraction over Unsafe Code
- Just because a function contains unsafe code, it does not mean we need to mark the entire
function as unsafe; in fact, in many cases it is much better to wrap unsafe code in a safe
function to provide a safe abstraction over that unsafe code
- The following code block is an example of when this abstraction could be useful. The body
of this function borrows two mutable slices from the same tuple, an operation which is
immediately marked as invalid by the compiler; however, because the two slices are not
overlapping, this is actually fundamentally OK. Note that we only wrap the code that
requires an unsafe block in an unsafe block:

```rust
use std::slice;

fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = slice.len();
    let ptr = slice.as_mut_ptr();

    assert!(mid <= len);

    unsafe {
        (slice::from_raw_parts_mut(ptr, mid),
         slice::from_raw_parts_mut(ptr.offset(mid as isize), len - mid))
    }
}
```

- We can also mark closures as unsafe; depending on the situation, structuring our code this
way may be cleaner:

```rust
use std::slice;

let address = 0x01234usize;
let r = address as *mut i32;

let slice: &[i32] = unsafe {
    slice::from_raw_parts_mut(r, 10000)
};
```

#### Using extern Functions to Call External Code
- There may be cases in whcih you need your Rust code to interact with code written in another
language; Rust's *extern* keyword allows us to do just this
- Rust's extern keyword will facilitate the creation and use of a *Foreign Function Interface
(FII)*; this is a way to define functions and enable a different language to call those functions
- The following block of code demonstrates how you would use the extern keywod to call the abs
function from C's standard library:

```rust
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```

### Accessing or Modifying a Mutable Static Variable
- Rust does not support global variables; it does, however, have *static variables* which behave
in a similar way to constants
- The names of static variables are written in screaming snake case (i.e. LIKE_THIS_RIGHT_HERE)
- The variable's type must be annotated
- Unlike constants, static variables have a fixed address in memory; using the value will always
access the same data (note that constants are allowed to duplicate their data)
- Also unlike constants, static variables can be mutable, but doing so is unsafe and thus requires
the use of an unsafe block. Note that just as with any other variable, we must mark the variable
as mutable with the mut keyword:

```rust
static mut COUNTER: u32 = 0;

fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}

fn main() {
    add_to_count(3);

    unsafe {
        println!("COUNTER: {}", COUNTER);
    }
}
```

### Implementing an Unsafe Trait
- A trait is considered unsafe when at least one of its methods has some invariant that the
compiler is unable to verify
- Traits are declared as unsafe just like anything else--by prefixing the trait with the unsafe
keyword. Note that if a trait is declared as unsafe, we must also mark all implementations of
that trait as unsafe as well:

```rust
unsafe trait Foo {
    // methods go here
}

unsafe impl Foo for i32 {
    // method implementations go here
}
```

## Advanced Traits

### Specifying Placeholder Types in Trait Definitions with Associated Types
- *Associated Types* connect a type placeholder with a trait such that the trait method
definitions can use these placeholder types in their signatures; it is then the burden of the
implementor of the trait to specify the concrete type to be used in that particular implementation
- One example of a trait with an associated type is the *Iterator* trait from the standard
library. Note that use of the *type* keyword in its definition; this placeholder type is used
to specify the relationship between the type required for the trait and the type required for its
methods:

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
```
- Associated types are similar to generics; however, associated types do not require the annotation
of types because we can't implement a trait on a particular type multiple times

### Default Generic Type Parameters and Operator Overloading
- Default type parameters are used in two main ways:
    - To extend a type without breaking existing code
    - To allow for customization in specific cases most users won't need
- We can specify a default concrete type for generic type parameters; doing so will eliminate the
need for implementors of the trait to specify a concete type if the default type will work. This
is especially useful if you plan for a particular type to be used in a majority of cases
- One example of a situation where this technique is useful is for operator overloading. Rust
does not allow for the creation of custom operators, or for the overloading of arbitrary
operators; however, we can overload operations and their traits from std::ops by providing
implementations for the traits associated with the operator. The following block demonstrates
how we could use this technique to provide an *add* operation for adding two Point instances
together:

```rust
use std::ops::Add;

#[derive(Debug, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    assert_eq!(Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
               Point { x: 3, y: 3 });
}
```

- This technique is valid because under the hood, the *Add* trait is written with generic type
parameters. Note that if no type is specified, the RHS will default to using Self as the concrete
type:

```rust
fn main() {
trait Add<RHS=Self> {
    type Output;

    fn add(self, rhs: RHS) -> Self::Output;
}
```

- We can also customize the trait to our liking. The following block of code demonstrates how we
could customize the Add trait to add the valus of two structs, each holding values of different
units:

```rust
use std::ops::Add;

struct Millimeters(u32);
struct Meters(u32);

impl Add<Meters> for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
```

### Fully Qualified Syntax for Disambiguation: Calling Methods with the Same Name
- Note that there are no rules in Rust which prevent a trait from having a method with the same
name as another trait's method, nor are there any rules that prevent you from implementing both
traits on the same type
- Note that it is also possible to implement a method directly on the type with the same name
as methods from traits
- To call methods with the same name, we must explicitly tell Rust which one we want to use;
to do this we simply access the trait or method through the type or trait via the :: syntax:

```rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}

fn main() {
    let person = Human;
    Pilot::fly(&person);
    Wizard::fly(&person);
    person.fly();
}
```

- There may be some cases, though, in which a particular method takes self as a parameter;
this can cause an issue if there are two types that implement one trait as Rust will be
unable to figure out which implementation of a trait to use based on the type of self. We
write these calls as follows:

```rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
}

fn main() {
    println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
}
```

### Using Supertraits to Require One Trait's Functionality Within Another Trait
- There may be cases in which you need one trait to use another trait's functionality. In these
cases, it is possible to specify a reliance on the dependent trait being implemented; the trait
you are relying on is referred to as a *supertrait*
- Consider the following rudimentary example of a trait called *OutlinePrint* with a method that
will print a particular value framed in asterisks; to do this we want to leverage the display
trait of the structure we are calling the trait on:

```rust
use std::fmt;

trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}
```

- If we wish to then use the trait on a particular type, we can simply provide an implementation
for the supertrait for that type:

```rust
struct Point {
    x: i32,
    y: i32,
}

use std::fmt;

impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}
```

### Using the Newtype Pattern to Implement External Traits on External Types
- We can leverage the *newtype pattern* to implement traits on external types
that we would otherwise not be able to; this is done by creating a wrapper type
and then implementing the desired trait on that wrapper rather than the external
type directly
- The following code block demonstrates how we could use this technique to implement
our own Display implementation on a vector. Note that wrap the vector in a new struct,
and then access the vector through the struct field:

```rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```

- **NOTE**: The downside of this technique is that because we are creating a new type,
we no longer have the methods of the nested type; if we wanted to access the methods of
the original type, we would have to provide implementations of those methods on our wrapper
type

## Advanced Types

### Using the Newtype Pattern for Type Safety and Abstraction
- Newtypes can be powerful tools for providing safe public APIs
- Newtypes can be used to statically enforce that values are never confused
- We can use newtypes to providing a public APIs without exposing the API of the inner type
- Newtypes can an also be used to hide internal implementation; this technique can be used
to acheive encapsulation with minimal baggage

### Creating Type Synonyms with Type Aliases
- We can create *type aliases* to give existing types another name; this is done via the
type keyword like so. Note that the alias created is a *synonym* for the existing type; it is not
a new type altogether:

```rust
type Kilometers = i32;
```

- The main use case for type synonyms is to reduce repetition. Consider a scenerio where a
particular type we are using multiple times has a lengthy signature; instead of writing
this signature each time we use the type, we can declare an alias to use instead:

```rust
type Thunk = Box<dyn Fn() + Send + 'static>;

let f: Thunk = Box::new(|| println!("hi"));

fn takes_long_type(f: Thunk) {
    // --snip--
}

fn returns_long_type() -> Thunk {
    // --snip--
    Box::new(|| ())
}
```

### The Never Type that Never Returns
- Rust has a special type named *!* that is sometimes referred to as the *empty type* because
it has no values; Rust calls this type the *never type* because it stands in the place of the
return type when a function will never return
- An example use case for the ! type is in the following diverging function; a diverging function
is one that will never return:

```rust
fn bar() -> ! {
    // --snip--
}
```

### Dyanamically Sized Types and the Sized Trait
- *Dynamically sized types*, sometimes referred to as *unsized types*, allow us to write code
using values whose size that can not be determined at compile time. These types are necessary
as Rust demands to know how much space to allocate for a particular value at compile time, so
we must have some way of specifying how it should handle values whose sizes will be determined
at runtime
- An example of a dynamically sized type is *str*. Note that we are not allowed to bind a string
slice directly to a variable of type str; this is because Rust needs to know how much memory to
allocate for any value of a particular type and this is not possible for str. The solution to this
is quite simple: instead of binding values directly to a str type, we bind the values to &str
which store the starting position and length of the slice; because we will always know the space
required to store these values, we can know alaways know the size of a &str regardless of how much
data it points to
- Rust has a particular trait called the *Sized* trait whose only purpose is to determine whether
or not a type's size is known at compile time; this trait is automatically implemented for all
types whose size can be calculated at compile time
- **NOTE**: Rust implicitly adds a bound on Sized to every generic function
- A trait bound on ?Sized is the opposite of a trait bound on Sized; the ? implies that
the trait may or may not be sized

## Advanced Functions and Closures

### Function Pointers
- Just like we can pass closures to functions, we can also pass functions to other functions
by creating a pointer to the function we want to pass; this is useful when we already have
a function that we want to use instead of defining a new closure to do the same thing
- We use the *fn* type annotation in the function signature to declare that a function
takes another function as a parameter:

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5);

    println!("The answer is: {}", answer);
}
```

- Function pointers implement all three of the closure traits (Fn, FnMut, FnOnce), so we can
always pass a function pointer as an argument for a function that expects a closure; however,
this is not true in the opposite case
- It is good practice in most cases to use a generic type and one of the closure traits so your
functions can accept either functions or closures, but there are cases in which you may want to
define a function that only accepts functions. For example, you are writing code that is meant
to interface with external code that doesn't have closures (e.g. C)

### Returning Closures
- Because closures are represented by traits, we can return them directly
- In most cases where you may want to return a trait, you can instead use the concrete type
that implements the trait as the return value of the function. Note, however, that we
cannot do this with closures because they do not have a concrete returnable type; we are not
able to use a function pointer as a return type. To get around this we use a trait object:

```rust
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```

## Macros
The term *macro* in Rust refers to a family features--*declarative* macros with *macro_rules!* and
the following three *procedural* macros:
- Custom macros that specify code added with the *derive* attribute used on structs and enums;
these use the  *#[derive]* syntax and proceed the definition
- Attribute-like macros that define custom attributes usable on any item
- Function-like macros that look like function calls but operate on the tokens specified as their
argument

### The Difference Between Macros and Functions
- Macros are a way of writing code that writes other code; this is generally referred to as
*metaprogramming*
- Macros are useful for reducing the amount of code we have to write and maintain
- Macros also offer some additional functionality that traditional functions do not:
    - While a function signature must declare the number and type of parameters a function has,
    macros can accept a variable number of parameters
    - Because macros are expanded before the compiler interprets the meaning of the code, a macro
    can implement a trait on a given type; this is not possiblw with functions as they are called
    at runtime and a trait needs to be implemented at complile time
- Despite offering some additional functionality, they have some downsides as well:
    - Because macros are essentially Rust code that writes more Rust code, their
    definitions are often much more complex than function definitions; this leads to
    their definitions generally being much more difficult to read, understand, and maintain
    - Unline functions which can be defined and called anywhere, macros must be defined or
    brought into the scope of a particular file before they are called

### Declarative Macros with macro\_rules! for General Metaprograming
- *Declarative* macros are the most commonly used form of macros in Rust
- Declarative macros allow us to write something that is similar to a match expression;
these macros can be used to create control structures, compare the resulting value
of expressions to patterns, and then run code associated with that given pattern
- To define a macro, we use the *macro_rules!* construct
- The following code block is a simplified version of the vec! macro. Note that the
*#[macro_export]* annotation indicates that the macro should be made available whenever the
crate in which the macro is defined is brought into scope. Also note that the name of the
macro is not succeeded by an ! in the definition:

```rust
// macro in use
let v: Vec<u32> = vec![1, 2, 3];

// macro definition
#[macro_export]
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```

### Procedural Macros for Generating Code from Attributes
- The second form of macros is *procedural* macros; these behave similarly to functions
- Procedural macros accept some code as an input, operate on that code, and produce some code
as an output; they do not match on patterns like other macros do
- When creating procedural macors, their definitions must be in their own crate with a special
crate type; a definition for a procedural macro looks as follows:

```rust
use proc_macro;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

### Attribute-like Macros
- Attribute-like macros are similar to custom derive macros; however, they allow us to create
new attributes rather than generating code for the derive attribute
- Attribute-like macros are also more flexibile than derive macros; they can be applied to
other items besides just structs and enums--such as functions
- The following code block is an example of how an attribute-like macro may be used. The use
of these macros are common in various frameworks--such as a web framework in this example; this
technique allows us to annotate functions with attribute macros that are defined within the
framework as a procedural macro:

```rust
#[route(GET, "/")]
fn index()
```

- This attribute may be defined in the framework in a way such as this:

```rust
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream
```

### Function-like Macros
- Function-like macros define macros that look like function calls
- Similarly to macro\_rules! macros, function-like macros offer more flexibility than functions as
they can take an unknown number of arguments
- **NOTE**: Function-like macros take a *TokenStream* parameter, and their definition manipulates
that TokenStream using Rust code just as the other procedural macro types would; this is different
from macro\_rules! macros in that macro\_rules! macros can only be defined using the match-like
syntax
- The following code block demonstrates how a function-like macro may be defined and called:

```rust
// macro call
let sql = sql!(SELECT * FROM posts WHERE id=1);

// definition
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream
```
