# Types

> **Core idea:** A C type determines how a value is interpreted, how much storage it typically requires, and which operations can be performed on it.

---

## Signed and Unsigned Integers

### Signed Integers

A signed integer can represent **negative, zero, and positive** values.

```c
int a;
signed int b;
signed c;        // same as signed int
```

### Unsigned Integers

An unsigned integer represents **zero and positive** values only.

```c
unsigned int a;
unsigned b;      // same as unsigned int
```

### Important Concept

For an integer with **N bits**:

- Unsigned → approximately `0` to `2^N - 1`
    
- Signed → typically approximately `-2^(N-1)` to `2^(N-1)-1`
    

Modern systems commonly use **two's complement** for signed integers.

> **RE relevance:** Signedness matters when interpreting registers and comparing values. The same bit pattern can represent different values depending on whether it is signed or unsigned.

Example:

```text
0xFFFFFFFF

unsigned → 4294967295
signed   → -1
```

---

# Character Types

A `char` occupies **1 byte**.

```c
char c = 'B';

printf("%c\n", c);  // B
printf("%d\n", c);  // commonly 66
```

A character is essentially an integer value representing a character according to the execution character set.

For ASCII:

```text
'A' = 65
'B' = 66
'a' = 97
'0' = 48
```

### Character Variants

```c
char a;           // signedness is implementation-defined
signed char b;    // explicitly signed
unsigned char c;  // explicitly unsigned
```

Example:

```c
char a = 10;
char b = 'B';

printf("%d\n", a + b);  // commonly 76
```

> **Important:** `char` is an integer type, so arithmetic can be performed on it.

> **RE relevance:** Characters often appear in assembly as **bytes**. Strings are simply sequences of character bytes terminated by `'\0'`.

---

# `short`, `long`, and `long long`

C provides integer types with different minimum ranges.

```c
short x;
int x;
long x;
long long x;
```

These can also be written as:

```c
short int x;
long int x;
long long int x;
```

Exact sizes depend on the platform/compiler.

Check them with:

```c
printf("%zu\n", sizeof(short));
printf("%zu\n", sizeof(int));
printf("%zu\n", sizeof(long));
printf("%zu\n", sizeof(long long));
```

### Important

Do **not** assume:

```text
int = 64 bits
```

On many modern systems:

```text
Windows x64:
int       = 32 bits
long      = 32 bits
long long = 64 bits
```

> **RE relevance:** Always determine the target architecture and ABI before assuming type sizes.

---

# Floating-Point Types

C provides:

```c
float
double
long double
```

Example:

```c
float f = 3.14159f;
double d = 3.14159;
```

Typical distinction:

```text
float       → lower precision
double      → higher precision
long double → implementation-dependent
```

### Precision

The number of decimal digits that can be represented accurately depends on the type and implementation.

```c
#include <float.h>

FLT_DIG
DBL_DIG
LDBL_DIG
```

Example:

```c
float f = 3.14159f;
float g = 0.00000265358f;

f += g;
```

The result may not exactly equal the mathematical result because floating-point numbers have **limited precision**.

> **RE relevance:** Floating-point values are represented very differently from integers. On x86-64, floating-point operations commonly use **XMM registers**.

---

# Integer Promotions

Small integer types such as:

```c
char
short
```

are commonly **promoted to `int`** when used in arithmetic expressions.

```c
char a = 10;
char b = 20;

printf("%d\n", a + b);
```

The addition is generally performed as an `int` operation.

> **RE relevance:** The source code may use `char`, but the compiler may perform operations using a **32-bit register** because of integer promotion.

---

# Converting Between Types

C allows values to be converted between different types.

```c
int x = 10;
double y = x;
```

Explicit conversion:

```c
double x = 3.14;
int y = (int)x;
```

This is called a **cast**.

```c
(type) expression
```

Example:

```c
int x = 10;
float y = (float)x;
```

> **RE relevance:** The compiler may insert instructions to convert between integer sizes, signedness, and floating-point formats.

---

# Numeric Bases

C supports several integer representations.

## Decimal

```c
int x = 42;
```

## Hexadecimal

Prefix with `0x` or `0X`:

```c
int x = 0x2A;
```

```text
0x2A = 42
```

Hexadecimal is extremely important in reverse engineering because **memory addresses, machine code, offsets, and bit patterns** are commonly displayed in hex.

## Octal

Prefix with `0`:

```c
int x = 012;
```

```text
012 octal = 10 decimal
```

### Common Beginner Trap

```c
int x = 11111;  // decimal
int y = 00111;  // octal
```

A leading `0` changes the interpretation to octal.

---

# Binary

Some compilers support the `0b` prefix:

```c
int x = 0b101010;
```

```text
101010₂ = 42₁₀
```

Binary is useful when reasoning about:

- Bits
    
- Flags
    
- Masks
    
- Permissions
    
- CPU registers
    
- Bitwise operations
    

> **RE relevance:** Binary and hexadecimal are essential when analyzing registers, instruction operands, memory, and flags.

---

# Integer Constants

Integer constants can include suffixes that specify their type.

```c
1234        // int
1234L       // long
1234LL      // long long

1234U       // unsigned int
1234UL      // unsigned long
1234ULL     // unsigned long long
```

Examples:

```c
unsigned int x = 1234U;
long long y = 1234LL;
```

---

# Floating-Point Constants

```c
3.14       // double
3.14f      // float
3.14L      // long double
```

