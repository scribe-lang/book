# Creating Variables

Variables are pretty much aliases that the programmers make for using memory to allocate some data.
Obviously, we do not want, for our sanity's sake, to use memory addresses directly for accessing our data.
That is really complicated, hard to remember, and an absolute nightmare to maintain.
So instead, we assign these memory locations, containing our data, names which we can use in our programs as needed.

As such, variables are a crucial component of a programming language.

There are two ways to create variables in Scribe.

## Definition

To define a variable in Scribe, the `let <identifier> = <expression>;` pattern is used.

The identifier will be the variable name, followed by the value expression after the assignment (`=`) symbol, terminated by a semicolon (`;`).

Since Scribe is a statically typed language, the type of each variable is necessary. However, in this (definition) format, the datatype is **inferred**.
In other words, the compiler can deduce the type of the variable from the expression.

This is the syntax used to import the `std/io` module in the Hello World example.

For example:

```rs
let an_int = 20;                   // type: i32
let a_string = "this is a string"; // type: *const i8 (just like C)
let a_flt = 2.5;                   // type: f32 (float)
```

## Declaration

To declare a variable, the `let <identifer>: <type>;` pattern is used.

As before, the identifier will be the variable name, followed by the datatype of the variable after a colon (`:`), terminated by a semicolon (`;`).

Here, we explicitly mention the datatype of the variable since there is no expression to deduce it from.

For example:

```rs
let an_int: u64; // type: u64 (unsigned 64 bit integer)
let a_flt: f64;  // type: f64 (64 bit float)
```

## Notes

Both the styles can be merged as well, as long as the `type` and `expression`'s type are same. The only exception is automatic type casting between primitive datatypes.

For example, if you want to create a variable with value `5` (which has type: `i32`), but of type: `u64`, you can do:

```rs
let u64int: u64 = 5;
```

# Variable (Re)assignment

A variable won't quite be a "variable" if its value cannot be changed. Therefore, to change (or set) the value of a variable, you can use the variable name, followed by the assignment (`=`) symbol, followed by the value.

Do note that the datatype of the variable can never be changed after a variable is created. Hence, for (re)assignment, the datatype of the new value must be the same as the one used/inferred when creating the variable.

For example:

```rs
let a = 5;
a = 10; // valid
a = "a string"; // invalid
```

Similarly,

```rs
let a: u64;
a = 10; // valid
a = "a string"; // invalid
a = 5.0; // valid
```

## Type Coercion

You may be wondering how is `a = 5.0;` valid. The datatype of a is `u64`, while that of `5.0` is `f32`.

The reason is that the compiler can automatically "coerce" between primitive (`i1, i8, i16, i32, i64, u8, u16, u32, u64, f32, f64`) types.
In other words, the data from these types can be used with a value/variable of another of these types.

This does come at a loss of data itself (about which more can be found on the Internet), therefore one must be careful when writing code that can be automatically type coerced.

# Variable Scope

The scope of a variable defines its `lifetime` - when is it created to when it is destroyed. Scribe, like many languages, uses `braces` (`{` and `}`) to define blocks of code.
Any variable has its lifetime bound to this block - it cannot be used outside this block.

For example:

```rs
let a = 5;
```

This is globally scoped - not contained within any block. Therefore, this variable can be used anywhere after its creation.

Another example:

```rs
let main = fn(): i32 {
	let a = 5;
};
```

Here, the variable `a` can be used anywhere **within** the braces of the `main` function (and of course, **after** the creation).

Scribe also supports `variable shadowing`. That is, if there is a variable in a parent block, a new variable can be created of the same name, within a new block, inside the parent block.

For example:

```rs
let main = fn(): i32 {
	let a = 5;
	{
		let a = "a string";
	}
};
```

The above code is valid. Within the `main` function's block, the integer variable `a` is usable. And within the block inside the `main` function, the string variable `a` is usable.

# Variable Names

One last important thing about variables is their name. Scribe defines specific rules based on which you can name variables, quite similar to most other languages.
These rules are that variable names:

* Must begin with an alphabet (irrelevant of the case) or underscore
* Can contain numbers anywhere except the first character
* Cannot contain any symbol other than alphabets, numbers, and underscores.

# Variable Type Modifiers

Type modifiers alter the working of a variable - what they store, how they work, etc.

There are 4 type modifiers in Scribe:
* Static
* Reference
* Constant
* Comptime

## Static

The static modifier allows the variable to be stored in global storage - the variable is created only once.
If such a variable is present inside a function, the variable will be initialized only once and it will maintain its value across function calls.

For example,

```rs
let io = @import("std/io");

let f = fn(): i32 {
	let static x = 5;
	return x++;
};

let main = fn(): i32 {
	io.println(f()); // prints 5
	io.println(f()); // prints 6
	io.println(f()); // prints 7
	return 0;
};
```

## Reference

Reference variables are the same as in C++ - they are references to other variables. In other words, reference variables can be considered as aliases to other variables.

These variables are created by using `&` with the type.

For example,

```rs
let f = fn(data: &i32) {
	data = 10;
};

let main = fn(): i32 {
	let a = 5;
	f(a);
	// now a is 10
	return 0;
};
```

## Constant

Variables, in general, can be modified. However, there are times when the programmer does not want a variable to be modifiable - it must stay constant. That's what the constant modifier allows.

Constant variables, once created, can never be modified again. Attempting to modify them is a compilation error.

For example,

```rs
let main = fn(): i32 {
	let const i = 10;
	i = 20; // error: i is a constant
	return 0;
};
```

## Comptime

Scribe contains a pretty nifty and useful feature - comptime variables. A comptime variable is special in the sense that the value of such a variable is evaluated during the code compilation.
Hence, some conditions apply for a variable to be allowed to be comptime:
1. There must be a value expression for the variable
2. The value expression must be able to be evaluated during compile time itself - it cannot contain data that cannot be computed during compilation

For example,

```rs
let comptime a = 50; // valid - 50 is a value which is known by the compiler
let f = fn(x: i32): i32 {
	return x * 5 + 25;
};

let comptime b = f(10); // valid - function call is provided a value known by the compiler, and everything inside the function body is computable by the compiler

let io = @import("std/io");

let f2 = fn(x: i32): i32 {
	io.println(x);
	return x * 5 + 25;
};

let comptime c = f2(10); // invalid - io.println() does not work at compile time

let arr = @array(i32, 10);

let comptime x = arr[1]; // valid - @array() creates an array and initializes it to zero at compile time
```

# Conclusion

Well, that is basically how variables, their reassignment, and their scopes, work. Not much to learn or understand and pretty easy - which is the goal!

Next, we'll understand the concept of datatypes and see some of the fundamental (primitive) datatypes in Scribe.
