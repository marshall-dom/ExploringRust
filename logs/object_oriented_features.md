# Obejct Oriented Programming Features of Rust

## Characteristics of Object-Oriented Languages

### Objects Contain Data and Behavior
One popular definition of OOP is as follows:

```
Object-oriented programs are made up of objects. An object packages both data and the
procedures that operate on that data. The procedures are typically called
methods or operations.
```

According to this definition, Rust can be classified as object-oriented; structs
and enums contain data and implementation blocks allow for methods to be associated
with those structs and enums. Rust does not consider these structures objects; however,
they still provide much of the same functionality

### Encapsulation that Hides Implementation Details
- One idea commonly associated with OOP is *encapsulation*--the idea that the implementation
details of a particular object are not accessible to the code which is using that object; to
interact with these objects we must use its public API. By separating the internals of objects
and the way we interact with them, it gives the programmer more flexibility to change and
refactor the implementation details of that object as well as protect against unwanted
changes or mutations
- We can leverage these principles in Rust by deciding whether or not to make our modules,
types, functions, and methods public or not. Everything in Rust is private by default;
we can make things public by annotating them with the *pub* keyword
- **NOTE**: Everything that we wish to be made public must be marked as such. For example,
just marking a particular struct as public will only make the struct itself publically visible;
its fields will still be private. To make its fields public, we must mark them as such
- The following is an example of one may choose to use encapsulation in Rust. Note that
the struct is public but its fields are not. Also note that the add, remove, and average
functions are public, but the update_average function is kept private:

```rust
pub struct AveragedCollection {
    list: Vec<i32>,
    average: f64,
}

pub struct AveragedCollection {
    list: Vec<i32>,
    average: f64,
}

impl AveragedCollection {
    pub fn add(&mut self, value: i32) {
        self.list.push(value);
        self.update_average();
    }

    pub fn remove(&mut self) -> Option<i32> {
        let result = self.list.pop();
        match result {
            Some(value) => {
                self.update_average();
                Some(value)
            },
            None => None,
        }
    }

    pub fn average(&self) -> f64 {
        self.average
    }

    fn update_average(&mut self) {
        let total: i32 = self.list.iter().sum();
        self.average = total as f64 / self.list.len() as f64;
    }
}
```

### Inheritance as a Type System and Code Sharing
- Inheritance is a mechanism by which one object can inherit from another object's definition,
thus gaining the parent object's data and behavior without having to redefine them
- Rust does not have inheritance; however, it does have traits which can be used to define
common behaviors. We can then provide implementations for these common behaviors for any
type

## Using Trait Objects That Allow for Values of Different Types
- One limitation of vectors is that they can only store elements of one particular type
- We can use enums to create data structures that can contain a variety of types; however, one
limitation of enums is that the variants must be predefined

### Defining a Trait for Common Behavior
There will be cases in which we want to create *extensible* types--types which are able
to be extended upon for a particular use case. For example, if we were to build a GUI library,
we may want to create a base structure called *component* that has a number of variants, such as
button, image, label, etc, but we also want the user of our library to be able to create custom
component types of their own. In a true object-oriented language this would be easy as we can use
inheritance; however, Rust does not support inheritance so we must employ different strategies to 
achieve this. The Rust way to go about this would be to create public traits for behavior
we associate with what would be that base object in an OO language, and then provide an
implementation for those behaviors on each structure we create. Then, instead of enforcing
type restrictions on our functions we can inforce implementation restrictions so that those
functions can only be called on types with implementations of said traits. An example of this
for the GUI library may be a *draw* trait which defines the logic of how that component will
be drawn and rendered on the screen. We can define this trait in our library as follows, and
then simply provide an implementation for draw to any component we wish:

```rust
pub trait Draw {
    fn draw(&self);
}
```

