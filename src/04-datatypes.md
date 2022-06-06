# Datatypes

Any data that exists must have a type associated with it, and since Scribe is statically typed, these datatypes must be known when the program is compiled.

Datatypes are abstractions over binary sequences that define what kind of data a variable contains, and how that data is represented in memory.

If you have any prior experience in programming, you probably know datatypes quite well.

Core datatypes in Scribe are:

| Types in C           | Scribe Equivalent |
|----------------------|-------------------|
| void                 | void              |
| bool                 | i1                |
| char                 | i8                |
| short                | i16               |
| int                  | i32               |
| long int             | i64               |
| unsigned char        | u8                |
| unsigned short       | u16               |
| unsigned int         | u32               |
| unsigned long int    | u64               |
| float                | f32               |
| double               | f64               |

Similar to C, Scribe contains pointers as well.

And the string literals are represented as `*const i8` (equivalent to `const char *` in C).

Let's dive a bit into the core datatypes.

# Core Datatypes - Primitive Types

## Integers

Scribe contains a total of 9 integer types varying in bit count and signed vs unsigned.

The signed variants are: `i1`, `i8`, `i16`, `i32`, and `i64`. The unsigned variants are: `u8`, `u16`, `u32`, and `u64`.

Depending on requirement, anyone of these can be used.

For example,

```rs
let a = 32;      // type: i32
let b: u64 = 64; // type: u64
let c = true;    // type: i1
```

## Floating Points

There are 2 floating-point types in Scribe - `f32` and `f64`.

For example,

```rs
let a = 3.2;      // type: f32
let b: f64 = 6.4; // type: f64
let c = 3.0;      // type: f32
```

The decimal point (`.`) is necessary to differentiate between an integer and a floating-point number.

## Booleans

In Scribe, booleans are represented using the `i1` type. `bool` is available as well and is an alias to `i1`.

For example,

```rs
let a = true;        // type: i1
let b: bool = false; // type: i1
let c: i1 = true;    // type: i1
```

## Characters

Characters are nothing but `i8` in Scribe. However, similar to C, characters are represented within single quotes (`'`).
These characters are byte-sized and no more than one character can be present within the single quotes.

Therefore, unicode characters must be defined as strings (within double-quotes).

For example,

```rs
let a = 'a'; // type: i8
```

## Strings

