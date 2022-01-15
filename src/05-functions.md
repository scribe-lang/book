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
* Variadic Functions

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


## Variadic Functions

These functions are special in that they have a parameter which is variadic - the function call can have infinite number of arguments in place of the variadic parameter.

There can be only one variadic parameter in a function and it is declared variadic by prepending triple dots (`...`) with its type.

For anyone coming from C, the `printf()` function in C is a variadic function.

Unlike C however, Scribe's variadic parameters do not require any type casting and the compiler always knows the types stored in the variadic parameter.

Combined with the `any` type, variadic parameters can be used to take any provided argument without limitation on the type of the argument.
This makes variadic functions quite powerful and very useful. The `io.println()` function that we have been using, for example, is a variadic function - it can take an infinite list of comma separated arguments.

For example,

```rs
let sum = fn(args: ...i32): i32 { // args is a variadic which accepts all arguments of type i32
	let comptime len = @valen(); // @valen() is an intrinsic which provides the number of variadic arguments (>= 0) with which this function is called
	let sum = 0;
	inline for let comptime i = 0; i < len; ++i { // we will learn more about inline loops later; do note that here 'inline' and 'comptime' must be present
		sum += args[i];
	}
	return sum;
};

let main = fn(): i32 {
	let s1 = sum(1); // s1 = 1 at runtime
	let comptime s2 = sum(1, 2, 3); // s2 = 6 at compile time
	return 0;
};
```

Variadic arguments can also be "unpacked" and passed to other variadic functions just by using the variadic parameter name.

For example,

```rs
let sum = fn(args: ...i32): i32 {
	let comptime len = @valen();
	let sum = 0;
	inline for let comptime i = 0; i < len; ++i {
		sum += args[i];
	}
	return sum;
};

let pass = fn(data: ...i32): i32 {
	return sum(data, 5); // unpacks data and passes all the provided variadic arguments, with 5 at the end, to sum()
};

let main = fn(): i32 {
	let s1 = pass(1); // s1 = 6 at runtime
	let comptime s2 = pass(1, 2, 3); // s2 = 11 at compile time
	return 0;
};
```

You can also perform your own type check on variadic arguments at compile time to ensure the given arguments' types are one of specified types.

Since these are completely compile time, they do not generate any code at runtime and hence will not impact performance.

For example, to limit a variadic function to work with types `i32` and `i64` only,

```rs
let sum = fn(args: ...any): i32 {
	let comptime len = @valen();
	inline for let comptime i = 0; i < len; ++i {
		inline if !@isEqualTy(args[i], i32) && !@isEqualTy(args[i], i64) {
			@compileError("Expected argument type to be either i32 or i64, found: ", @typeOf(args[i]));
		}
	}
	let sum: i64 = 0;
	inline for let comptime i = 0; i < len; ++i {
		sum += args[i];
	}
	return sum;
};

let main = fn(): i32 {
	let s1 = sum(1);
	let comptime s2 = sum(1, 2, 3, 4.5); // compilation fails - 4.5 is neither i32 nor i64
	return 0;
};
```

For those who are accustomed to programming and variadics, feel free to check out the standard library's [std/io](https://github.com/scribe-lang/scribe/blob/main/headers/std/io.sc) module, which contains multiple variadic functions.

# Conclusion

Well, that is fundamentally how functions in Scribe work. It's recommended to write some sample programs to understand and ease into them.

Next up, we'll be looking at Generics.
