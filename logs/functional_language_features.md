# Functional Language FeaturesL Iterators and Closures

## Closures: Anonymous Functions that Can Capture Their Environment
- Closures can be created in one place an then called in a different context
- Unlike functions, closures can capture values from the scope in which they're defined

### Creating an Abstraction of Behavior with Closures
One way to avoid multiple calls to an expensive function may be to call that function
once at the beginning of your process and store its return in a variable; however,
this is inefficient if there are cases which do not require a call to that function.
Alternatively, we can refactor that code to use closures--defining and storing
the function itself in a variable rather than the function's return value. If a function
only has one calling function, it may be more efficient to write this logic as a closure
as it will then share the same scope. We define closures with the following syntax, specifying
the parameters to the closure inside pipes:

```rust
let some_closure = |num| {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    num
};
```

### Closure Type Inference and Annotation
Type definitions are not required for the parameters or return values of closures.
Because closures are only relevant in a narrow context, the compiler is able to
reliably infer type. Depending on the situation, however, it may be beneficial to
add explicit type annotations for the sake of clarity and readability. I can see
this being useful if the closure does some "magic" and you want to improve the readability
for others that may have to understand your code in the future

**NOTE**: Closure definitions will have a single concrete type inferred per value. This means
that the usage of the closure must be consistent. The types are inferred; this does not mean that
you are given the flexibility of generics for free

### Storing Closures Using Generic Paramenters and the Fn Traits
- Another potential strategy is to implement memoization by defining a struct that holds both a
closure and the result of that closure. In this implementation, the struct will only execute
the closure if the resulting value is needed and then cache the result. This is beneficial
for a few reasons:
1. Efficiency
2. Encapsulation

- When defining a struct that holds a closure, types must be explicitly defined; recall that
struct definitions require explicit type annoations for each of its fields

- To define custom data types that hold closures, we use generics and trait bounds:

```rust
struct Cacher<T>
    where T: Fn(u32) -> u32
{
    calculation: T,
    value: Option<u32>,
}
```

- A simple implementation for this structure may look like the following:

```rust
struct Cacher<T>
    where T: Fn(u32) -> u32
{
    calculation: T,
    value: Option<u32>,
}

impl<T> Cacher<T>
    where T: Fn(u32) -> u32
{
    fn new(calculation: T) -> Cacher<T> {
        Cacher {
            calculation,
            value: None,
        }
    }

    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(v) => v,
            None => {
                let v = (self.calculation)(arg);
                self.value = Some(v);
                v
            },
        }
    }
}
```

### Capturing the Environment with Closures
- Closures offer additional functionality over traditional functions because they
can access the scope in which they are defined. For example, we are able to write
code like the following. Note that the closure is able to access the data bound to
the variable x:

```rust
let x = 4;

let equal_to_x = |z| z == x;
```
- This benefit comes at a cost of overhead, however, as the closure uses additional memory
to store the values captured from the environment
- Closures can capture values from their environment in three ways:
    1. FnOnce consumes the variables it captures from its enclosing scope. The closure takes
    ownership of tehese avriables and moves them into the closure when it is defined. This
    type of closure can only be called once
    2. FnMut mutably borrows values. This means that it is able to change the environment
    3. Fn borrows values from the environment immutably
- Rust infers which trait to use based on how the closure uses the values captured from the
environment
- All closures implement FnOnce because every closure can be called at least once
- Closures that do not move captured variables also implement FnMut
- Closures that do not need access to the captured variables also implement Fn
- We can force closures to take ownership of values by using the move keyword before the parameter
list:

```rust
let x = vec![1, 2, 3];

let equal_to_x = move |z| z == x;
```

## Processing a Series of Items with Iterators
- The iterator pattern provides a handle to perform some task on a sequence of items
- Iterators contain the logic for iterating over each item in a sequence as well as determining
when that sequence has finished
- In Rust, iterators are lazy; they do nothing until they are consumed

### The Iterator Trait and the next Method
- All iterators implement a trait called Iterator, which itself implements a method called next
- Next can be called directly or be leveraged by loops
- **NOTE**: If you wish to use the next method, the iterator must be made mutable. Calling the next
method on an iterator changes the internal state that the iterator uses to track its position in 
the given sequence

### Methods that Consume the Iterator
Methods that call an iterator's next method are referred to as consuming adaptors; this is because
calling them uses up the iterator. An example of this is the sum method, whiich takes ownership of
the iterator when called and repeatedly calls next until the end of the sequence is reached. Note 
that an iterator can not be used after it is used by a consuming adaptor, for that function takes
ownership of the iterator

### Methods that Produce Other Iterators
- Iterator adaptors are methods defined on the Iterator trait that allow you to change iterators
into different kinds of iterators
- Calls to iterator adaptors can be chained in order to perform more complex actions in a readable
way
- **NOTE**: Because iterators are lazy, a consuming adaptor method must be called to get results
from calls to iterator adaptors
- The following code block demonstrates an example in which this strategy may be used. Here we
pass an iterator into the map function and then consume that result with a call to collect(),
which will collect those results into a vector:

```rust
let v1: Vec<i32> = vec![1, 2, 3];

let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

assert_eq!(v2, vec![2, 3, 4]);
```

### Using Closures that Capture Their Environment
- One common use of closures is the filter iterator adaptor
- The filter method takes a closure that takes each item from the iterator and returns a boolean.
If the closure returns true, the value will be included in the iterator produced by filter; if
false, the value will not be included in the resulting iterator. The following code block
demonstrates how this may be used to filter by a particular value. The filter method is
passed a closure which checks if the size of the shoe matches the specified size, and then
collect is called on the resulting iterator to collect those shoes into a vector:

```rust
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_my_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter()
        .filter(|s| s.size == shoe_size)
        .collect()
}
```

### Creating Our Own Iterators with the Iterator Trait
- Iterators can be created for other collection types in the standard library
- Iterators can be created for custom types by providing an implementation for the
iterator trait on that type
- Note that the only method required to satisfy the iterator trait is a next method:

```rust
struct Counter {
    count: u32,
}

impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;

        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}
```

- **NOTE**: Once a data type implements the Iterator trait, it also has access to any of the
default implementations defined for the Iterator trait

## Comparing Performance: Loops vs. Iterators
- Iterators in Rust are considered zero-cost abstractions, meaning that they don't require
any additional overhead at runtime; they get compiled down to roughly the same code as if
you were to write the lower-level code yourself
- Rust's compiler also generates optimized code depending on the particular use case. For example,
it will unroll a loop if the scenario calls for it
