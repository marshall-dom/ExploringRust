# Programming a Guessing Game

## Reflection

While this was not my first time being exposed to Rust and its syntax,
it was, in fact, my first time writing a program in the language as well as
getting my hands on some of the tools built into Cargo.

#### First Impressions of Cargo:

- While I can appreciate the approach taken here, I still find it somewhat
annoying to have to manually specify dependencies in the Cargo.toml file in order
to add them to a project. After using Go and it's dependency management system for
a while (i.e. go modules and go-get), it felt weird having to do this by hand again.

- **ADDENDUM** - I have since discovered that external creates can be installed through
cargo via cargo install. I had a hard time believing that they would not include this
in their system. Disregard the previous bullet!

- I love being able to run "cargo fmt" at any time to automatically format my code
to the latest spec conventions. I found this really useful; in fact, I will probably
set a keymap for this in vim so I can do it even more quickly.

- Feedback from the compiler is excellent! Not only is it incredibly clear, but it is
also thorough.

#### Language Specific Thoughts

- The idea that everything is immutable by default and must be explicitly marked mutable is
nice IMO. This forces you to be thoughtful in your design choices and write better code.

- The idea of having a small scope by default is appealing to me, as keeping things as minimal
and efficient as possible is a strong selling point of the language for me.

- As expected, syntax and verbage is different due to the fact the language is not object-oriented.
For example, the :: syntax signifies an "associated function", which can be likened to a method of
an object in OO languages. In the case of Rust, methods do not exist; instead there are implementations
of functions for types.

- This particular page of documentation talked a lot about "binding"--specifically the binding of
variables to instances. I like this perspective better than "assignment," for it further enforces
the idea of the relationship between a variable and its associated value. One of the pitfalls of
current teaching philosophies, IMO, is the idea that a particular value is "assigned" to a variable,
or that a variable "contains" a particular value, when under the hood this is not actually how this
process is handled. If this same way of thinking is applied to more complex programming--namely
concurrency, it will eventually cause big problems. I digress...

- I also like the idea of having syntax for both mutable and non-mutable references. It is my
understanding that this is soley for the compiler, so the inlusion of this extra check is really
cool considering it comes with no performance cost at runtime.

- I am a fan (so far) of the way Rust seems to manage error handling. I very much like the notion
of a result type, and I like the idea of creating custom result types even more.

- The syntax for matching is also nice and very straightforward. I find the idea of having what
is essentially a list of arms to be quite tidy, and the use of arrow syntax instead of separating
cases by colons and indentation to be very readable.

- Note to self: Rust defaults to i32 for integer values!

- The use of match statements to switch (this verbage will take a minute to adjust to) result
types is an elegant method of error handling. This offers clarity and allows for bindings
to be more succint than in other languages.
