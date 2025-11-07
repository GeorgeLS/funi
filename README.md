# Overview

funi is a single header library for C++ that unifies unit testing and fuzz testing.

It is completely agnostic of your unit test framework and can be configured using some macro definitions.

## Requirements

So, there are three macros that you need to provide when compiling in "unit test mode":

- `FUNI_TEST_DRIVER`: This must be a macro that is going to be substituted in place of `FUNI_TEST` which is the macro that defines your unit test. When compiling for fuzzing mode, you don't need to provide that
- `FUNI_INIT`: This must be a function that accepts `argc` and `argv` that is going to be called as the first function in `main`. You only need that if you require some initialization procedure to run. When compiling for fuzzing mode, you don't need to provide that
- `FUNI_TEST_RUNNER`: This is a function that will run all the unit tests. When compiling for fuzzing mode, you don't neeed to provide that.

If you only want to use the macros for defining unit/fuzzing tests, the only thing you need to do is include `"funi.h"`.
In the file where you want to have the main function, you need to define `FUNI_IMPLEMENTATION` before including the header file.

Finally, if you wanna compile for fuzzing mode, you can pass `-DFUNI_FUZZ` to your compiler.

## API

funi provides a set of macros that can be used to define unit test and fuzzing tests in one compact way.

The idea is that through the macro, you will be able to provide a value/expression that will be used when running in unit test mode. When compiling for fuzzing mode, this value will be ignored, and some random/fuzz value will be provided instead.

Some of the macros, have support for constraints that will only be enforced when compiling for fuzzing mode and ignored for unit testing.

Here's a complete list of the macros provided:

- `V`: This is a variadic macro that expands to its arguments and stands for `Value`. This was introduced mostly to bypass some macro restrictions and to be able to group multiple values together so that we can distinguish the values from the constraints. Basically this is only needed when used with the `BYTES` macro
- `C`: This is a macro that accepts a single argument and just expands to it. This was just added for _readability_ purposes. Can be completely ommited
- `I8`: Represents a 8-bit signed integer
  - Unit test behavior: The first value will be used
  - Fuzzing mode behavior: First value will be ignored and instead provide a value by calling `FuzzedDataProvider::ConsumeIntegral<int8_t>()`. If two more values are provided, then `FuzzedDataProvider::ConsumeIntegralInRange<int8_t>(min, max)` will be used
- `U8`: Represents a 8-bit unsigned integer
  - Unit test behavior: The first value will be used
  - Fuzzing mode behavior: First value will be ignored and instead provide a value by calling `FuzzedDataProvider::ConsumeIntegral<uint8_t>()`. If two more values are provided, then `FuzzedDataProvider::ConsumeIntegralInRange<uint8_t>(min, max)` will be used
- `I16`: Represents a 16-bit signed integer
  - Unit test behavior: The first value will be used
  - Fuzzing mode behavior: First value will be ignored and instead provide a value by calling `FuzzedDataProvider::ConsumeIntegral<int16_t>()`. If two more values are provided, then `FuzzedDataProvider::ConsumeIntegralInRange<int16_t>(min, max)` will be used
- `U16`: Represents a 16-bit unsigned integer
  - Unit test behavior: The first value will be used
  - Fuzzing mode behavior: First value will be ignored and instead provide a value by calling `FuzzedDataProvider::ConsumeIntegral<uint16_t>()`. If two more values are provided, then `FuzzedDataProvider::ConsumeIntegralInRange<uint16_t>(min, max)` will be used
- `I32`: Represents a 32-bit signed integer
  - Unit test behavior: The first value will be used
  - Fuzzing mode behavior: First value will be ignored and instead provide a value by calling `FuzzedDataProvider::ConsumeIntegral<int32_t>()`. If two more values are provided, then `FuzzedDataProvider::ConsumeIntegralInRange<int32_t>(min, max)` will be used
- `U32`: Represents a 32-bit unsigned integer
  - Unit test behavior: The first value will be used
  - Fuzzing mode behavior: First value will be ignored and instead provide a value by calling `FuzzedDataProvider::ConsumeIntegral<uint32_t>()`. If two more values are provided, then `FuzzedDataProvider::ConsumeIntegralInRange<uint32_t>(min, max)` will be used
- `I64`: Represents a 64-bit signed integer
  - Unit test behavior: The first value will be used
  - Fuzzing mode behavior: First value will be ignored and instead provide a value by calling `FuzzedDataProvider::ConsumeIntegral<int64_t>()`. If two more values are provided, then `FuzzedDataProvider::ConsumeIntegralInRange<int64_t>(min, max)` will be used
- `U64`: Represents a 64-bit unsigned integer
  - Unit test behavior: The first value will be used
  - Fuzzing mode behavior: First value will be ignored and instead provide a value by calling `FuzzedDataProvider::ConsumeIntegral<uint64_t>()`. If two more values are provided, then `FuzzedDataProvider::ConsumeIntegralInRange<uint64_t>(min, max)` will be used
