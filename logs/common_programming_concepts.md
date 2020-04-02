# Common Programming Concepts

### Disclaimer

My comments on this section will most likely be incredibly brief considering I already have
experience with a number of other languages. I would expect most of my comments on this module
to be notes about Rust conventions, as well as thoughts syntax and design choices.

### Notes

##### Variables and Mutability

- Rust's naming convention for constans is to use all upercase with underscores between words,
and underscores can be used to separate zeroes in large numbers for readability.
    - Ex: 100000 can be written as: 100_000
    - I actually think this is brilliant

- Rust allows for variable showing as common place--meaning variable names can be reused within
the same scope. To be honest, I am unsure how I feel about this, as I fear it may cause issues
of readability if abused by careless or lazy developers. Shadowing is nice when the new assignments
are close to each other; however, if names are reused a number of lines apart it could be vexing
to debug if working on someone elses code. Overall, though, I am on board.

##### Data Types

- It is recommended to use the default integer types unless otherwise needed. Types 'isize' and
'usize' are primarily used when indexing collections.
    - Be careful when defining types as overflow can occur if insufficient space is allocated
    - Rust will wrap values if in release mode to avoid breakage

- The default floating-point type is f64

- char literals use single quotes; string literals use double quotes.

- Tuples are considered single compound elements and therefore bind to the entire tuple

- Tuples can be indexed through dot notation (e.g. x.0 for the 0th index of tuple x)

- Arrays must be of a single type
    - Useful when you want data to be allocated on the stack rather than the heap
    - Ensure a fixed number of elements

- Out of bounds errors will surface at runtime; not checked by the compiler

##### Functions

- Convention dictates the use of snake case for function and variable names

- Arguments can be named in the function declaration

- The type of each parameter must be specified in the function signature

- Statements are instructions that perform some action and do not return a value

- Expressions evaluate to a resulting value

- Convention dictates that return statements do not require a return keyword. Instead,
they are written as expressions with no semicolon.

- Return types are included in the function signiture after an arrow

##### Comments

- Comments start with two slashes and continue through the end of the line

##### Control Flow

- Parentheses not needed in condition of if statement

- if/let statements can be used for variable bindings
    - if this syntax is used, all arms must be of the same type

- For loops are the recommended loop type for most cases as they are by definition the safest
