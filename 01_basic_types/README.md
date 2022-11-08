# Basic types

A scalar type represents a single value. Rust has four primary scalar types: integers,  
floating-point numbers, booleans, and characters.

## Integers
Signed numbers are stored using two’s complement representation. Note that number literals  
that can be multiple numeric types allow a type suffix, such as `57u8`.

|  Length   | Signed | Unsigned | 
|:---------:|:------:|:--------:|
|   8-bit   |   i8   |    u8    |
|  16-bit   |  i16	  |   u16    |
|  32-bit   |  i32	  |   u32    |
|  64-bit   |  i64	  |   u64    |
| 128-bit 	 | i128	  |   u128   |
|   arch    | isize  |  usize   |

Integer division rounds down to the nearest integer.

### Integers overflow

Let’s say you have a variable of type u8 that can hold values between 0 and 255. If you
try to change the variable to a value outside of that range, such as 256, integer overflow  
will occur, which can result in one of two behaviors. When you’re compiling in debug mode,  
Rust includes checks for integer overflow that cause your program to panic at runtime if  
this behavior occurs.

When you’re compiling in release mode with the --release flag, Rust does not include  
checks for integer overflow that cause panics. Instead, if overflow occurs, Rust performs  
two’s complement wrapping. 

To explicitly handle the possibility of overflow, you can use these families of methods  
provided by the standard library for primitive numeric types:

- Wrap in all modes with the `wrapping_*` methods, such as `wrapping_add`
- Return the `None` value if there is overflow with the `checked_*` methods
- Return the value and a boolean indicating whether there was overflow with the 
`overflowing_*` methods
- Saturate at the value’s minimum or maximum values with `saturating_*` methods

## Floating points

The two types are `f32` and `f64`. All floating-point types are signed. Floating-point  
numbers are represented according to the IEEE-754 standard. The f32 type is a 
single-precision  float, and f64 has double precision.

## Bools

A boolean type in Rust has two possible values: true and false. Booleans are one byte in size. 

## Char

Rust’s char type is the language’s most primitive alphabetic type. Note that we specify 
char  literals with single quotes, as opposed to string literals, which use double quotes.  
Rust’s char type is four bytes in size and represents a Unicode Scalar Value, which means  
it can represent a lot more than just ASCII.

# Compound types

Compound types can group multiple values into one type. Rust has two primitive compound  
types: tuples and arrays.

## Tuple

A tuple is a general way of grouping together a number of values with a variety of types 
into  one compound type. Tuples have a fixed length: once declared, they cannot grow or  
shrink in size.

We create a tuple by writing a comma-separated list of values inside parentheses. Each  
position in the tuple has a type, and the types of the different values in the tuple don’t 
have to be the same.

Tuple elements can be accessed directly by using a period (.) followed by the index of the 
value we want to access. Tuples can be destructured using pattern matching.

```rust
fn func() {
    // tuple with explicit type declaration
    let x: (i32, f64, u8) = (500, 6.4, 1);
    let five_hundred = x.0;
    let six_point_four = x.1;
    let one = x.2;
    
    // destructuring a tuple
    let y = (500, 6.4, 1);
    let (a, b, c) = y;
}
```

The tuple without any values has a special name, unit. This value and its corresponding  
type are both written () and represent an empty value or an empty return type. Expressions  
implicitly return the unit value if they don’t return any other value.

## Array

Another way to have a collection of multiple values is with an array. Unlike a tuple, every 
element of an array must have the same type. Unlike arrays in some other languages, arrays  
in Rust have a fixed length. Arrays are useful when you want your data allocated on the 
stack  rather than the heap.

```rust
fn func() {
    // explicit type signature
    let a: [i32; 5] = [1, 2, 3, 4, 5];
    // type inferred (if possible)
    let b = [1, 2, 3, 4, 5];
    let one = b[0];
    let five = b[4];
    
    // five times 3
    let c = [3; 5];
}
```

If you try to access an element of an array that is past the end of the array the program 
will crash at runtime.  In many low-level languages, this kind of check is not done, and when
you provide an incorrect index, invalid memory can be accessed. Rust protects you against 
this kind of error by immediately exiting instead of allowing the memory access and continuing.

## Struct