The fundamental "string" in Scribe is the same as in C - pointer to i8 (char). All string literals have the type `*const i8` (C's equivalent to `const char *`).

For example,

```rs
let a = "Hello World"; // type: *const i8
```

Scribe, unlike C however, also contains a `String` type in standard library. The `String` type and C-style strings can be converted to each other using provided functions (more in the String standard library chapter of this manual).

## Enums

Similar to C, Scribe contains enums which have integer values. However unlike C, enum variables cannot be used by themselves. They must be preceded by the enum name and dot (`.`).

This allows for having same enum tags across multiple enums without them clashing with each other.

For example,

```rs
let A = enum {
	ZERO,
	ONE,
	TWO
};

let B = enum {
	ZERO, // no clash here
	FIRST,
	SECOND,
};

let zero_in_a = A.ZERO;
let zero_in_b = B.ZERO;

let main = fn(): i32 {
	let t = A.ONE == B.FIRST; // true - both contain value 1
	let f = A.ONE == B.ZERO;  // false - 1 != 0
	return 0;
};
```

## Structures

Scribe provides structures as a data structure to pack multiple (existing) types/structures into a single unit. Similar to C, structure members are accessed using the dot (`.`) operator.

To instantiate a structure, the structure is "called" using braces (`{...}`) with the member values as arguments.

For example,

```rs
let A = struct {
	i: i32;
	f: f64;
};

let B = struct {
	u: u64;
	a: A; // B contains variable a, which is of type structure A
};

let main = fn(): i32 {
	let a = A{3, 2.5};
	let b = B{1, a};
	let c = B{10, A{3, 5.0}};
	let p = a.i + b.u;
	let q = a.f + b.a.f;
	let r = c.u + c.a.f;
	return 0;
};
```

Unlike C however, pointers to structures are deduced internally, so even for structure pointers, the dot (`.`) operator applies (no arrow (`->`) or dereference (`*`) operator required).
We will see this behavior when we understand pointers in Scribe.

Scribe also allows for "member functions" for structures. However, the term "member functions" is an incorrect term for Scribe because these functions are not defined inside the structure itself.
In this language, functions can be **associated** to structures, hence referred to as "associated functions". These functions work on structures, but can be defined anywhere after the structure itself is created.

For example,

```rs
let A = struct {
	i: i32;
	f: f64;
};

let getI in A = fn(): i32 {
	return self.i; // every associated function will have a "self" variable which is a reference to the struct (instance) to which the function is associated
};

let main = fn(): i32 {
	let a = A{1, 2.5};
	let x = a.getI(); // x = 1
	return 0;
};
```

## Arrays

Scribe provides the ability to create arrays - a contiguous sequential collection of data of a single structure/type.
The array size must be known at compile time, that is, the array length must be known when writing the code itself.

For example,

```rs
let arr1 = @array(i32, 10);     // 1D (size: 10) array of type i32
let arr2 = @array(i32, 10, 10); // 2D (size: 10x10) array of type i32

let A = struct {
	i: i32;
	f: f64;
};

let arr3 = @array(A, 10); // 1D (size: 10) array of type structure A
let data1 = arr3[1].i;    // data in arr3's member 'i' at the first index
```

Scribe does not have a specific array type notation (usually something like `int[5]` in most languages). Instead, in Scribe, arrays are passed around as pointers themselves.

An important thing to note about arrays in Scribe is that they are always initialized to zero values. That is, unlike C, the array values are never undefined.

## Pointers

Pointers allow the use of heap memory to dynamically (at runtime) allocate memory which can be of variable size.
These variables usually store either memory addresses to other variables, or the memory address to some data on the heap.
They may also not point to any data - when their value is zero (`nil`).

Note that any memory allocated during runtime by the program must be deallocated by the program as well.

For example,

```rs
let c = @import("std/c"); // contains C functions

let main = fn(): i32 {
	let a = 5;
	let ptr_a = &a;
	let data_a = *ptr_a;
	*ptr_a = 10; // now a is also 10

	let arr = mem.alloc(i32, 10); // allocate memory equal to 10 i32's at runtime
	arr[1] = 5; // set value 5 at index 1 of arr
	mem.free(i32, arr); // deallocate memory which was allocated by malloc
	return 0;
};
```

## Special Types

Scribe contains some special types for specific use cases. These are a bit advanced and must not be used unless needed (you will know if they're needed).

These types are used solely in function signatures. As such, they are not used anywhere else.

### Type

`type` is a special type, a variable of which can contain a type. This is all compile time only and no code is generates for this.
This is used with generics (more on generics later) to pass datatypes as argument. A variable of type `type` **must** be declared `comptime`.

This is how `mem.alloc()` function, used above, works. We can pass `i32` type to malloc because its signature's first parameter is of type `type`.

For example,

```rs
let f = fn(comptime T: type, data: T): T {
	return data * data;
};

let main = fn(): i32 {
	let p = f(i32, 10);  // p is of type i32, with value 100
	let q = f(f32, 2.5); // q is of type f32, with value 6.25
	return 0;
};
```

### Any

`any`, as you may have guessed, allows variable of any type to be passed to a function.
If used as return type, it allows the compiler to deduce what type is being returned by the function through the return statements present inside the function.

```rs
let f = fn(data: any): any {
	return data * data;
};

let main = fn(): i32 {
	let p = f(i32, 10);  // p is of type i32, with value 100
	let q = f(f32, 2.5); // q is of type f32, with value 6.25
	let comptime a = f(i32, 10);  // a is of type i32, with value 100, set at compile time
	let comptime b = f(f32, 2.5); // b is of type f32, with value 6.25, set at compile time
	return 0;
};
```

# Conclusion

Well, this was a bit long, but here we cover all the core datatypes in Scribe. There is a lot more to these datatypes but that is out of scope of this manual.

Take your time to understand these datatypes as they are incredibly useful and important for any programming language. Now, we will move on to functions.
