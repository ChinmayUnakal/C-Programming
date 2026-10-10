# Fixed-Width Integer Types

C normally provides integer types such as:

```c
char
short
int
long
long long
```

However, their exact sizes are not completely fixed by the C standard.

For example, `int` is guaranteed to be **at least 16 bits**, but it is not required to always be exactly 32 bits.

When a program needs an integer with an exact number of bits, C provides **fixed-width integer types** through:

```c
#include <stdint.h>
```

These types are useful when the exact integer size matters, especially when working with:

- Binary data
    
- Network protocols
    
- File formats
    
- Hardware registers
    
- Cryptography
    
- Reverse engineering
    

---

# The Bit-Sized Types

The `<stdint.h>` header provides integer types with an exact number of bits.

The commonly used types are:

```c
int8_t
int16_t
int32_t
int64_t
```

These are **signed** integer types.

For unsigned integers:

```c
uint8_t
uint16_t
uint32_t
uint64_t
```

Example:

```c
uint32_t value;
```

This specifies that `value` is an unsigned integer with exactly **32 bits**, provided the implementation supports that type.

---

# Signed vs Unsigned Fixed-Width Types

Signed types can represent both positive and negative values.

For example:

```c
int32_t x;
```

Unsigned types represent zero and positive values:

```c
uint32_t x;
```

For an `N`-bit signed integer, the usual range is:

```text
-2^(N-1)  to  2^(N-1)-1
```

For an `N`-bit unsigned integer:

```text
0  to  2^N-1
```

For example, a 32-bit unsigned integer has a range of:

```text
0 to 4,294,967,295
```

while a 32-bit signed integer has a range of:

```text
-2,147,483,648 to 2,147,483,647
```

---

# Exact-Width Types

The main exact-width types are:

|Type|Width|Signed?|
|---|--:|---|
|`int8_t`|8 bits|Yes|
|`uint8_t`|8 bits|No|
|`int16_t`|16 bits|Yes|
|`uint16_t`|16 bits|No|
|`int32_t`|32 bits|Yes|
|`uint32_t`|32 bits|No|
|`int64_t`|64 bits|Yes|
|`uint64_t`|64 bits|No|

These types are especially useful when the program needs a predictable integer width.

---

# Maximum Integer Size Type

C also provides integer types designed to be capable of representing values from any signed or unsigned integer type.

These are:

```c
intmax_t
uintmax_t
```

They are provided by:

```c
#include <stdint.h>
```

Example:

```c
intmax_t value;
uintmax_t unsigned_value;
```

`intmax_t` is a signed integer type capable of representing values of any signed integer type.

`uintmax_t` provides the corresponding capability for unsigned integer types.

These types are useful when code needs to handle the **largest integer representation supported by the implementation** without knowing the original integer type.

---

# Using Fixed-Size Constants

`<stdint.h>` also provides macros for creating integer constants with a specific type.

For example:

```c
INT32_C(100)
```

creates an integer constant suitable for use with `int32_t`.

For unsigned constants:

```c
UINT32_C(100)
```

Example:

```c
int32_t x = INT32_C(100);
uint32_t y = UINT32_C(100);
```

Other examples include:

```c
INT64_C(123456789);
UINT64_C(123456789);
```

This is useful when the type of an integer constant needs to match the intended fixed-width integer type.

---

# Why Use Integer Constant Macros?

Consider a value such as:

```c
123456789
```

The type selected for an integer literal depends on the rules of the C language and the implementation.

When working with fixed-width types, using:

```c
INT64_C(123456789)
```

makes the intended integer type explicit.

This becomes particularly useful when writing portable code that must work across different platforms.

---

# Limits of Fixed-Width Integers

Fixed-width integers have a limited range.

For example:

```c
uint8_t x;
```

can represent:

```text
0 to 255
```

It cannot represent:

```text
256
```

If an unsigned integer exceeds its representable range, its value wraps according to unsigned arithmetic rules.

Example:

```c
uint8_t x = 255;

x++;
```

The resulting value is:

```text
0
```

Conceptually:

```text
255 + 1
   ↓
256
   ↓
8-bit representation
   ↓
0
```

---

# Signed Integer Limits

Signed integer overflow is different.

For example:

```c
int8_t x = 127;

x++;
```

The mathematical result would be:

```text
128
```

but `128` cannot be represented by an 8-bit signed integer.

Signed integer overflow causes **undefined behavior**.

This distinction is important:

```text
Unsigned overflow → defined modulo behavior

Signed overflow   → undefined behavior
```

---

# Integer Limits

The `<stdint.h>` header provides macros that describe the limits of the fixed-width types.

Examples:

```c
INT8_MIN
INT8_MAX

UINT8_MAX

INT16_MIN
INT16_MAX

UINT16_MAX

INT32_MIN
INT32_MAX

UINT32_MAX

INT64_MIN
INT64_MAX

UINT64_MAX
```

