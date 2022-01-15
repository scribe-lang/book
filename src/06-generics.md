# Generics

Generics, also known as **templates**, are special structures and functions which contain one or more variables with placeholder types.
These placeholder types are replaced by actual types when the generic structure/function is created/used.

In modern programming, generics are one of the most useful features as they allow a programmer to write "generic code" which can be used with different types by replacing the placeholders.

Generics are the reason why data structures like `vectors` (dynamically sized arrays), `maps` (dictionaries), etc. are available in standard libraries of languages like C++.
Since they use generic datatypes, they can work with any actual type the programmer requires.

Let's understand generics and their use.

## Generic Structs

These structures contain one or more variables which have a "template"/generic type.

For example,

```rs
let GenA = struct<T> {
	data: T;
	ptr: *T
};
```

These structures cannot be instantiated directly. Instead, they must first be specialized to replace generic types with the actual types, and then initialized like any other structure.

For example,

```rs
let GenA = struct<T> {
	data: T;
	ptr: *T
};

let main = fn(): i32 {
	let a = GenA(i32){5, nil};
	return 0;
};
```

Here, we specialize `GenA` with `i32` as the generic's replacement type, and then instantiate that with `5` and `nil` for `data` and `ptr` respectively.

## Generic Functions

These functions contain one or more of either `any` type, or `type` type as a parameter.

With `any`, as previously described, any argument can be provided.

With `type` however, a specific type is utilized which can also be used elsewhere - say, as return type, variable declaration type, etc.

Combined with generic structures, we can also use a variable with type `type` to specialize a structure.

For example,

```rs
let GenA = struct<T> {
	data: *T;
};

let new = fn(comptime T: type): GenA(T) {
	return GenA(T){nil};
};

let main = fn(): i32 {
	let a = new(f32); // returns an instance of GenA, specialized with f32
	// a.data is of type *f32
	return 0;
};
```

All functions associated to generic structures are also generic functions themselves.
Scribe provides the generic type as a member of the structure instance in these functions, accessible using `self`.

For example,

```rs
let GenA = struct<T> {
	data: *T;
};

let set in GenA = fn(newdata: *self.T): &self { // self can also be used as return type, which will become &GenA(self.T)
	self.data = newdata;
	return self;
};

let main = fn(): i32 {
	let a = GenA(i32){nil};
	let p: *i32 = nil;
	let q: *i32 = nil;
	a.set(p).set(q); // chaining is possible since set() returns &self
	return 0;
};
```

In Scribe, many data structures are made using generic structures and functions like:

* [std/vector](https://github.com/scribe-lang/scribe/blob/main/headers/std/vec.sc),
* [std/map](https://github.com/scribe-lang/scribe/blob/main/headers/std/map.sc), and
* [std/linked_list](https://github.com/scribe-lang/scribe/blob/main/headers/std/linked_list.sc)


Note that the generics are used and eradicated internally by the scribe compiler. That is, the generated C code (and subsequently the executable binary) will have no information of generics whatsoever.

It is also recommended to check out the examples and standard library of Scribe to understand more about generics if you understand the concepts well, as they can be used in a variety of ways and have their own nuances.

# Conclusion

Here, we have learnt about generics and their usage in Scribe. We shall now move on to conditionals.
