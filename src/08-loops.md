# Loops

Loops are sequences of code that run over and over again, usually, as long as some condition is met. They, like functions, help a lot in reducing repetitive code.

For example, if we want to make a multiplication table of 12 from 1 to 10, would we write 10 statements by hand? It would look something like this:

```rs
let io = @import("std/io");

let main = fn(): i32 {
	io.println("12 x 1 = ", 12 * 1);
	io.println("12 x 2 = ", 12 * 2);
	io.println("12 x 3 = ", 12 * 3);
	io.println("12 x 4 = ", 12 * 4);
	io.println("12 x 5 = ", 12 * 5);
	io.println("12 x 6 = ", 12 * 6);
	io.println("12 x 7 = ", 12 * 7);
	io.println("12 x 8 = ", 12 * 8);
	io.println("12 x 9 = ", 12 * 9);
	io.println("12 x 10 = ", 12 * 10);
	return 0;
};
```

That works, sure. But for sure it isn't convenient! And this is just for tables from 1-10. What about 1-100, or 1-1000? That's 1000 lines of code which is incredibly hard to maintain.

That's where loops come in. The above code, using loops, could simply be written as:

```rs
let io = @import("std/io");

let main = fn(): i32 {
	for let i = 1; i <= 10; ++i {
		io.println("12 x ", i, " = ", 12 * i);
	}
	return 0;
};
```

And if we want to do this for, say, 1000, all we have to do is change the `i <= 10` condition to `i <= 1000`! That's all!

That's a really basic example of loops, but it should paint the picture of their fundamental purpose.

Each execution of the loop block is called loop iteration. As such, the above loop had 10 iterations as the loop block was executed 10 times.

There are special statements that can be used with loops:
* **continue** - This causes a loop's current iteration to be "skipped" from where the `continue` statement is present.
* **break** - This causes a loop to stop executing from where the `break` statement is present.

In Scribe, there are 3 varieties of loops:

* For Loops
  * Simple For Loops
  * Inline For Loops
* While Loops
* For-each Loops

## For Loops

Similar to conditionals, there are two types of for-loops:

* Simple For Loops
* Inline For Loops

### Simple For Loops

These are the most basic and standard loops that exist throughout various programming languages. These loops consist of 4 components:

* **Initialization** - This component is used to initialize the loop, often by creating a variable. Like in the above example, we initialized `i` with value `1`. This component is executed just once, right before the loop starts.
* **Condition** - This defines how many times the loop must be run. Before each loop iteration, this condition is checked. If the condition is true, the iteration occurs, otherwise the loop is stopped.
* **Increment/Decrement** - This This component allows for modification in some value - often to update the variable that is being looped through (above, `++i`) after each execution of the loop block.
* **Loop Block** - This is the body of the loop. Whatever we want executed multiple times comes here.

Aside from loop block, all the other components are optional and we can also skip them all should we want to, which will make for an infinite loop - a loop that never ends.

The loop block can also be empty if there is nothing to be done inside the loop.

Needless to say, the loop we wrote for generating table of 12 was a simple for loop.

For example,

```rs
let io = @import("std/io");

let main = fn(): i32 {
	let i = 0;
	for ; ; {
		if i >= 10 { break; }
		if i == 5 { continue; }
		io.println(i);
		++i;
	}
	return 0;
};
```

The above loop will print all numbers from 0-9 (inclusive), except 5.

### Inline For Loops

Similar to inline conditionals, Scribe supports inline for loops which are executed at compile time. It requires that all the components of the loop, except the loop block, must be evaluable at compile time.

Note that the loop block will not executed like it is during runtime, but instead, the block will be repeated as many times as the loop condition holds true.
Therefore it is unwise to make large inline for loops as they will increase code size, increase compilation time, and most likely reduce code performance as well.
An infinite inline for loop will cause the compiler to never finish compiling (and if not stopped, cause the system to run out of memory).