### Trait Objects Perform Dynamic Dispatch
- Recall that when we use trat bounds on generics, the compiler generates nongeneric
implementations of functions and methods for each concrete type that we use in place
of a generic type parameter; this is called *static dispatch*
- **NOTE**: Static Dispatch -> Compiler knows what method you are calling at compile time
- **NOTE**: Dynamic Dispatch -> Compiler can not tell which method you are calling at runtime;
in these cases the compiler will emit code that at runtime will figure out which method to call
- Trait objects require dynamic dispatch; this comes with an additional performance cost at
runtime
- Rust uses the pointers inside of the trait to determine which methods to call, which
means that these references must be looked up at runtime
- Dynamic dispatch also prevents Rust from being able to perform optimizations at compile-time,
as the compiler is unable to inline a method's code

### Object Safety is Required for Trait Objects
- We are only able to make *object-safe* traits into trait objectsl this requires adherence
to two main rules. There are more complex rules associated with object safety; however,
a trait is considered object safe if **all** of the methods defined in the trait have the following
properties:
    - The return type is **NOT** self
    - There are **NO** generic type parameters

- Trait objects must be object safe, for once you have used a particular trait object, Rust no
longer knows the concrete Self type; once this occurs there is no way the method can use the
original concrete type
- The same logic applies for the use of generic type parameters, for once they are filled with
concrete type parameters upon usage of the trait, the concrete types become part of the type that
implements the trait; once this occurs, there is no way to know which types to fill the generic
type parameters with

## Implementing an Object-Oriented Design Pattern
- The *state pattern* is an OO design pattern in which a value has some internal state represented
by a set of state objects, and the that value's behavior changes based on the internal state
- We can mimic this design pattern in Rust using structs and traits instead of objects and
inheritance
- In this design pattern, each state obejct is responsible for its own behavior and for governing
when it should change into another state; the value that holds a state object knows nothing
about the different behavior of the states or when to transition between states

### An Example Implemenation: Blog Post
Here is the functionality we would like to achieve:
1. A blog starts as an empty draft
2. When the draft is done, a review of the post is requested
3. When the post is approved, it gets published
4. Only published blog posts return content to print, so unapproved posts cannot be accidentally
published

The following represents example usage of the API we are looking to create:

```rust
use blog::Post;

fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");
    assert_eq!("", post.content());

    post.request_review();
    assert_eq!("", post.content());

    post.approve();
    assert_eq!("I ate a salad for lunch today", post.content());
}
```

#### Defining Post and Creating a New Insatnce in the Draft State
- We can begin building our library by defining a *Post* struct to represent our posts;
this should be made public. We can also implement a public *new* function for creating
new Post instances. We'll also create a private State trait for keeping track of state
and a *Draft* struct to represent drafts:

```rust
pub struct Post {
    state: Option<Box<dyn State>>,
    content: String,
}

impl Post {
    pub fn new() -> Post {
        Post {
            state: Some(Box::new(Draft {})),
            content: String::new(),
        }
    }
}

trait State {}

struct Draft {}

impl State for Draft {}
```

#### Storing the Text of the Post Content
- As part of Post's core functionality, we want to be able to add text (duh); to accomplish this,
we will implement a method add_text. By implementing this as a method we can encapsulate the
content field of the Post structure instead of exposing it as public; it will also allow us to
modify implementation details later on, such as how the data bound to the content field is read.
Note that 1) this method takes a mutable reference to self, and 2) this method has no interaction
with the state field whatsoever:

```rust
pub struct Post {
    content: String,
}

impl Post {
    // --snip--
    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }
}
```

#### Requesting a Review of the Post Changes Its State
We also need to add functionality to request to review a particular post; this should
change its state from *Draft* to *PendingReview*. To do this, we can write a public
method named *request_review* that takes a mutable reference to self; this public method
wil then call a private/internal request_review method on the current state of Post.
The internal method will be the one to consume the current state and return a new one.

Because this request_review method is a behavior to be associated with all Post states,
we add the method to the State trait. Note that the input parameter is a Box<Self> rather
than self.


```rust
pub struct Post {
    state: Option<Box<dyn State>>,
    content: String,
}

impl Post {
    // --snip--
    pub fn request_review(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.request_review())
        }
    }
}

trait State {
    fn request_review(self: Box<Self>) -> Box<dyn State>;
}

struct Draft {}

impl State for Draft {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        Box::new(PendingReview {})
    }
}

struct PendingReview {}

impl State for PendingReview {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }
}
```

