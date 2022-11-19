
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

    // The .into() call doesn't consume the original
    // value because it operates on a cloned value.
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
