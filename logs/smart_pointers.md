# Smart Pointers
- Smart pointers are data structures that not only hold addresses to a particular piece of memory,
but also have additional metadata and capabilities
- In Rust, smart pointers often own the data they point to
- Smart pointers are usually implemented as structs; the characteristic that distinguishes a smart
pointer from other ordinary structs is the implentation of the Deref and Drop traits
- The Deref trait allows an instance of the smart pointer struct to behave like a reference
- The Drop trait allows you to customize the code that executes when an instance of the smart
pointer goes out of scope and is "dropped" from memory
- Some of the most commonly used smart pointers in the standard library are:
    - Box<T> for allocating values on the heap
    - Rc<T> for reference counting with allowance for multiple ownership
    - Ref<T>, RefMut<T>, and RefCell<T> to enforce borrowing rules at runtime instead of
    compile time

## Using Box<T> to Point to Data on the Heap
- Boxes allow you to store data on the heap instead of on the stack
- Boxes have no performance overhead other than the typical overhead associated with storing
data on the heap
- Boxes are most often used in the following situations:
    - When you have a type whose size can not be known at compile time and you wish to use
    a value of that type in a context that requires an exact size
    - When you have a large amount of data and you want to transfer ownership but ensure the
    data will not be copied when you do so
    - When you want to own a value and you care only that it is a type that implements a
    particular trait rather than being of a specific type

- Transfering ownership of a large amount of data can take a long time because the data is copied
around on the stack. Storing that data in a box makes this operation more efficient, as only a
the small amount of pointer data is copied around on the stack while the data it references
stays static on the heap
- The syntax for initializing a new box looks as follows. The value b will be stored on the stack,
but point to some_value which is allocated on the heap. We can now interact with the box similar
to how we would interact with any other value that is stored on the stack. Also, just like any
other owned value, the box value along with the data it points to will be deallocated once
the box goes out of scope:

```rust
let b = Box::new(some_value);
```

### Enabling Recursive Types with Boxes
- Rust needs to know how much space a particular type takes up at compile time; however,
it is impossible to determine the size of a recursive type at compile time as this
nesting of values could theoretically continue infinitely. By inserting a box in a recursive
type definition, we can get around this problem

### Computing the Size of a Non-Recursive Type
To determine how much space will need to be allocated for a particular data type, Rust
will iterate through each of the type's variants and calculate the space needed to store
the type for that particular variant by summing the required space from that type to bedrock.
In other words, if one variant of the the type holds a multi-dimensional vector or another
custom data type, it will sum the space needed with consideration of each level. This is the
heart of the issue in computing the size of a recursive type, for it is not possible to
reach bedrock; each round of calculation will just lead to another round of calculation one
level deeper

### Using Box<T> to Get a Recursive Type with a Known Size
Smart pointers allow us to circumvent the issue of not being able to calculate space, as the
size of a pointer will not be impacted by the amount of data it points to. This means
instead of nesting recursive types directly, we can wrap them in smart pointers and nest
those pointers inside one another; this implementation will be behave as if you just placed
the pointers next to each other on the stack, instead of nesting them inside one another. Rust's
implementation of boxes provide a method of indirection and heap allocation to accomplish just
this

## Treating Smart Pointers Like Regular References with the Deref Trait
- The behavior of the dereference operator can be customized by providing an implementation of
the deref trait
- It is possible to implement the Deref trait in a way that will allow you treat a smart pointer
like you would a regular reference; this is useful as it allows you to operate on smart pointers
with the same code you would use to operate on regular references

### Following the Pointer to the Value with the Dereference Operator
When working with regular references, we may use the dereference operator in the following
manner. In this example, we bind a variable x to a value and then bind another variable y
to a reference to the variable x; this is a common paradigm. In order to access the value
bound to y, though, we must first dereference it:

```rust
let x = some_value;
let y = &some_value;

assert_eq!(some_value, *y);
```

### Defining Our Own Smart Pointer
- The Box<T> type is defined as a tuple struct in the standard library, so we will
use a similar structure for our example:

```rust
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}
```

- In order to make our custom box type behave like the standard box structure, we
need to implement the Deref trait. The only method required to implement the Deref trait
is deref which borrows self and returns a reference to the data held in the structure; this
is necessary so that when we can return the value that the box points to when we wish
to deference the box. Also note that we must define the associated type for the trait
to use by passing in the type of the box:

```rust
use std::ops::Deref;

struct MyBox<T>(T);
impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}
```