Without a suffix, a decimal floating-point constant is normally a `double`.

---

# Scientific Notation

Floating-point constants can use `e` or `E`.

```c
double x = 1.23456e5;
```

Meaning:

```text
1.23456 × 10^5
= 123456
```

Examples:

```c
1.2345e10
1.2345e+10
1.2345e10F
1.2345e10L
```

Printing:

```c
printf("%e\n", 123456.0);
```

---

# Hexadecimal Floating-Point Constants

C also supports hexadecimal floating-point notation:

```c
double x = 0xa.1p3;
```

Here:

```text
p3 = × 2³
```

This notation is useful when an **exact binary floating-point representation** matters.

---

# How Types Appear in Reverse Engineering — x86-64

In reverse engineering, **the type usually disappears from the binary**.

The CPU does not know:

```c
int
float
struct
unsigned
```

Instead, it sees:

- Bytes
    
- Addresses
    
- Registers
    
- Instructions
    
- Memory
    
- Control flow
    

The RE analyst must infer the original type from how the compiler uses the data.

---

## 1. Integer Width → Register Width

C:

```c
int x = 10;
```

On x86-64, `int` is commonly 32-bit.

You may see:

```asm
mov eax, 10
```

Where:

```text
EAX = lower 32 bits of RAX
RAX = 64-bit register
```

For a 64-bit integer:

```c
long long x = 10;
```

You may see:

```asm
mov rax, 10
```

### Quick mapping

```text
C type              Common x86-64 representation

char                8-bit
short               16-bit
int                 32-bit
long long           64-bit
pointer             64-bit
```

---

## 2. Signed vs Unsigned

The **same bits** can represent different values.

For example:

```text
0xFFFFFFFF
```

Could mean:

```text
unsigned → 4294967295
signed   → -1
```

The compiler chooses different comparison instructions depending on signedness.

### Signed comparison

Common instructions:

```asm
jg
jge
jl
jle
```

### Unsigned comparison

Common instructions:

```asm
ja
jae
jb
jbe
```

This is extremely important when reversing conditional logic.

---

## 3. `char` → Byte Operations

C:

```c
char c = 'A';
```

May result in operations involving:

```asm
al
```

or memory accesses such as:

```asm
movzx eax, BYTE PTR [rbp-1]
```

`BYTE PTR` indicates an 8-bit memory value.

---

## 4. Integer Promotion in Assembly

C:

```c
char a = 10;
char b = 20;

int x = a + b;
```

The compiler may load the bytes and extend them into a larger register:

```asm
movzx eax, BYTE PTR [...]
movzx edx, BYTE PTR [...]
add  eax, edx
```

`movzx` = **zero extension**.

For signed values, you may see:

```asm
movsx eax, BYTE PTR [...]
```

`movsx` = **sign extension**.

### Remember

```text
movzx → unsigned extension
movsx → signed extension
```

This is a major clue when identifying types during RE.

---

## 5. Integer vs Floating Point

Integer:

```c
int x = 10;
```

typically uses general-purpose registers:

```text
RAX
RBX
RCX
RDX
...
```

Floating-point:

```c
double x = 3.14;
```

commonly uses:

```text
XMM0
XMM1
XMM2
...
```

Example:

```asm
movsd xmm0, QWORD PTR [...]
```

`movsd` commonly indicates a scalar `double` operation.

---

## 6. Hexadecimal Values in RE

You will constantly encounter:

```text
0x401000
0x7fffffffe000
0x20
0x100
0xFFFFFFFF
```

These may represent:

- Memory addresses
    
- Stack offsets
    
- Structure offsets
    
- Constants
    
- Bit masks
    
- Function addresses
    
- Instruction operands
    

Example:

```asm
mov eax, DWORD PTR [rbp-0x10]
```

This can suggest:

```c
int x;
```

stored at a stack offset.

---

## 7. Type Inference

When reversing C, ask:

> **"What type would make these instructions make sense?"**

For example:

```asm
mov eax, DWORD PTR [rbp-0x4]
cmp eax, 10
jg  ...
```

Likely:

```c
int x;
if (x > 10)
```

But:

```asm
mov eax, DWORD PTR [rbp-0x4]
cmp eax, 10
ja  ...
```

suggests an **unsigned comparison**.

---

# RE Type Identification Cheat Sheet

|C concept|Common x86-64 clue|
|---|---|
|`char`|8-bit memory/register|
|`short`|16-bit|
|`int`|32-bit|
|`long long`|64-bit|
|pointer|64-bit address|
|unsigned extension|`movzx`|
|signed extension|`movsx`|
|signed comparison|`jg`, `jl`, etc.|
|unsigned comparison|`ja`, `jb`, etc.|
|`float`|XMM registers, `ss` instructions|
|`double`|XMM registers, `sd` instructions|
|hexadecimal|common representation of addresses/values|
|bit flags|masks, `and`, `or`, `xor`, `test`|

---

## Long-Term Mental Model

```text
C TYPE
  ↓
Determines interpretation + size
  ↓
Compiler generates machine instructions
  ↓
Types mostly disappear
  ↓
RE analyst reconstructs the type
  ↓
Based on:
    ├── operand size
    ├── register used
    ├── sign/zero extension
    ├── comparison instructions
    ├── arithmetic instructions
    ├── memory access
    └── calling convention
```

> **Key RE principle:** **Don't trust variable names or assume types from appearance. Infer the type from how the binary actually uses the data.**