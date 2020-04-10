# Fearless Concurrency
- Concurrency and parallelism are at the core of Rust's development
- Rust's strict rules not only ensure memory safety but also prevent many concurrency problems;
many concurrency errors are caught as compile-time errors in Rust due to ownership and type
checking

## Using Threads to Run Code Simultaneously
- Using threads to run concurrent computations can offer considerable perfomance benefit; however,
improper management of threads can lead to problems such as:
    - Race conditions
    - Deadlocks
    - Bugs that are hard to reliably reproduce and fix

### Creating a New Thread with spawn
We can create a new thread by calling thread::spawn and passing a closure as an argument;
this will spawn a new thread and execute the closure in that thread. Note that threads spawned
via this function will be stopped when the main thread ends regardless of its own state

### Waiting for All Threads to Finish Using join Handles
- Not only are threads spawned via thread::spawn not guaranteed to finish executing, but they are
not guaranteed to run at all. To guarantee that the thread will run *and* finish, we must save
the return value of thread::spawn as a variable; the return type of thead::spawn is a JoinHandle
- A JoinHandle is an owned value that, when we call the join method on it, will wait for its
thread to finish executing
- Calling join on a JoinHandle blocks the current thread until the thread represented by the
handle terminates
- This technique is demonstrated by the following code block. We create a handle for the new
thread by binding the thread to the variable handle; this variable is of type JoinHandle.
We then call join() on the handle to ensure that the current thread does not terminate before
the thread we spawned has a chance to run and finish. Note that we call join() after the logic
in the current thread; this will allow the logic in the current thread to be executed before
blocking:

```rust
fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```
The output from this code would look like this:

```
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

### Using move Closures with Threads
- The move closure if often used alongside thread::spawn to use data from one thread in another;
just like the move keyword is used with variables to transfer ownership, the move keyword is
used in this context to transfer ownership of values from one thread to another
- We can have the closure capture the environmentby just using the pipe syntax; however, doing
so will throw errors unless specific lifetimes are provided. This is because we must ensure that
the closure does not outlive the values borrowed by the closure. Rather than borrowing values,
we can use the move keyword to force a transfer of ownership of the any variables referenced
inside of the closure. Note that this will mean we can not access moved variables from the current
thread until the closure returns ownership:

```rust
let handle = thread::spawn(move || {
    println!("The move syntax looks like this")
})
```

## Using Message Passing to Transfer Data Between Threads
- Rust uses *channels* to pass messages from one thread to another; this is a popular modern
approach to concurrency as it allows for threads to share memory by communicating rather than
communicating by sharing memory
- Channels have two halves:
    - A transmitter which is upstream
    - A receiver which is downstream
- We call methods on transmitters to pass data, and call methods on receivers to listen for
and collect messages sent through the channel
- A channel is said to be closed if either half is dropped
- Channels are instantiated by calling mpsc::channel(); this will return both a transmitter and
receiver in the form of a tuple. Note that the names tx and rx are used by convention to refer
to transmitters and receivers:

```rust
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();
}
```

- **NOTE**: mpcs is short for *multiple producer, single consumer*; this reflects the paradgim
followed throughout this package
- Rust's implementation of channels allows for multiple transmitters but only *one* receiver
- Transmitters have a method *send* that will send the value is it passed through the channel;
it returns a result type that will contain an error if the channel's receiver has already been
dropped. Likewise, receivers have methods *recv* and *try_recv* that are used to capture messages
from the channel. recv will block the main thread to wait for a value to be sent; try_recv
will immediately return a Result type instead of blocking
- The following block of code represents a simplistic use of this technique. Note the use of
unwrap() is used here for simplicity; potential errors should be handled more throuroughly in a
real program:

```rust
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

### Channels and Ownership Transference
- Rust was designed with concurrency and parallelism in mind, and its ownership rules are
integral to its guarantees made about safe currency
- The send function is one example of this; when it is called on a transmitter, the transmitter
assumes ownership of the value it is passed. Then, when the value is moved, the receiver assumes
ownership when it is captured on the other end. This means that potential errors from poorly
written concurent code can be caught at compile-time instead of manifesting themselves at runtime.
Nice!

