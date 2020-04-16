# Patterns and Matching
Patterns consist of some combination of the following:
- Literals
- Destructured arrays, enums, structs, or tuples
- Variables
- Wildcards
- Placeholders

## All the Places Patterns Can Be Used

### Match Arms
- One example of patterns in Rust is through the use of match expressions; these expressions
use patterns to match on a particular value and then an expression to run if the given
value matches that arms pattern:

```rust
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```

- Match expressions must be exhaustive; this means that there must be an arm for each
possible pattern
- The "\_" pattern will match anything, but never binds to a variable; this pattern can
be used as a catchall or default case arm of a match statement

### Conditional if let Expressions
- if let expressions are an alternative to match expressions that can be used to run expressions
if a particular value matches its pattern
- if let expressions can be used as a shorter way of writing match expressions with only one case;
however, they can also contain else if, else if let, and else blocks to account for more cases
- Unlike match statements, the conditions in a series of if let, else if, else if let arms are
not required to relate to each other; this makes them more flexible than match expressions. The
following code block serves as an example of this. Note that while if let expressions offer
additional flexibility over match expressions and will support more complex requirements, they
come with their own set of downsides; because if let expression arms do not require exhaustive
pattern matching, the compiler can not guarantee against the possiblity of bugs:

```rust
fn main() {
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = "34".parse();

    if let Some(color) = favorite_color {
        println!("Using your favorite color, {}, as the background", color);
    } else if is_tuesday {
        println!("Tuesday is green day!");
    } else if let Ok(age) = age {
        if age > 30 {
            println!("Using purple as the background color");
        } else {
            println!("Using orange as the background color");
        }
    } else {
        println!("Using blue as the background color");
    }
}
```

### while let Conditional Loops
while let conditional loops leverage if let statements to execute a particular block of
code until the pattern fails to match; this can be used in the following way:

```rust
fn main() {
    let mut stack = Vec::new();

    stack.push(1);
    stack.push(2);
    stack.push(3);

    while let Some(top) = stack.pop() {
        println!("{}", top);
    }
}
```

### for Loops
for loops use pattern matching as well; the code block nested in the for loop will continue
to be run until the value produced by the iterator no longer matches the condition:

```rust
fn main() {
let v = vec!['a', 'b', 'c'];

    for (index, value) in v.iter().enumerate() {
        println!("{} is at index {}", value, index);
    }
}
```

### let Statements
While it isn't as explicit, let statements in fact use patterns as well; let statments
actually look something like this under the hood. Rust compares the expresion against the
pattern and assigns any names it finds; in other words, the provided expression is bound
to whatever variable name provided in the let statement:

```rust
let PATTERN = EXPRESSION;
```

### Function Parameters
Function parameters are also patterns in the way that the parameters provided in the function
signature are matched to the arguments provided when the function is called; the same is true
with closure parameter lists

## Refutability: Whether a Pattern Might Fail to Match
- There are two types of patterns:
    - Refutable
    - Irrefutable
- Patterns that will match for any value passed are irrefutable; patterns that fail to match for
some possible value are refutable. For example, a let statement would be an irrefutable pattern
because the pattern cannot fail to match; an if let statement with a pattern other than "\_" would
be a refutable pattern as it will not match in some case
- Function parameters, let statements, and for loops can only accept irrefutable patterns, as the
program cannot do anything meaningful when values don't match
- The if let and while let expressions will except refutable and irrefutable patterns, but the
compiler will throw a warning for the use of irrefutable patterns as they render the expression
meaningless; the entire point of using these conditional expressions is to manage control flow 
based on the matching of the pattern to a particular value
- Match arms must use refutable patterns, except for the last arm which should use an irrefutable
pattern to match any remaining values

## Pattern Syntax

### Matching Literals
We can match patterns against literals directly:

```rust
let x = 1;

match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

### Matching Named Variables
Named values are irrefutable patterns that match many values; however, a complication arises
when we attempt to use them in match expressions. Because match expressions start a new scope,
variables that we declare as part of a pattern inside the match expression will shadow those
with the same name outside of the match scope

### Multiple Patterns
We can match multiple patterns in one arm by using the pipe syntax; the first arm of the
following match expression will match on either 1 or 2:

```rust
let x = 1;

match x {
    1 | 2 => println!("one or two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

### Matching Ranges of Values with ..=
We an use the ..= syntax to match on an inclusive range of values; the first arm of the following
match expression will match on any value between 1 and 5. Note that ranges are only allowed with
numeric values or char values:

```rust
let x = 5;

match x {
    1..=5 => println!("one through five"),
    _ => println!("something else"),
}
```

### Destructuring to Break Apart Values
We can use patterns to destructure structs, enums, tuples, and refrences to different parts
of these values

#### Destructuring Structs
The following code block demonstrates how we can break apart a struct using a pattern with
a let statement. We can either do this by providing named variables for the fields of the struct,
or by binding the fields themselves to a variable:

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x, y } = p;
    assert_eq!(0, x);
    assert_eq!(7, y);
}
```

With this knowledge, we can then apply this technique to a match expression to match on
particular fields of a struct:

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    match p {
        Point { x, y: 0 } => println!("On the x axis at {}", x),
        Point { x: 0, y } => println!("On the y axis at {}", y),
        Point { x, y } => println!("On neither axis: ({}, {})", x, y),
    }
}
```

