# Conditionals

Conditionals are decision making constructs in a programming language.
Basically, if we want to perform some action based on specific condition or criteria, we use conditionals.

In Scribe, there are two variants of conditionals:

* Simple Conditionals
* Inline Conditionals

## Simple Conditionals

These are the usual conditionals that exist in most programming language. In Scribe, the following syntax is followed:

```rs
if <expression> {
	// do this if expression yields a truthy value
} elif <expression> {
	// do this if the first if fails, but elif's expression yields a truthy value
} else {
	// do this if none of the if's and elif's yield a truthy value
}
```

The `elif` and `else` sections are optional and may be skipped as required.

A "truthy value" is one of:

* boolean `== true`
* integer `!= 0`
* float `!= 0.0`
* pointer `!= nil`

For example,

```rs
let io = @import("std/io");

let main = fn(): i32 {
	if true { // boolean true is a truthy value, therefore the following block is executed
		io.println("This is displayed");
	}
	let ptr: *i32 = nil;
	if ptr { // nil is not a truthy value, therefore the else block is executed
		io.println("Pointer points to some data");
	} else {
		io.println("Pointer points to no data");
	}
	let a = 2;
	if a == 1 { // this condition fails therefore the following block is not executed
		io.println("a is 1");
	} elif a == 2 { // this condition succeeds, therefore the following block is executed
		io.println("a is 2");
	} else {
		io.println("a is neither 1 nor 2");
	}
	return 0;
};
```

## Inline Conditionals

These are special conditionals that are evaluated at compile time.
This means that the compiler will execute the conditional during compilation and whichever section yields a truthy value, that section will be left in the code.
The other sections of the conditional will be removed completely.

Also, if none of the sections evaluate to a truthy value, the entire conditional will be removed.

Note that the conditional's block will not executed like it is during runtime. Instead, the block will be placed as long as its conditonal yields a truthy value.

For example,

```rs
let A = struct {
	data: i32;
};

let getData in A = fn(): i32 {
	return self.data;
};

let get = fn(from: &any): i32 {
	inline if @isEqualTy(from, A) {
		return from.getData();
	} elif @isEqualTy(from, i32) {
		return from;
	} else {
		@compileError("Cannot return i32 from data of type: ", @typeOf(from));
	}
};

let main = fn(): i32 {
	let a = A{5};
	let b = 20;
	let c = "a string";
	let a1 = get(a); // calls a.getData()
	let b1 = get(b); // returns b
	let c1 = get(c); // compilation fails (@compileError() is called) - c is neither an instance of A nor an i32
	return 0;
};
```

As visible from the above example, inline conditionals can be quite powerful - they allow decision making based on compile time known information.
The code is valid because whichever condition does not hold true, the block for that conditional will be erase altogether.
Therefore, compiler will not complain about the lack of, say, `getData()` associated function not existing, when an `i32` is passed to the function.

# Conclusion

Hence, we understand how conditionals in Scribe work and what all they can do.

Next, we will understand about loops.
