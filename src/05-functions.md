# Functions

Functions are groups of instructions, combined to perform specific tasks.
Using functions, we can avoid having to repeat a task's code over and over again whenever we want to perform the task.

Functions are quite pervasive in programming languages - there would be rarely any programming language without these.

When we create a function with its group of instructions, it is called **function definition**.
The group of instructions/statements is called **function body**. And, the usage of the function, it is called a **function call**.

We may need to provide functions with some additional data, or retrieve some data from it. For that, we use function **arguments** and function **return values** respectively.

# Scribe Functions

There are a variety of functions in Scribe. Some of them are:

* Simple Functions
* Intrinsic Functions
* Associated Functions
* Callback Functions

Let's go over each of them.

## Simple Functions

These are the most basic functions in Scribe. The `main` function, for example, is actually a simple functions.
Like any other function, these may or may not take arguments and which may or may not return data.

For example,

```rs
let io = @import("std/io");

let greet = fn(whom: *const i8) { // simple function - takes a *const i8 as argument, returns nothing (void)
	io.println("Hello ", whom);
};

let main = fn(): i32 { // simple function - takes no argument, returns an i32
	greet("Electrux");
	return 0;
};
```

## Intrinsic Functions

These functions are built into the compiler and cannot be created in Scribe code. They never emit any code and perform their task while the code is being compiled.

We have been using intrinsic functions all along - `@import()` is an intrinsic function which instructs the compiler to import a source file during compilation.

As you may have guessed, intrinsic functions are always prepended by `@` when they are called.

Another intrinsic that we have used is the `@array()` function with which array creation is possible.

For example,

```rs
let io = @import("std/io");
let arr = @array(i32, 10);
```

## Associated Functions

These functions are associated to a structure or type. Yes, functions can be associated to built-in types (like `i1`, `i32`, `f32`, etc.) as well, except for pointer and array types.

Such functions work on the instance of the structure/type they are associated to.

Internally, these functions are provided a `self` argument which is a reference to the instance itself.

For example,

```rs
let io = @import("std/io");

let A = struct {
	i: i32;
};

let getI in A = fn(): &i32 { // return reference to the member "i"
	return self.i;
};

let disp in i32 = fn() {
	io.println(self);
};

let main = fn(): i32 {
	let a = A{1};
	let p = a.getI(); // p is now a reference to a.i because getI() returns a reference
	p = 20;
	p.disp();         // displays 20
	a.getI().disp();  // displays 20 - p was a reference to a.i
	a.getI() = 10;    // works because getI() returns a reference
	a.getI().disp();  // displays 10
	return 0;
};
```

Associated functions may be created anywhere in the code as long as they are created **before** their usage.

There are times when you want to created associated functions on constant instances of types.
For example, a `toStr()` function for an `i32` does not require the `i32` variable to be modifiable, and this function should be usable on `const i32` as well.

Scribe can handle this situation. A specification detail of Scribe is that the type/struct after `in` in a let statement is **not** a type. It is a **type expression**.

Therefore, for our `const` usecase, all you need to do is create the function as:

```rs
let toStr in const i32 = fn() {
	// ...
};
```

Quite convenient, right?!

## Callback Functions

Callback functions are simple functions, but they are passed around to other functions so that the other function can call them.

That may sound weird, but this is a common concept which is used in many languages including Javascript (most notably), C, and C++.

For example,

```rs
let io = @import("std/io");

let add5 = fn(data: i32): i32 {
	return data + 5;
};

let sub5 = fn(data: i32): i32 {
	return data - 5;
};

let f = fn(data: i32, cb: fn(x: i32): i32) {
	io.println("Result is: ", cb(data));
};

let f2 = fn(data: i32, cb: any) {
	io.println("Result is: ", cb(data));
};

let main = fn(): i32 {
	f(10, add5); // Result is: 15
	f(10, sub5); // Result is: 5
	f2(10, sub5); // Result is: 5
	return 0;
};
```

Note that the function to which callback is passed **must** have the callback argument with same function signature as the callback function itself.

The special type `any` can be used if desired, but that is more error prone since anything can be provided for `any` instead of just functions with specific signatures.