Functions like `io.println()` are able to exist in Scribe userland (without special compiler magic) code because of variadic parameters, inline for loops, and inline conditionals.

As we have seen in the Functions chapter of this manual, inline for loops can be used to iterate over a variadic parameter to work with each of the provided argument.

For example, a `println()` function can be written as:

```rs
let c = @import("std/c"); // imports functions from C
let string = @import("std/string"); // contains string.String and associated functions for integer to string (<int>.str())

let println = fn(data: ...&const any): i32 { // takes variadic number of arguments each of type &const any (any const reference), and returns an i32
	let comptime len = @valen();
	let sum = 0;
	inline for let comptime i = 0; i < len; ++i {
		inline if @isCString(data[i]) {
			sum += c.fputs(data[i], c.stdout);
		} elif @isCChar(data[i]) {
			sum += c.fputc(data[i], c.stdout);
		} elif @isEqualTy(data[i], string.String) {
			sum += c.fputs(data[i].cStr(), c.stdout);
		} else {
			let s = data[i].str();
			defer s.deinit();
			sum += c.fputs(s.cStr(), c.stdout);
		}
	}
	sum += c.fputc('\n', c.stdout);
	return sum;
};

let main = fn(): i32 {
	println(); // prints a newline ('\n') character
	println(5); // none of inline if's are true, therefore else is used
	println("Hello ", 1); // first argument is a cstring, second is an integer
	return 0;
};
```

In the function, a compile time iteration (using inline for) is done over each of the variadic parameter's provided argument.
Based on the type of that argument, we determine which function to call in order to display the data.

Note that the iteration variable (here, `i`) **must** be created as `comptime` or the results may be undefined.

## While Loops

Unlike for loops which consist of 4 components, while loops consist of just 2 - loop condition and loop body.
In a sense, these can be considered a subset of for loops, simply providing a cleaner approach for creating a loop when only the condition is involved.

For example,

```rs
let io = @import("std/io");

let main = fn(): i32 {
	let i = 0;
	while i < 10 {
		if i == 5 { continue; }
		io.println(i++);
	}
	return 0;
};
```

There is no `inline` variant for while loops.

## For-each Loops

These loops are generally used to list over items in a data structure. At the end of the day, they are syntactic sugar which are transformed by the compiler into simple for loops.

For example,

```rs
let io = @import("std/io");
let vec = @import("std/vec");

let main = fn(): i32 {
	let v = vec.new(i32, true);
	defer v.deinit(); // we'll come to defer in the next chapter
	for let i = 0; i < 10; ++i {
		v.push(i);
	}
	for e in v.each() { // iterates over each element, the reference of which is in variable 'e'
		io.println(e);
	}
	for e in v.eachRev() { // iterates over each element, in reverse
		io.println(e);
	}
	for e in v.each() {
		e = e + 1; // since e is a reference to an element in the vector, it can be modified
	}
	io.println(v);
	return 0;
};
```

There are some requirements that must be met in order to use for-each loops on your own structure (type):

* The iterator generation function (here, `each()`, and `eachRev()`) must return an iteration structure (here `iter.Iter`, see [std/iter](https://github.com/scribe-lang/scribe/blob/main/headers/std/iter.sc)) that contains the following functions:

  * `begin(): E`
  * `end(): E`
  * `next(E): E`
  * `at(E): &any`

  Where, `E` is the iteration counter, and `any` is the data (usually reference) that should be stored in the for each iteration variable (`e`).

* The structure (type) must contain a function with signature as `at(E): D`, where `E` is the iteration counter (index for a vector), and `D` is the returned data against the iteration counter (element reference for a vector).

Once these conditions are met, any custom datatype (structure) can be used with a for-each loop.

As with `while` loops, there is no `inline` variant for for-each loops.

# Conclusion

Here we learn about the various types of loops in Scribe, and their usage.
Next up, we will understand an interesting Scribe feature - `defer`.