Example:

```c
printf("%d\n", INT32_MAX);
```

This allows code to use the implementation's defined limits rather than manually writing the numbers.

---

# Optional Fixed-Width Types

An important detail is that not every implementation is required to provide every exact-width type.

For example:

```c
int64_t
```

is available only if the implementation has a signed integer type that is **exactly 64 bits wide** and satisfies the required representation constraints.

Therefore, portable code should not blindly assume that every `intN_t` type exists on every possible C implementation.

The `<stdint.h>` header only defines an exact-width type when the implementation can provide it.

---

# Format Specifiers

When printing fixed-width integer types with `printf()`, the correct format specifier is important.

Instead of assuming that:

```c
uint32_t
```

is always equivalent to `unsigned int`, C provides format macros through:

```c
#include <inttypes.h>
```

For example:

```c
printf("%" PRIu32 "\n", value);
```

This is the portable way to print a `uint32_t`.

Similarly:

```c
printf("%" PRId32 "\n", value);
```

prints an `int32_t`.

---

# Common `inttypes.h` Format Macros

Some commonly used macros are:

|Type|`printf()` macro|
|---|---|
|`int8_t`|`PRId8`|
|`uint8_t`|`PRIu8`|
|`int16_t`|`PRId16`|
|`uint16_t`|`PRIu16`|
|`int32_t`|`PRId32`|
|`uint32_t`|`PRIu32`|
|`int64_t`|`PRId64`|
|`uint64_t`|`PRIu64`|

Example:

```c
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

int main(void)
{
    uint32_t value = 4000000000U;

    printf("Value: %" PRIu32 "\n", value);
}
```

The unusual-looking syntax:

```c
"%" PRIu32
```

works because `PRIu32` expands to the appropriate format string for the implementation.

---

# `intmax_t` Format Specifiers

`inttypes.h` also provides format macros for the maximum-width integer types.

For signed:

```c
PRIdMAX
```

For unsigned:

```c
PRIuMAX
```

Example:

```c
intmax_t value = 123456;

printf("%" PRIdMAX "\n", value);
```

---

# Reverse Engineering Perspective

Fixed-width integers are extremely important when working with binaries.

A binary does not think in terms of:

```text
"int"
"long"
"short"
```

in the abstract C sense.

At the machine level, you frequently encounter values with specific widths:

```text
8-bit
16-bit
32-bit
64-bit
```

For example:

```c
uint32_t flags;
uint64_t address;
uint16_t port;
uint8_t byte;
```

can make the intended data width much clearer when reconstructing structures from a binary.

For example:

```c
struct Header
{
    uint32_t magic;
    uint16_t version;
    uint16_t flags;
    uint64_t size;
};
```

This is much more explicit about the expected layout than using implementation-dependent types such as:

```c
struct Header
{
    unsigned int magic;
    unsigned short version;
    unsigned short flags;
    unsigned long long size;
};
```

When analyzing binary structures, knowing the exact width of each field helps determine:

- Structure size
    
- Field offsets
    
- Alignment
    
- Memory layout
    
- How instructions access the fields
    

---

# Mental Model

```text
             <stdint.h>
                  │
                  ▼
       Fixed-width integer types
                  │
        ┌─────────┴─────────┐
        │                   │
      Signed             Unsigned
        │                   │
   ┌────┼────┐         ┌────┼────┐
   ▼    ▼    ▼         ▼    ▼    ▼
 int8 int32 int64     uint8 uint32 uint64
```

For printing:

```text
Fixed-width type
       │
       ▼
   <inttypes.h>
       │
       ▼
  Format macro
       │
       ▼
    printf()
```

---

# Long-Term Takeaways

- `<stdint.h>` provides integer types with specified widths.
    
- Common signed types are:
    

```c
int8_t
int16_t
int32_t
int64_t
```

- Common unsigned types are:
    

```c
uint8_t
uint16_t
uint32_t
uint64_t
```

- `intmax_t` represents the widest signed integer type available.
    
- `uintmax_t` represents the widest unsigned integer type available.
    
- `INT32_C()` and `UINT32_C()` can create appropriately typed integer constants.
    
- Fixed-width integers still have finite ranges.
    
- Unsigned overflow has defined modulo behavior.
    
- Signed integer overflow results in undefined behavior.
    
- `<stdint.h>` provides macros such as `INT32_MAX` and `UINT64_MAX` for integer limits.
    
- Exact-width types are optional when the implementation cannot provide the required width.
    
- `<inttypes.h>` provides portable `printf()` format macros such as `PRId32` and `PRIu64`.
    
- Fixed-width types are particularly useful for **binary formats, protocols, structures, hardware interfaces, and reverse engineering**.
    
- In RE, explicitly sized types make it easier to reason about **memory layout, field offsets, and instruction-level operations**.