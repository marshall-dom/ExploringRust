# Understanding Ownership

### What is Ownsership?

##### Stack and Heap

- Data stored on the stack must have a known, fixed size

- Data with an unknown size at compile time or a size that may change must be stored
on the heap

- Less efficient to retreive data from the heap, so storing data on the stack is preferable
when possible

##### Ownership Rules

1. Each value in Rust has a variable that is called its _owner_
2. There can only be one owner at a time
3. When the owner goes out of scope, the value will be dropped


##### The String Type

- Rust has two string types:
    1. String literals (immutable)
    2. Strings (mutable)

- Strings are allocated on the heap and are able to store an amount of data that
is unknown at compile-time

- Strings can be made from string literals with the associated function:

```rust
    let s = String::from("Hello");
```

- String literals are fast and efficient because they are hardcoded directly to the final
executable. This is possile because the contents are known at compile-time.


##### Memory and Allocation

- In Rust, memory is automatically returned once the variable that owns it goes out of scope

- When ownership of a value stored on the stack is transfered, the value is essentially copied
from the first to second.

- After this block of code executes, both x and y will equal 5:

```rust
    let x = 5;
    let y = x;
```

###### Move:

- When ownership of a value stored on the heap is transfered; however, the value is not copied.
Instead, the pointer is handed off from the first to the second variable, and the first will be
considered invalid

- After this block of code, s2 will point to "hello" and s1 will be considered invalid:

```rust
    let s1 = String::from("hello");
    let s2 = s1;
```

###### Clone:

- If you _do_ want to deeply copy the heap data instead of just moving the pointer, use the
"clone" method

- After this block of code executes, s1 and s2 will point to two separate addresses on the heap,
each with their own data:

```rust
    let s1 = String::from("hello");
    let s2 = s1.clone();
```

- The copy trait is actually called implicitly on data stored on the stack (e.g. integers)

- The following types implement Copy:
    - All the integer types
    - The Boolean type
    - All the floating point types
    - The char type
    - Tuples if they contain only types that also Copy