### Creating Multiple Producers by Cloning the Transmitter
- In many cases, we may want to have multiple transmitters by which to send data
through a particular channel; we can do this by cloning the transmitter and passing
that clone to the desired thread instead of the original transmitter itself:

```rust
use std::thread;
use std::sync::mpsc;
use std::time::Duration;

fn main() {
// --snip--

let (tx, rx) = mpsc::channel();

let tx1 = mpsc::Sender::clone(&tx);
thread::spawn(move || {
    let vals = vec![
        String::from("hi"),
        String::from("from"),
        String::from("the"),
        String::from("thread"),
    ];

    for val in vals {
        tx1.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

thread::spawn(move || {
    let vals = vec![
        String::from("more"),
        String::from("messages"),
        String::from("for"),
        String::from("you"),
    ];

    for val in vals {
        tx.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

for received in rx {
    println!("Got: {}", received);
}

// --snip--
}
```

## Shared-State Concurrency
While the use of channels has its benefits in certain use cases, we can also write concurrent
code with the more traditional shared-state paradgim. Shared-state concurrency is inherently
more prone to errors than using channels because it requires multiple ownership; however,
Rust's strict ownership rules offer a lot of help in making shared-state concurrency more safe

### Using Mutexes to Allow Access to Data from One Thread at a Time
- *Mutex* is short for *mutual exclusion*; this meaning only one thread can access some
piece of data any given time
- Note that in Rust, the Mutex type is actually a smart pointer
- A mutex guards shared-state through the use of a lock; a thread must acquire that lock in order
to access the data its mutex is guarding. Once a thread is finished accessing the data, the lock
is forfeited and returned so other threads can then acquire the lock
- The API for Rust's mutex looks as follows. Calling lock on the mutex will return a Result type
*LockResult* containg either a smart pointer called *MutexGuard* or an error if the lock can not be
acquired. MutexGuard implements the Deref trait to point to the inner data guarded by the mutex,
as well as an implementation of the Drop trait that will automatically release the lock when the
MutexGuard instance goes out of scope:

```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }

    println!("m = {:?}", m);
}
```

### Sharing a Mutex Between Multiple Threads
- Rust has a special reference counting type *Arc<T>* designed for use in concurrent situations.
Note that Arc derived from *atomically reference counted type*; atomic types are types for which
reading and writing are guaranteed to happen in a single instruction and therefore can not have
access interupted--making them concurrency safe
- We are able to (generally) interact with atomics in the same manner as regular types, but they
are able to be safely shared across threads; this safety comes with a performance hit, however,
as atomics have additional guarantees that must be enforced
- The following code block represents one way in which an Arc type may be used to share data
across threads. Note that we wrap a mutex in an Arc type, and then pass a clone of that Arc to
each thread we spawn:

```rust
use std::sync::{Mutex, Arc};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

### Similarities Between RefCell<T>/Rc<T> and Mutex<T>/Arc<Tt
- Similarly to the Cell family of types, Mutex types also provide interior mutabilility. In the
same way we can wrap an Rc in a RefCell to mutate the data referenced in the Rc, we can wrap
a Mutex inside of of Arc to mutably reference the contents of the Mutex
- Rust's compiler can not provide absolute protection against logical errors when using mutexes.
Similarly to the way in which Rc values can create a reference cycle, improper management of
mutexes can create deadlocks

## Extensible Concurrency with the Sync and Send Traits
Rust actually has very few concurrency features built in; note that most of
the features we have discussed thus far have been from the standard library,
not the language. There are, however, two concurrency concepts that are a part of the
core language--both of which are located in std::marker:
    - Sync
    - Send

### Allowing Transference of Ownership Between Threads with Send
- The *Send* marker trait indicates that ownership of the type implementing Send is able
to be transfered between threads
- Almost every Rust type is Send, but there are exceptions such as Rc<T>; this is due to the
fact that Rc<T> can be cloned. Reference count types that are meant to be used exclusively in
single threaded contexts do not account for the possibility that multiple threads may update
the reference count at the same time
- **NOTE**: Any type composed entirely of Send types is automatically marked as Send as well;
almost all primitive types besides raw pointers are Send types

### Allowing Access from Multiple Threads with Sync
- The *Sync* marker trait indicates that it is safe for the type implementing Sync to be
referenced from multiple threads
- **NOTE**: Just as with Send, types composed entirely of Sync types are also Sync