### Implicit Deref Coercions with Functions and Methods
Deref coercion converts a reference to a type that implements Deref into a reference
to a type that Deref can convert the original type into; this happens automatically in
Rust when a reference to a particular type's value is passed as an argument to a function
or method that doesn't match the parameter type in the function of method definition. Rust
does this through a sequence of calls to the deref method; in the end the type we provided
will have been converted into the type the parameter needs

The following is an example application of the concept. Here, store a String in our
MyBox type and then attempt to call a function that takes a string slice. Rust will
perform deref coercion under the hood to make this call possible without the need
for additional derefercing and casting:

```rust
fn hello(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m);
}
```

### How Deref Coercion Interacts with Mutability
- Just as we can use the Deref trait to override the deference operator on immutable references,
we can use the DerfMut trait to override the deference operator on mutable references
- Rust performs deref coercion when it finds types and trait implementations in the following
cases:
    - From &T to &U when T: Deref<Target=U>
    - From &mut T to &mut U when T: DerefMut<Target=U>
    - From &mut T to &U when T: Deref<Target=U>
- Rust will coerce a mutable reference to an immutable one; however, it is impossible to
perfrom the reverse operation. Immutable references will never coerce to mutable references;
this makes logical sense given the borrowing rules. While there can be multiple immutable
references to the same piece of data, there can only be one mutable reference to any particular
data at one time. Rust can coerce a mutable reference to an immutable one because it can
be guarantee that no borrowing rules will be broken by creating a new immutable reference; however,
Rust cannot guarantee that creating a new mutable reference to a piece of data will not break
these rules

## Running Code on Cleanup with the Drop Trait
- The second integral trait to the smart pointer is Drop, which allows us to customize what code
is executed when a particular value is about to go out of scope
- A custom implementation for the Drop trait can be added to any type
- Unlike some other lower-level langauges that require the programmer to manually manage the
freeing of memory and resources after each use of a smart pointer, Rust will handle this
automatically provided that you specify the code you wish to be executed when a particular
value goes out of scope. When the value goes out of scope, Rust will call the predefined
drop method

### Dropping a Value Early with std::mem::drop
- There are scenarios in which you may want to clean up a value early
    - Ex: if you are using a smart pointer to manage locks, you may want to call drop early
    to release the lock so other code in the same scope can acquire it
- Rust does not allow you to call the drop method directly; this must be done through
std::mem::drop
- The std::mem::drop function is different from the drop method; we call it by passing the
value we wish to be dropped as an argument:

```rust
let pointer = some_smart_pointer {};
drop(pointer);
```

## Rc<T>, the Reference Counted Smart Pointer
- Most of the time, ownership is clear in Rust; however, we can use the reference counting type
to enable multiple ownership of values; Rc<T> keeps track of the number of references to determine
if the value is still in use. When there are no longer any references to that value, the value
will be cleaned up
- **NOTE**: Rc<T> can only be used in single-threaded scenarios. Reference counting is handled
differently in multithreaded scenarios

### Using Rc<T> to Share Data
We can use the refernce counted smart pointer to implement shared ownership--a feat
that is not allowed under normal circumstances by the compiler. To understand how
Rc affords us this ability, let's first examine the issue it resolves. For this we will
use the cons list as an example:

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}
```

In this implementation, the variants of Cons have ownership of the data they hold; therefore,
something like the following is not allowed. This is because when when a is bound to the first
Cons, a assumes ownership of the Cons along with its data. Likewise, when a is embedded in the
Cons that is bound to b, b assumes ownership of a. Due to ownership rules, the next binding
to c is invalid, as b already owns a:

```rust
let a = Cons(5,
    Box::new(Cons(10,
        Box::new(Nil))));
