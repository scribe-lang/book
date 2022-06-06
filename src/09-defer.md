# Defer

A common problem programmers face with C is that since memory allocation is done manually, it must later be deallocated manually as well. We often forget to do that, which results in memory leaks.

Another similar issue stems when there are multiple exists (returns) from a function. If the deallocation must be done at the end of the function, it would either have to be done before all returns,
or there would be multiple gotos to a single tag, instead of multiple returns, where the deallocation will be done and finally function will be exited (returned).

Both of the approaches are quite cumbersome and annoying to work with.

Therefore, Scribe provides a `defer` language keyword which is followed by an expression that must be executed before the function exists (returns).
This `defer` may be placed anywhere in the code and its expression is guaranteed to be called before the function returns (does not include crashes).

As such, a programmer would generally write a `defer` right after the allocation, therefore not having to remember to deallocate later on, and reducing code redundancy when multiple returns are involved.

It also makes reading code easier in the future as one can simply see the allocations and their respective defers just after them to ensure all data is being allocated and deallocated correctly.

For example,

```rs
let c = @import("std/c");

let main = fn(): i32 {
	let arr1 = mem.alloc(i32, 10); // allocate 10x i32's
	let arr2 = mem.alloc(f64, 20); // allocate 20x f64's
	defer mem.free(i32, arr1);
	defer mem.free(f64, arr2);

	if true {
		// both frees would be added here automatically
		return 1;
	}
	// both frees would be added here automatically
	return 0;
};
```

Note that the deferred expressions are added in the **reverse** order of their usage. In the above example, the `mem.free()`'s will be added before the returns such that `arr2` is free'd first.

`defer` is not limited to a function. It actually works on scopes, so something like this is correct too:

```rs
let c = @import("std/c");
let io = @import("std/io");

let main = fn(): i32 {
	for let i = 0; i < 10; ++i {
		let arr = mem.alloc(i32, i + 1);
		defer mem.free(i32, arr);
		let addr = @as(u64, arr);
		io.println("Allocated address: ", addr);
		// mem.free() is called here
	}
	return 0;
};
```

# Conclusion

Defer is an interesting and useful construct which provides a balance between the complexities of C++'s RAII, and C's error prone ways.

Credits to the Zig programming language which uses `defer` as well, from where I so shamelessly copied the concept.

For now, this concludes the main Scribe features. Feel free to check out the standard library documentation which may come in quite handy.
