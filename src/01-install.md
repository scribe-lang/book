# Scribe Compiler Installation

## Prerequisites

* CMake build system (>= `3.21.1`)
* C++17 standard conforming compiler (GCC, LLVM, etc.)

## Steps

1. Clone the Scribe repository - [scribe-lang/scribe](https://github.com/scribe-lang/scribe.git)
2. Go to the cloned repository and create a `build` directory.
3. Go into the `build` directory and invoke the build commands:
```bash
cmake .. -DCMAKE_BUILD_TYPE=Release && make -j install
```
4. This will install the compiler and its libraries in `$PREFIX_DIR` diectory (default: `$HOME/.scribe`).
5. To run the compiler without specifying the path, add `$PREFIX_DIR/bin` to `$PATH`.
6. That's it! The scribe compiler is now ready for use.

## CMake Environment Variables

### $CXX

This variable is used for specifying the C++ compiler if you do not want to use the ones auto decided by the script, which uses `g++` by default for all operating systems except Android and BSD, for which it uses `clang++`.

For example, to explicitly use `clang++` compiler on an ubuntu (linux) machine, you can use:
```bash
CXX=clang++ cmake .. -DCMAKE_BUILD_TYPE=Release && make install
```

### $PREFIX_DIR

This variable will allow you to set a `PREFIX_DIR` directory for installation of the language after the build.

**NOTE** that once the script is run with a `PREFIX_DIR`, manually moving the generated files to desired directories will not work since Scribe's codebase uses this `PREFIX_DIR` internally itself.

Generally, the `/usr` or `/usr/local` directories are used for setting the `PREFIX_DIR`, however that is totally up to you. Default value for this is `$HOME/.scribe`.

Scribe compiler binary can be found as `$PREFIX_DIR/bin/scribe`.

An example usage is:
```bash
PREFIX_DIR=/usr/local cmake .. -DCMAKE_BUILD_TYPE=Release && make install
```

That concludes the compiler installation! Now you're ready to compile scribe programs, and we can start writing our first program: Hello World!
