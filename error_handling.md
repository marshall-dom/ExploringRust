# Error Handling

Some notes before diving in...

- Rust breaks errors into two umbrella categories:
    1. Recoverable Errors
    2. Unrecoverable Errors

- Recoverable errors are generally in relation to user input or i/o, and are
generally handled by reporting the problem to the user and retrying the operation

- Unrecoverable errors are caused by bugs in Rust code

- Rust has no notion of exceptions; instead, it has the Result type

- Operations that have the potential to throw recoverable errors generally return Result
types, which can contain either a type or an error

- For unrecoverable errors, the panic! macro is used to stop the execution of the program

## Unrecoverable Errors with Panic!

- When the panic! macro executes, the program will do the following:
    1. print a failure message
    2. unwind and clean up the stack
    3. quit

- The error message written will display the line in which panic! is called;
however, this may not be where the error actually occured depending on how the code
was structured. There is also a backtrace function built into panic that allows us
to run a backtrace to examine the program's failure in more detail

- Similarly, if the panic is a result of an error in the code and not called by us,
there is a possiblity that message will point to a line of a file in some library
and not neccessarily the line of your code that caused the issue. We can use the
backtrace function for this as well

- A backtrace is a list of all of the functions that have been called up to the point
of failure

- **NOTE**: In order to get the full information provided by the backtrace, debug
symbols must be enabled

## Recoverable Errors with Result

- The Result enum is defined to have two variants, Ok and Err:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

- One way to handle returned result types is with a match statment like the following. In
this case, the value in f will be an instance of Ok that contains a file handle. In the case
that open returns an error, the value in f will be an instance of Err that contains more
information regarding the error. The body of the match statement will unwrap the result in
either case--binding the file handle to f if Ok, or calling panic! with the unrwapped error:

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("Problem opening the file: {:?}", error)
        },
    };
}
```

- We can also used nested match expressions to things like this:

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => panic!("Problem opening the file: {:?}", other_error),
        },
    };
}
```

- While nesting match expressions will work, they can quickly become an ugly mess.
As an alternative, we can return closures on match arms. Depending on the situation,
this can be much easier to read:

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Problem creating the file: {:?}", error);
            })
        } else {
            panic!("Problem opening the file: {:?}", error);
        }
    });
}
```

### Shortcuts for Panic on Error: Unwrap and Expect

- Match expressions work well enough in some cases, but they can be verbose and do not always
do the best job of communicating intent. As an alternative, the methods unwrap and expect can
be used to handle Result types. Both methods are described as shortcut methods because they
will either return the value wrapped in Ok on success or panic on an error

- Unrwap can be chained onto a function call that returns a result type. The value returned
by the function will be consumed by unwrap--which will either return the value wrapped in Ok
or call panic!. As you can see, this is much more succinct:

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

- Expect will behave similarly; however, it allows you to specify an error message to panic!
Here is the same example code handled with expect instead of unwrap:

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

### Propogating Errors

- We can also leverage the result type to abstract away error handling from supporting functions
we write. This allows us to handle potential errors in the main code of our program instead of
each function. This is relatively easy to do; instead of panicing inside the function, we just
return the error instead:

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```
- Rust also provides an operator ? called "try" which allows us to achieve something similar
with a more concise syntax. On success this will return the Ok type and unwrap it, and on
failure it will convert the error to the returned error type and return that error (provided
that the particular error type implements the from method for accomplishing this conversion):

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

- The ? operator also supports chaining and could be simplified even further:

```rust
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();

    File::open("hello.txt")?.read_to_string(&mut s)?;

    Ok(s)
}
```

- **NOTE**: The ? operator can only be used in functions that return Result types!

## To panic! or Not to panic!

- In general, it is wise to return result types from functions that may fail, that way
the calling code can decide how to best handle the error based on context. Sometimes, however,
it is most appropriate to call panic! directly

### Examples, Prototype Code, and Test

- In these situations, it is often helpful to panic! immediately for the following reasons:
    1. So the program fails immediately if any method or function fails
    2. If you are prototyping and have not yet decided how you want to handle the potential errors 
    3. They can be expressive in examples (e.g. some_function().expect("this thing failed here
    because..."))

### Cases in Which You Have More Information Than the Compiler

- There will be cases in which there is some other logic that ensures the returned Result will
have an Ok value, but this logic is not necessarily understood by the compiler. If you can
verify manually with absolute certainty that there will never be an Err returned, then
it is perfectly acceptable to use unwrap. For example:

```rust
use std::net::IpAddr;

let home: IpAddr = "127.0.0.1".parse().unwrap();
```

### Guidelines for Error Handling

- If it is possible that your code could end up in a "bad state", you should have your
code panic

- According to the Rust documentation, a "bad state" in this context is when "some assumption,
guarantee, contract, or invariant has been brokwen, such as when invalid values, contradictory
values, or missing values are passed to your code"--plus one or more of the following:
    - The bad state is not something that is expected to happen occassionally
    - Your code after this point needs to rely on not being in this bad state
    - There is not a good way to encode this information into the types you use

- If your code is meant to be used as part of a library, it may be wise to panic! in the case
that someone calls your code and passes in values that don't make sense. This would alert the
person using your library of the bug in their code and allow them to fix it

- Similarly, it is often appropriate to panic! if you're calling external code that is out of
your control and it returns an invalid state that you have no way of fixing

### Creating Custom Types for Validation

- If you anticipate having a lot of potential errors that stem from a particular value or
operation, you could create a custom type to store that value and build validation into
its associated functions. By structuring your code this way, it will limit the error 
handling required in the calling code. This may look something like the following:

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {
            value
        }
    }

    pub fn value(&self) -> i32 {
        self.value
    }
}
```