#### Adding the approve Method that Changes the Behavior of content
The *approve* method will function similarly to *request_review* in that it will update the
state value when it has been been approved:

```rust
pub struct Post {
    state: Option<Box<dyn State>>,
    content: String,
}

impl Post {
    // --snip--
    pub fn approve(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.approve())
        }
    }
}

trait State {
    fn request_review(self: Box<Self>) -> Box<dyn State>;
    fn approve(self: Box<Self>) -> Box<dyn State>;
}

struct Draft {}

impl State for Draft {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        Box::new(PendingReview {})
    }

    // --snip--
    fn approve(self: Box<Self>) -> Box<dyn State> {
        self
    }
}

struct PendingReview {}

impl State for PendingReview {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }

    // --snip--
    fn approve(self: Box<Self>) -> Box<dyn State> {
        Box::new(Published {})
    }
}

struct Published {}

impl State for Published {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }

    fn approve(self: Box<Self>) -> Box<dyn State> {
        self
    }
}
}
```

### Trade-offs of the State Pattern
- Using the state pattern can be beneficial as it is easily extensible if we want to add
additional functionality; it eliminates the need for match expressions and we can add new
states by simply creating a new struct and implementing the shared traits for that struct.
- With that said, there are also a few significant downsides to this design pattern:
    - Because states implement the transitions between states, some of the states become
    couple to each other; this could become problematic if we wish to add additional states
    in between those coupled states as we would have to rewrite a good bit of code
    - Because of Rust's object saftey rules, writing duplicate logic is necessary for this
    design pattern

- Although it is possible to use this object-oriented design pattern in Rust, it fails to take
full advantage of the things that make Rust so powerful as a language; for this reason, trying
to copy and paste OO paradigms into Rust is generally unwise

#### Encoding States and Behaviors as Types
Rather than trying to encapsualte the states and transitions completely, we can instead encode
the states into different types; this will allow us to take advantage of Rust's type checking
system. The following code block is one example of how we may go about implementing this. Note
that DraftPost has no content method at all, and that Post::new returns a new DraftPost rather
than a new Post; modeling our data in this way makes it so all posts start as draft posts and
ensures that draft posts never have their content publicly available:


```rust
pub struct Post {
    content: String,
}

pub struct DraftPost {
    content: String,
}

impl Post {
    pub fn new() -> DraftPost {
        DraftPost {
            content: String::new(),
        }
    }

    pub fn content(&self) -> &str {
        &self.content
    }
}

impl DraftPost {
    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }
}
```

#### Implementing Transitions as Tranformations into Different Types
If we were to continue following this pattern, we could also create a custom type for the
transitions between states as follows. Note that the request_review method of DraftPost
creates a PendingReviewPost instance by accessing its own private content field, and that
the only method implemented for PendingReviewPost is approve, which takes self as an argument
and returns a Post. The technique allows us to transfer states without exposing the data
externally. Moreover, the request_review and approve methods take ownership of self and thus
consume the DraftPost and PendingReviewPost instances; this means that there will be no lingering
DraftPost and PendingReviewPost instances when we are done:

```rust
pub struct Post {
    content: String,
}

pub struct DraftPost {
    content: String,
}

impl DraftPost {
    // --snip--

    pub fn request_review(self) -> PendingReviewPost {
        PendingReviewPost {
            content: self.content,
        }
    }
}

pub struct PendingReviewPost {
    content: String,
}

impl PendingReviewPost {
    pub fn approve(self) -> Post {
        Post {
            content: self.content,
        }
    }
}
```

#### The Final API
By levering the type system to implement encapsulation in a way that takes better advantage
of Rust's safety guarantees, we are left with an API that can be used in a way such as this:

```rust
use blog::Post;

fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");

    let post = post.request_review();

    let post = post.approve();

    assert_eq!("I ate a salad for lunch today", post.content());
}
```

Boom! There we go.