let b = Cons(3, Box::new(a));
let c = Cons(4, Box::new(a));
```

#### So What?
Now, we could alter the definition of Cons to hold references to its variants rather than owning
them; however, this wouldn't do much in the way of solving our shared ownership problem. If we
change the Cons structure to hold references, we would then have to specify and enforce strict
lifetime constraints; this would make it so that every element in the Cons list would have to live
as long as the entire list itself. This is no bueno, for the structure utilizes temprorary nil
values and these values are dropped before any reference is made to them--breaking the lifetime
contraints

#### The Real Solution
The better method of resolving this issue is to use Rc<T> instead of Box<T> to hold the list
references; doing so means that we can clone the list instead of transfering ownsership. Note that
the call to clone here does not generate a deep copy of the data, it only increments the reference
count; this is much faster than making deep copies and allows us to write more performant code
while still reaping most of the benefits of shared ownership:

```rust
let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
let b = Cons(3, Rc::clone(&a));
let c = Cons(4, Rc::clone(&a));
```

### Cloning an Rc<T> Increases the Reference Count
- When an Rc<T> is initialize its reference count is 1
- Each time Rc::clone() is called, the count is incremented by 1
- When the drop trait is called on one of the clones, the count is automatically
decremented by 1 to reflect this change
- When the reference count reaches 0, the Rc<T> is cleaned up automatically
- **NOTE**: The nature of Rc<T> prohibits it to be used with mutable references; this
would violate Rust's borrowing rules

## RefCell<T> and the Interior Mutability Pattern
Mutating data that is currently pointed to by immutable references is forbidden by Rust's
borrowing rules; however, Rust has a design pattern known as *Interior Mutability* that allows
us to mutate this kind of data through the use of unsafe code. We leverage Rust's interior
mobility pattern to guarantee that borrowing rules will not be broken at runtime, even
though the compiler can not check for it at compile-time. To make these guarantees, we wrap our
unsafe code in a data structure that can then be wrapped in a safe API; this allows us to treat
the outer type as mutable. One type that follows this pattern is the RefCell<T> type

### Enforcing Borrowing Rules at Runtime with RefCell<T>
- RefCell<T> represents single ownership of the data it holds
- Recall the following borrowing rules:
    - At any given time, we can have *either* one mutable reference *or* any number of immutable
    ones. Multiple mutable references are not allowed, and no mutable references are allowed if
    there is already an immutable reference to the data
    - References must always be valid; this means that the data being referenced must live
    (at least) as long as the reference
- RefCell<T> differs from Box<T> because the borrowing rules are enforced for RefCell<T> at runtime
rather than at compile time. Unsafe code allows us to bypass these checks by the compiler; however,
if the borrowing rules are violated at any point in the execution of our program it will panic and
exit. This means we must exercise caution when using this method! The compiler will not help us
- Like Rc<T>, RefCell<T> is only valid for single-threaded code; an attempt to use it in a
multithreaded context will result in an error being thrown at compile-time


### Interior Mobility: A Mutable Borrow to an Immutable Value
- Per Rust's borrowing rules, you are unable to mutably borrow an immutable value (duh...)
- There are certain scenarios, though, where it may be benefical to have a particular value
behave as mutable in certain contexts but appear as immutable to all other code; this can
be achieved via the use of RefCell<T>

#### Mock Objects:
- One potential use case for interior mobility is mock objects--specific types of test doubles
that are used to record what happens during a test; they allow us to make assertions about
what took place
- Rust is not an object oriented language, and therefore Rust does not have objects; this also
means that Rust does not have built in support for mock objects. However, we can create structs
that will grant us the same functionality
- The following code block serves as an example of how one may go about writing a test with
a mock object. In this case we create a mock messenger object that will keep track of messages
it is told to send. It implements a trait Messenger and has its own send function designed to
log sent messages. Note that the send method takes an immutable reference to self, which breaks
Rust's borrowing rules and keeps the code from compiling; however, we can't change this to a
mutable reference because it must match the signature of the Messenger trait which requires
an immutable reference:

```rust
// Messenger struct:

pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<'a, T: Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T> LimitTracker<'a, T>
    where T: Messenger {
    pub fn new(messenger: &T, max: usize) -> LimitTracker<T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }

    pub fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage_of_max = self.value as f64 / self.max as f64;

        if percentage_of_max >= 1.0 {
            self.messenger.send("Error: You are over your quota!");
        } else if percentage_of_max >= 0.9 {
             self.messenger.send("Urgent warning: You've used up over 90% of your quota!");
        } else if percentage_of_max >= 0.75 {
            self.messenger.send("Warning: You've used up over 75% of your quota!");
        }
    }
}
```

```rust
#[cfg(test)]
mod tests {
    use super::*;

    struct MockMessenger {
        sent_messages: Vec<String>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger { sent_messages: vec![] }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        let mock_messenger = MockMessenger::new();
        let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);

        limit_tracker.set_value(80);

