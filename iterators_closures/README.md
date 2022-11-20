
## Closures

Rust’s closures are anonymous functions you can save in a variable or pass as arguments to other functions. 
You can create the closure in one place and then call the closure elsewhere to evaluate it in a different 
context. Unlike functions, closures can capture values from the scope in which they’re defined.

There are more differences between functions and closures. Closures don’t usually require you to annotate 
the types of the parameters or the return value like fn functions do.

```rust
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

Closures can capture values from their environment in three ways, which directly map to the three ways a
function can take a parameter: borrowing immutably, borrowing mutably, and taking ownership. The closure
will decide which of these to use based on what the body of the function does with the captured values.

If you want to force the closure to take ownership of the values it uses in the environment even though
the body of the closure doesn’t strictly need ownership, you can use the move keyword before the parameter 
list.


### Fn, FnMut, FnOnce

The way a closure captures and handles values from the environment affects which traits the closure 
implements, and traits are how functions and structs can specify what kinds of closures they can 
use. Closures will automatically implement one, two, or all three of these Fn traits, in an additive 
fashion, depending on how the closure’s body handles the values:

- `FnOnce` applies to closures that can be called once. All closures implement at least this trait, 
because all closures can be called. A closure that moves captured values out of its body will 
only implement FnOnce and none of the other Fn traits, because it can only be called once.
- `FnMut` applies to closures that don’t move captured values out of their body, but that might mutate 
the captured values. These closures can be called more than once.
- `Fn` applies to closures that don’t move captured values out of their body and that don’t mutate 
captured values, as well as closures that capture nothing from their environment. These closures 
can be called more than once without mutating their environment, which is important in cases such 
as calling a closure multiple times concurrently.