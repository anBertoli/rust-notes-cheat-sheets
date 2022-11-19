
### Generics with functions 

To parameterize the types in a new single function, we need to name the type parameter, just
as we do for the value parameters to a function (in the example the parameter is named `T`).
The generic type is usually costarring by a type, that is, the concrete type must implement
the trait to be used in the function.

```rust
// accepts every type that implements PartialOrd
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut max = &list[0];
    for item in list {
        if item > max {
            max = item;
        }
    }
    max
}

fn use_largest() {
    // use it with different types, both i32 
    // and &str implement the PartialOrd trait
    let max = largest(&[4,6,10,9]);
    println!("{:?}", max);
    let max = largest(&["ehy", "hello", "world"]);
    println!("{:?}", max);

    // functions can specify the concrete type
    // with the "turbofish" notation 
    let max = largest::<u32>(&[4,6,10,9]);
    println!("{:?}", max);
    let max = largest::<i64>(&[4,6,10,9]);
    println!("{:?}", max);
}
```

Multiple generic types can be defined in a function signature, each with multiple trait 
bounds. The `where` clause can be used to make things clearer in a function signature.

```rust
use std::ops::AddAssign;

fn clone_and_add_to_all<T, U, V>(seq: T, num: V) -> Vec<U>
where
    T: AsRef<[U]>,
    U: AddAssign<U> +  Clone,
    V: Into<U>,
{
    // Convert the number into the proper type, thanks to
    // the Into trait implemented on the parametric type 'V'.
    let num_converted = num.into();

    // The to_vec() method is implemented for all slices
    // composed by 'Clone'able types, as U in this case.
    let seq_ref: &[U] = seq.as_ref();
    let mut vec: Vec<U> = seq_ref.to_vec();

    // The value is cloned() so the original
    // value is not consumed.
    for item in &mut vec {
        *item += num_converted.clone()
    }

    vec
}

fn use_add_to_all() {
    let vec: Vec<u32> = vec![1,2,3,4,5];
    let vec_2 = clone_and_add_to_all(&vec, 10_u16);
    println!("{:?}", vec_2);

    let vec: Vec<u64> = vec![1,2,3,4,5];
    let vec_2 = clone_and_add_to_all(&vec, 10_u16);
    println!("{:?}", vec_2);

    let vec: Vec<i64> = vec![1,2,3,4,5];
    let vec_2 = clone_and_add_to_all(&vec, 10_u32);
    println!("{:?}", vec_2);

    // ⚠️ doesn't work, u64 cannot be converted into
    // i64 safely, so Into<i64> is not implemented
    //      let vec: Vec<i64> = vec![1,2,3,4,5];
    //      let vec_2 = add_to_all(&vec, 10_u64);
}
```


#### Lifetime elision rules

They’re a set of particular cases that the compiler will consider, and if your code fits 
these cases, you don’t need to write the lifetimes explicitly. The compiler uses three rules 
to figure out the lifetimes of the references when there aren’t explicit annotations. The 
first rule applies to input lifetimes, and the second and third rules apply to output lifetimes. 
If the compiler gets to the end of the three rules and there are still references for which it 
can’t figure out lifetimes, the compiler will stop with an error. These rules apply to fn 
definitions as well as ´impl´ blocks.
 
- The first rule is that the compiler assigns a lifetime parameter to each parameter that’s a reference. 
In other words, a function with one parameter gets one lifetime parameter: fn foo<'a>(x: &'a i32); a 
function with two parameters gets two separate lifetime parameters: fn foo<'a, 'b>(x: &'a i32, y: &'b i32); 
and so on.

- The second rule is that, if there is exactly one input lifetime parameter, that lifetime is assigned 
to all output lifetime parameters: fn foo<'a>(x: &'a i32) -> &'a i32.

- The third rule is that, if there are multiple input lifetime parameters, but one of them is &self or 
&mut self because this is a method, the lifetime of self is assigned to all output lifetime parameters.
This third rule makes methods much nicer to read and write because fewer symbols are necessary.

### All in one

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```