        assert_eq!(mock_messenger.sent_messages.len(), 1);
    }
}
```

- We can, however, use interior mutabilty to solve this problem. Instead of just storing
the logged messages in a vector, we can instead wrap that vector in a RecCell<T>; this will
allow the send method to modify the log. Once we modify the assertion to check the newly
nested vector, this code will run exactly as we intended:

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use std::cell::RefCell;

    struct MockMessenger {
        sent_messages: RefCell<Vec<String>>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger { sent_messages: RefCell::new(vec![]) }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.borrow_mut().push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        // --snip--
        let mock_messenger = MockMessenger::new();
        let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);
        limit_tracker.set_value(75);

        assert_eq!(mock_messenger.sent_messages.borrow().len(), 1);
    }
}
```

### Keeping Track of Borrows at Runtime with RefCell<T>
- While traditional references are notated with the & and &mut syntax, RefCell<T>
has its own *borrow* and *borrow_mut* methods; these allow us to interact with RefCell<T>
through a safe API
- The borrow method will return a smart pointer of the type Ref<T>, and the borrow_mut method
will return a smart pointer of the type RefMut<T>; both types implement Deref thus we can
interact with them like traditional references
- RefCell<T> keeps a count of how many Ref<T> and RefMut<T> pointers are currently active at
any given time. With each call to borrow, the count of immutable borrows is incremented; with
each call to borrow_mut, the count of mutable borrows is incremented. Just as with Rc<T>, the
counts are decremented automatically when a reference goes out of scope
- Just like the compile-time borrowing rules, RefCell<T> will allow any number of immutable
references **OR** one mutable borrow at any given time


### Having Multiple Owners of Mutable Data by Combining Rc<T> and RefCell<T>
- One common way to use RefCell<T> is in combination with Rc<T>
- By storing a RefCell<T> inside of an Rc<T>, you can have a value that can
have mutiple owners **AND** mutate
- Consider the following implementation of our Cons List enum; using this techinque
allows us to have an outwardly immutable reference but leverage interior mutability
through RefCell<T> and its methods:

```rust
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

    let b = Cons(Rc::new(RefCell::new(6)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(10)), Rc::clone(&a));

    *value.borrow_mut() += 10;
}
```

## Reference Cycles Can Leak Memory
Though Rust enforces strict memory saftey rules, it is still possible to accidently create memory
leaks in Rust programs; making use of structures such as Rc<T> and RefCell<T> come with the
potential for vulnerabilities that can not be checked at compile-time. Given the nature of these
types, it is possible to create reference cycles; if items refer to each other in a cycle, the
reference count will never reach 0 and the values will never be cleaned up

### Preventing Reference Cycles: Turning an Rc<T> into a Weak<T>
- The method Rc::clone creates a strong reference and increments the strong_count of the Rc<T>.
We can instead, however, create a weak reference to the value via the Rc::downgrade method and
pass that reference to the Rc<T>
- Rc::downgrade produces a smart pointer of type Weak<T>; this increments the weak_count
- Weak references are useful because the weak count for a particular Rc<T> does not have
to reach 0 in order for the Rc<T> instance to be cleaned up
- Weak references do not express ownership and will never cause a reference cycle; any cycle
involving weak references will be broken once the strong reference count of values involved
reaches 0
- Note that a value any particular Weak<T> references may have already been dropped, we must
interact with these values indirectly. To retrieve a value referenced by a Weak<T>, we use the
*upgrade* method for the Weak<T> instance which will return an Option<Rc<T>>; the option will
return some if the value is still valid and None if the value has already been dropped


### Creating a Tree Data Structure: A Node with Child Nodes
A tree structure is a great example use case for this technique, as we want each node to own
its children, yet we also want to share that ownership with other variables so we can access each
node in the tree directly. We can achieve this by wrapping our node structures in an Rc<T>,
and then wrapping the vector of nodes in a RefCell<T>:

```rust
struct Node {
    value: i32,
    children: RefCell<Vec<Rc<Node>>>,
}
```
The following block of code represents how we may interact with this kind of data structure.
Note the use of clone here:

```rust
fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        children: RefCell::new(vec![]),
    });

    let branch = Rc::new(Node {
        value: 5,
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });
}
```

### Adding a Reference from a Child to Its Parent
If we want to make child nodes aware of their parent node, we can add a parent field to our Node
structure; however, the use of another RefCell<T> here will cause a reference cycle and a potential
memory leak. To combat this, we use a weak reference to the node instead of a strong one:

```rust
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}
```

These changes are reflected in the following driver code. Note the use of the borrow, borrow_mut,
and upgrade methods here:

```rust
fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());

    let branch = Rc::new(Node {
        value: 5,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });

    *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
}
```
