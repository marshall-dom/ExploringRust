# Understanding Ownership

## What is Ownsership?

#### Stack and Heap

- Data stored on the stack must have a known, fixed size

- Data with an unknown size at compile time or a size that may change must be stored
on the heap

- Less efficient to retreive data from the heap, so storing data on the stack is preferable
when possible

#### Ownership Rules

1. Each value in Rust has a variable that is called its _owner_
2. There can only be one owner at a time
3. When the owner goes out of scope, the value will be dropped


#### The String Type

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


#### Memory and Allocation

- In Rust, memory is automatically returned once the variable that owns it goes out of scope

- When ownership of a value stored on the stack is transfered, the value is essentially copied
from the first to second.

- After this block of code executes, both x and y will equal 5:

```rust
    let x = 5;
    let y = x;
```

##### Move:

- When ownership of a value stored on the heap is transfered; however, the value is not copied.
Instead, the pointer is handed off from the first to the second variable, and the first will be
considered invalid

- After this block of code, s2 will point to "hello" and s1 will be considered invalid:

```rust
    let s1 = String::from("hello");
    let s2 = s1;
```

##### Clone:

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


## References and Borrowing

- Use references to refer to some value without passing ownership

- In the following block of code, we use a reference to the bound value of s, so s can
still be used later; Rust calls this borrowing. If we were to pass s into the function, that function would now
have ownership of the bound value of s and s could no longer be used:

```rust
    let s = String::from("hello");
    let len = calculate_length(&s);
    println!("The length of '{}' is {}.", s, len);
```

- By default, borrowed values can not be mutated; however, they can be marked as mutable
if mutation is required. This is demonstrated by the following code block:

```rust
let mut s = String::from("hello");
change(&mut s);
```

- **NOTE** - You can only have one mutable reference to any one piece of data in a
particular scope

- Rust also forbids the creation of mutable references while there is already an immutable
one in scope

- Rust's imposed restrictions allow the compiler to prevent data races at compile time.

- Rust's compiler will also guarantee that there are no dangling references

#### Rules of References

- At any given time, you can have either one mutable reference of any number of
immutable references

- References must always be valid


## The Slice Type

- The slice is another data type that does not have ownership

- Slices let you reference a contigious sequence of elements in a collection rather
than the whole collection

- The following code block demonstrates string slice syntax:

```rust
let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];
```

- Range is specified within square brackets; starting is inclusive, ending index is exclusive

- Rust allows for the starting index to be dropped if it is the first index, and the ending index
to be dropped if it is the last

- Using slices allow for the data bound to the slice to live connected to the original string
being referenced. This allows the compiler to ensure that the references into the String remain
valid

- String slices are written as "&str"

- **NOTE** String literals are actually slices; they are references pointing to the specific point
of the binary that contains the string literal (Recall that string literals are stored directly in
the binary)

- Slices can also be used on other data types, like arrays, in similar fashion