- `F32`: Represents a 32-bit floating-point number
  - Unit test behavior: The first value will be used
  - Fuzzing mode behavior: First value will be ignored and instead provide a value by calling `FuzzedDataProvider::ConsumeFloatingPoint<float>()`. If two more values are provided, then `FuzzedDataProvider::ConsumeFloatingPointInRange<float>(min, max)` will be used
- `F64`: Represents a 64-bit floating-point number
  - Unit test behavior: The first value will be used
  - Fuzzing mode behavior: First value will be ignored and instead provide a value by calling `FuzzedDataProvider::ConsumeFloatingPoint<double>()`. If two more values are provided, then `FuzzedDataProvider::ConsumeFloatingPointInRange<double>(min, max)` will be used
- `STR`: Represents a `std::string`
  - Unit test behavior: The first value will be used
  - Fuzzing mode behavior: First value will be ignored and instead provide a value by calling `FuzzedDataProvider::ConsumeRandomLengthString()`. If one move value is provided, then `FuzzedDataProvider::ConsumeRandomLengthString(max)` will be used
- `BYTES`: Represents a `std::vector<uint8_t>`. You need to provide a comma separated list of values with the `V` macro.
  - Unit test behavior: The values will be used
  - Fuzzing mode behavior: Values will be ignored and instead provide values by calling `FuzzedDataProvider::ConsumeRemainingBytes<uint8_t>()`. If a `C(max_size)` is provided after the values, then `FuzzedDataProvider::ConsumeBytes<uint8_t>(max_size)` will be used
- `BOOL`: Represents a boolean value
  - Unit test behavior: The first value will be used
  - Fuzzing mode behavior: First value will be ignored and instead provide a value by calling `FuzzedDataProvider::ConsumeBool()`

## Example Usage

Here's an example usage of a simple file that defines a unit test and a main function for compiling that to a program. In our example we assume gtest:

```cpp
// Could also pass them as compiler -D flags
#define FUNI_INIT(argc, argv) ::testing::InitGoogleTest(&argc, argv)
#define FUNI_TEST_DRIVER TEST
#define FUNI_TEST_RUNNER(...) RUN_ALL_TESTS()
#define FUNI_IMPLEMENTATION
#include "funi.h"
#ifndef FUNI_FUZZ
#include <gtest/gtest.h>
#endif

static int add(int x, int y) {
    return x + y;
}

FUNI_UNIT(calculator, test_add) {
    // Use 1 as the unit test value or a random value between
    // uint8_t::MIN and uint8_t::MAX when fuzzing
    int x = I8(1);
    // Use 2 as the unit test value or a random value between
    // uint8_t::MIN and uint8_t::MAX when fuzzing
    int y = I8(2);

    int res = add(x, y);
    FUNI_ASSERT_EQ(res, 3);
}

FUNI_MAIN
```

If you compile this file, you will generate a google test executable.

The code above will be expanded to the following:

```cpp
#define FUNI_INIT(argc, argv) ::testing::InitGoogleTest(&argc, argv)
#define FUNI_TEST_DRIVER TEST
#define FUNI_TEST_RUNNER(...) RUN_ALL_TESTS()
#define FUNI_IMPLEMENTATION
#include "funi.h"
#include <gtest/gtest.h>

static int add(int x, int y) {
    return x + y;
}

TEST(calculator, test_add) {
    int x = 1;
    int y = 2;

    int res = add(x, y);
    EXPECT_EQ(res, 3);
}

int main(int argc, char** argv) {
    ::testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```

If you however pass `-DFUNI_FUZZ` the code will be expanded to the following:

```cpp
#define FUNI_IMPLEMENTATION
#include "funi.h"

static int add(int x, int y) {
    return x + y;
}

TEST(calculator, test_add) {
    uint8_t x = __FDP->ConsumeIntegral<uint8_t>();
    uint8_t y = __FDP->ConsumeIntegral<uint8_t>();

    int res = add(x, y);
    FUNI_ASSERT_EQ(res, x + y);
}

int main(int argc, char** argv) {
    return run_funi_tests(argc, argv);
}
```

Another example using the `V` and `C` macro with the unit test is:

```cpp
FUNI_UNIT(calculator, test_add) {
    // Use 1 as the unit test value or a random value between
    // 2 and 5 when fuzzing
    int x = I8(V(1), C(2, 5));
    // Use 2 as the unit test value or a random value between
    // 3 and 6 when fuzzing
    int y = I8(V(2), C(3, 6));

    int res = add(x, y);
    FUNI_ASSERT_EQ(res, x + y);
}
```

# DISCLAIMER

This is still a work in progress. For example, the `FUNI_ASSERT_EQ` macro is not yet implemented.
