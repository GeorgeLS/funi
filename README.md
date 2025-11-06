# Overview

funi is a single header library for C++ that unifies unit testing and fuzz testing.

It is completely agnostic of your unit test framework and can be configured using some macro definitions.

## Usage

So, there are three macros that you need to provide when compiling in "unit test mode":

- `FUNI_TEST_DRIVER`: This must be a macro that is going to be substituted in place of `FUNI_TEST` which is the macro that defines your unit test. When compiling for fuzzing mode, you don't need to provide that
- `FUNI_INIT`: This must be a function that accepts `argc` and `argv` that is going to be called as the first function in `main`. You only need that if you require some initialization procedure to run. When compiling for fuzzing mode, you don't need to provide that
- `FUNI_TEST_RUNNER`: This is a function that will run all the unit tests. When compiling for fuzzing mode, you don't neeed to provide that.

If you only want to use the macros for defining unit/fuzzing tests, the only thing you need to do is include `"funi.h"`.
In the file where you want to have the main function, you need to define `FUNI_IMPLEMENTATION` before including the header file.

Finally, if you wanna compile for fuzzing mode, you can pass `-DFUNI_FUZZ` to your compiler.


## API

funi provides a set of macros that can be used to define unit test and fuzzing tests in one compact way.

Here's the complete set of macros:

- `I8`, `U8`, `I16`, `U16`, `I32`, `U32`, `I64`, `U64`, `F32`, `F64`: Generate
