# Hello World!

Here is a simple Hello World program written in Scribe.

```rs
let io = @import("std/io");

let main = fn(): i32 {
	io.println("Hello World");
	return 0;
};
```

That's it!

Let's break it down a bit.

## Importing A Module

For dealing with any form of console I/O, we require the `std/io` module in our program. This module defines all the necessary functions and variables used to perform console I/O.

To import a module, we use the `@import(<module name>)` function and create a variable (`io`) from the result of that function. More about creating variables in the next chapter.

## The Main Function

In any program, there must be a `main` function which is the `entry point` for the execution of the code. In other words, when you run the code, this is where the code logic will start executing from.

To create a function, we create a variable - name of the function, with the function expression as value.

The function expression is the `fn` keyword, followed by its signature `(<arguments>): <return type>`, followed by its body enclosed in braces (`{...}`), terminated by the semicolon (`;`).

This function returns an `i32` - a `32 bit integer` data.

## Displaying Hello World

To display something on the console, the `println(...)` function inside the `io` module is called.
To call a function residing inside a module, we use club the module name and function using the dot (`.`) operator.

Whatever we want displayed via println, we provide it as an argument to the function. For our purposes, we passed the string `"Hello World"`.

## Returning From Function

Since the `main` function returns an `i32`, we must return a value of that type. Here, we return `0` which for command line, basically means that the program exited successfully (exit status).


# Conclusion

That's all! That's how you write a simple Hello World program in Scribe.

To run this program, save it in a file (say, `hello.sc`) and run the compiler on it using the following command: `scribe hello.sc`.

That will generate an executable binary named `hello` in the current directory, which can be run like this: `./hello`.

That will produce the following output:
```
Hello World
```

Next, we will understand how variables in Scribe work.