#### Destructuring Enums
Similar to how we can break apart structs by their fields, we can also destructure enums
to access their inner fields; this is done in the following way through the :: syntax:

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);

    match msg {
        Message::Quit => {
            println!("The Quit variant has no data to destructure.")
        }
        Message::Move { x, y } => {
            println!(
                "Move in the x direction {} and in the y direction {}",
                x,
                y
            );
        }
        Message::Write(text) => println!("Text message: {}", text),
        Message::ChangeColor(r, g, b) => {
            println!(
                "Change the color to red {}, green {}, and blue {}",
                r,
                g,
                b
            )
        }
    }
}
```

#### Destructuring Nested Structs and Enums
We can also destructure nested structs and enums with the same :: syntax:

```rust
enum Color {
   Rgb(i32, i32, i32),
   Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

fn main() {
    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

    match msg {
        Message::ChangeColor(Color::Rgb(r, g, b)) => {
            println!(
                "Change the color to red {}, green {}, and blue {}",
                r,
                g,
                b
            )
        }
        Message::ChangeColor(Color::Hsv(h, s, v)) => {
            println!(
                "Change the color to hue {}, saturation {}, and value {}",
                h,
                s,
                v
            )
        }
        _ => ()
    }
}
```

#### Destructuring Structs and Tuples
We can mix and match destructuring patterns for even more powerful operations The following
block demonstrates how we could extract all of the primitive values out of a complex structure;
here we break apart a structure which has nested structs and tuples inside of a tuple:

```rust
let ((feet, inches), Point {x, y}) = ((3, 10), Point { x: 3, y: -10 });
```

### Ignoring Values in a Pattern
There are cases in which we may want to have the catchall arm of a match expression do nothing;
we can do this with the \_ pattern

#### Ignoring an Entire Value with \_
While the underscore pattern is useful in the last arm of a match expression, it can also be used
in any pattern where we wish to ignore a particular value. The following code block demonstrates
how it may be used in a function signature; this code will completely ignore the value passed
as the first argument. Unlike using named values, the compiler will not throw a warning about
unused function parameters if they are unnamed:

```rust
fn foo(_: i32, y: i32) {
    println!("This code only uses the y parameter: {}", y);
}
```

#### Ignoring Parts of a Value with a Nested \_
We can also nest underscore patterns inside of other patterns to ignore parts of values
instead of the entire value; this is helpful in cases where we want to test for some part
of a value but have no use for other parts. In situations such as this, it is responsible
to structure our code in the following way. Note that in the following implmentation, we
are unable to overwrite an existing setting; however, we can unset the setting and then
give that setting a value if it is currently unset:

```rust
let mut setting_value = Some(5);
let new_setting_value = Some(10);

match (setting_value, new_setting_value) {
    (Some(_), Some(_)) => {
        println!("Can't overwrite an existing customized value");
    }
    _ => {
        setting_value = new_setting_value;
    }
}
```

We can also use the underscore in multiple places within the same pattern to ignore particular
values of that pattern as follows:

```rust
let numbers = (2, 4, 8, 16, 32);

match numbers {
    (first, _, third, _, fifth) => {
        println!("Some numbers: {}, {}, {}", first, third, fifth)
    },
}
```

#### Ignoring an Unused Variable by Starting its Name with \_
It may be useful in some cases to create a variable that you won't use yet (e.g. when
you are prototyping or just starting a project). To tell the compiler ignore an unused
variable, we can preface the variable name with an underscore.

**NOTE**: Prefacing a variable with an underscore will still bind the value to the variable;
this is different from just using an underscore, which will not bind anything

#### Ignoring Remaining Parts of a Value with ..
For values that have many parts, we can use the .. syntax to use oly a few parts and ignore
the rest; this allows us to avoid writing underscores for each of the ignored values. Note
that the use of this syntax must be unambiguous; if it is not clear which values are to be ignored
an error will be thrown by the compiler:

```rust
struct Point {
    x: i32,
    y: i32,
    z: i32,
}

let origin = Point { x: 0, y: 0, z: 0 };

match origin {
    Point { x, .. } => println!("x is {}", x),
}
}
```

### Extra Conditionals with Match Guards
A *match guard* is an additional if condition added to a partern on a match arm. For example:

```rust
let num = Some(4);

match num {
    Some(x) if x < 5 => println!("less than five: {}", x),
    Some(x) => println!("{}", x),
    None => (),
}
```

### @ Bindings
The @ operator allows us to create variables that hold values at the same time we are testing
to see whether or not that value matches a particular pattern; this is especially useful in match
expressions or in other conditional statements where we wish to use that bound variable in the
body of the following arm if matched:

```rust
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello { id: id_variable @ 3..=7 } => {
        println!("Found an id in range: {}", id_variable)
    },
    Message::Hello { id: 10..=12 } => {
        println!("Found an id in another range")
    },
    Message::Hello { id } => {
        println!("Found some other id: {}", id)
    },
}
```
