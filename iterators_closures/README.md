
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
because all closures can be called. A closure that moves captured values out of its body (or drops
them) will only implement FnOnce and none of the other Fn traits, because it can only be called once.
- `FnMut` applies to closures that don’t move captured values out of their body, but that might mutate 
the captured values. These closures can be called more than once.
- `Fn` applies to closures that don’t move captured values out of their body and that don’t mutate 
captured values, as well as closures that capture nothing from their environment. These closures 
can be called more than once without mutating their environment, which is important in cases such 
as calling a closure multiple times concurrently.

Every Fn meets the requirements for FnMut, and every FnMut meets the requirements for FnOnce. They’re
not three separate categories. Instead, Fn() is a subtrait of FnMut(), which is a subtrait of FnOnce().
This makes Fn the most exclusive and most powerful category.

### Example FnOnce

Using FnOnce in the trait bound expresses the constraint that `unwrap_or_else` is only going to call
`f` at most one time. Every closure trait is a FnOnce so all can be used in place of a FnOnce.

```rust
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}

fn example() {
    // The closure is a FnOnce because it consumes the string,
    // so it can be called only one time. It matches the method 
    // signature so it is accepted.
    let non_copy_val = String::from("Ehy");
    let mut my_fn_once = || {
        drop(non_copy_val);
        String::new()
    };

    None.unwrap_or_else(my_fn_once);

    // The closure is a FnMut because it modifies the string,
    // so it can be called multiple times. It is a subtype of
    // FnOnce so it is accepted by unwrap_or_else.
    let mut non_copy_val = String::from("Ehy");
    let mut my_fn_mut = || {
        non_copy_val.push_str(" guys");
        String::new()
    };
    my_fn_mut();
    my_fn_mut();

    None.unwrap_or_else(my_fn_mut);

    // The closure is a Fn because it doesn't modify the 
    // string, so it can be called multiple times. It is a 
    // subtype of FnOnce so it is accepted by unwrap_or_else.
    let my_fn = || {
        println!("{:?}", non_copy_val);
        String::new()
    };
    my_fn();
    my_fn();

    None.unwrap_or_else(my_fn);
}
```

### Example FnMut

The following example shows how the FnMut traits works. FnMut is a subtype of FnOnce so
FnOnce closures doesn't satisfy FnMut, while Fn closures do.

```rust
// Map requires a closure that can be called multiple times
// even if it mutates the surrounding captured environment.
fn map<V, U, F>(list: Vec<V>, mut map_fn: F) -> Vec<U>
    where
        F: FnMut(V) -> U,
{
    let mut out = Vec::with_capacity(list.len());
    for l in list {
        out.push(map_fn(l))
    }
    out
}

fn example() {
    // ❌ The closure is FnOnce because it moves out a variable from
    // its environment, so it doesn't meet the `map` requirements.
    let mut strings: Vec<String> = vec![];
    let s = "str".to_string();
    let my_fn_once = |n| {
        strings.push(s);
        n + 1
    };

    let res = map(vec![1, 2, 3, 4], my_fn_once); // ❌ doesn't compile
    println!("result: {:?}", res);
    println!("strings: {:?}", strings);

    // The closure is FnMut because it mutates the environment but
    // doesn’t capture, mutate, or move out anything from its
    // environment, so it meets the `map` bound requirements.
    let mut sum = 0;
    let my_fn_mut = |n| {
        sum += n;
        n + 1
    };

    let res = map(vec![1, 2, 3, 4], my_fn_mut);
    println!("result: {:?}, count: {:?}", res, sum);

    // The closure is a Fn because it doesn't modify the environment.
    // It is a subtype of the required FnMut trait so it is accepted
    // by the `map` function.
    let my_fn = |n| {
        format!("num: {:?}", n)
    };

    let res = map(vec![1, 2, 3, 4], my_fn);
    println!("result: {:?}", res);
}
```

### Example Fn

The following example shows how the Fn traits works. Fn is a subtype of FnOnce and FnMut so
FnOnce and FnMut closures doesn't satisfy Fn.

```rust
// Map requires a closure that can be called multiple times
// even if it mutates the surrounding captured environment.
fn requires_fn<F: Fn()>(my_fn: F) {
    my_fn();
    my_fn();
    my_fn();
}

fn example() {
    // ❌ The closure is FnOnce because it moves out a variable from
    // its environment, so it doesn't meet the `map` requirements.
    let mut strings: Vec<String> = vec![];
    let s = "str".to_string();
    let my_fn_once = |n| {
        strings.push(s);
        n + 1
    };

    let res = requires_fn(vec![1, 2, 3, 4], my_fn_once); // ❌ doesn't compile
    println!("result: {:?}", res);
    println!("strings: {:?}", strings);

    // ❌ The closure is FnMut because it mutates the environment but
    // doesn’t capture, mutate, or move out anything from its
    // environment. It doesn't meet the stricter Fn requirements.
    let mut sum = 0;
    let my_fn_mut = |n| {
        sum += n;
        n + 1
    };

    let res = requires_fn(vec![1, 2, 3, 4], my_fn_mut); // ❌ doesn't compile
    println!("result: {:?}, count: {:?}", res, sum);

    // The closure is a Fn because it doesn't modify the environment.
    // It meets the function requirements.
    let my_fn = |n| {
        format!("num: {:?}", n)
    };

    let res = requires_fn(vec![1, 2, 3, 4], my_fn);
    println!("result: {:?}", res);
}
```

### Move

If you want to force the closure to take ownership of the values it uses in the environment even 
though the body of the closure doesn’t strictly need ownership, you can use the `move` keyword 
before the parameter list.

```rust
use std::thread;

fn example() {
    let list = vec![1, 2, 3, 4, 5, 6, 7];
    println!("Before: {:?}", list);
    
    // It could be only borrowed, but `move` forces
    // the list to be moved inside the closure.
    let some_seven = Some(7_u16).unwrap_or_else(move || {
        println!("From thread: {:?}", list);
        0
    });

    // ❌ doesn't compile, `list` was moved into the closure
    println!("After: {:?}", list);
}
```

Using move doesn't affect if the closure is a `Fn`, `FnMut` or `FnOnce`. This is determined 
only by how the code treats the captured/moved values.

```rust
fn example() {
    // This closure uses the `move` keyword but 
    // it is a Fn closure nonetheless.
    let list = vec![1, 2, 3];
    let my_fn = move || println!("From thread: {:?}", list);
    requires_fn(my_fn);
}
```