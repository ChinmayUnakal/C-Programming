# Pointers to Pointers

A **pointer to a pointer** stores the address of another pointer.

Example:

```c
int x = 3490;

int *p = &x;

int **q = &p;
```

Memory conceptually:

```text
q
│
▼
p
│
▼
x
│
▼
3490
```

---

# Levels of Indirection

```c
int x = 3490;
int *p = &x;
int **q = &p;
```

Then:

```c
x     // 3490
*p    // 3490
**q   // 3490
```

Each `*` removes one level of pointer indirection.

---

# Address-of and Dereference

The `&` operator adds one pointer level.

|Expression|Result Type|
|---|---|
|`x`|`int`|
|`&x`|`int *`|
|`&p`|`int **`|
|`&q`|`int ***`|

The `*` operator does the opposite.

|Expression|Result Type|
|---|---|
|`q`|`int **`|
|`*q`|`int *`|
|`**q`|`int`|

Conceptually:

```text
&
│
▼
Add pointer level

*
│
▼
Remove pointer level
```

---

# Multiple Dereferences

You can use multiple `*` operators.

Example:

```c
int ***p;
```

Then:

```c
p     // int ***
*p    // int **
**p   // int *
***p  // int
```

Each dereference removes one level.

---

# `&` and `*` Relationship

For a valid dereferenceable pointer:

```c
&*p == p
```

Conceptually:

```text
Pointer
  │
  ▼
*p       Dereference
  │
  ▼
&p       Take address again
  │
  ▼
Original pointer
```

Similarly, for an object:

```c
*&x == x
```

These operators can effectively undo each other when the expressions are valid.

---

# Pointer-to-Pointer Example

```c
#include <stdio.h>

int main(void)
{
    int x = 3490;

    int *p = &x;
    int **q = &p;

    printf("%d %d\n", *p, **q);
}
```

Output:

```text
3490 3490
```

Memory:

```text
q ───► p ───► x
             │
             ▼
            3490
```

---

# Pointers to Pointers and `const`

`const` can apply to different levels of a pointer.

Example:

```c
int *const p = &x;
```

`p` is a **const pointer to int**.

You cannot change `p`:

```c
p++;  // Error
```

But, if `x` is not const:

```c
*p = 100;  // Allowed
```

---

# `const` at Multiple Pointer Levels

Example:

```c
int x = 3490;

int *const p = &x;
int *const *q = &p;
```

Here:

```text
q ───► const pointer p ───► int x
```

The type of `q` must correctly preserve the `const` qualification of the pointer it points to.

Another example:

```c
int *const *const q = &p;
```

Now:

- `q` cannot be changed
    
- The pointer `p` reached through `q` is const
    

Always read `const` at the level where it appears.

---

# Multibyte Values

Most objects occupy more than one byte.

Example:

```c
int x = 123;
```

The number of bytes is given by:

```c
sizeof x
```

The address:

```c
&x
```

is the address of the **first byte** of the object.

Conceptually:

```text
Object x

First byte                 Last byte
   │                           │
   ▼                           ▼
[ byte ][ byte ][ byte ][ byte ]
▲
│
&x
```

---

# Object Representation

The bytes making up an object are called its **object representation**.

You can inspect an object's bytes through:

```c
unsigned char *
```

Example:

```c
int x = 0x12345678;

unsigned char *p = (unsigned char *)&x;
```

Then:

```c
p[0]
p[1]
p[2]
...
```

access individual bytes of the object's representation.

---

# Why `unsigned char *`?

C specifically allows an object's representation to be accessed through:

```c
char *
signed char *
unsigned char *
```

This is useful for:

- `memcpy()`
    
- Binary data
    
- Serialization
    
- Memory inspection
    
- Reverse engineering
    

---

# A Simple `memcpy()`

Conceptually:

```c
void *my_memcpy(void *dest, const void *src, size_t n)
{
    const unsigned char *s = src;
    unsigned char *d = dest;

    while (n-- > 0)
        *d++ = *s++;

    return dest;
}
```

It works byte by byte.

Conceptually:

```text
Source Bytes
    │
    ▼
[ A ][ B ][ C ][ D ]
    │
    ▼
Copy each byte
    │
    ▼
[ A ][ B ][ C ][ D ]
Destination Bytes
```

---

# Understanding `*d++ = *s++`

Post-increment has lower priority than dereference in this context.

Conceptually:

```c
*d++ = *s++;
```

means:

```c
*d = *s;

d++;
s++;
```

So one byte is copied, then both pointers advance.

---

# Inspecting a `struct` in Memory

Example:

```c
#include <stdio.h>

struct foo {
    char a;
    int b;
};

int main(void)
{
    struct foo x = {0x12, 0x12345678};

    unsigned char *p = (unsigned char *)&x;

    for (size_t i = 0; i < sizeof x; i++) {
        printf("%02X\n", p[i]);
    }
}
```

This prints every byte in the object's representation.

---

# Struct Padding

Memory might conceptually look like:

```text
[ 12 ][ ?? ][ ?? ][ ?? ][ 78 ][ 56 ][ 34 ][ 12 ]
   │              padding              │
   │                                   │
 x.a                                 x.b
```

The padding bytes exist for alignment.

Their values should not generally be assumed to have meaningful values.

---

# Endianness

Suppose:

```c
int b = 0x12345678;
```

On a little-endian system, memory might contain:

```text
78 56 34 12
```

The least significant byte comes first.

Conceptually:

```text
Value:

0x12 34 56 78

Little-endian memory:

78 56 34 12
```

This is extremely important in:

- Reverse engineering
    
- Networking
    
- Binary exploitation
    
- File formats
    

---

# The Null Pointer and Zero

These are related but different concepts:

```c
NULL
```

```c
0
```

```c
'\0'
```

```c
(void *)0
```

Important:

```text
NULL
    ↓
Null pointer constant / null pointer representation context

0
    ↓
Integer zero

'\0'
    ↓
Character with value zero
```

A null pointer means:

> It does not point to a valid object or function.

Do not confuse:

```c
NULL
```

with:

```c
'\0'
```

The latter is commonly used as the string terminator.

---

# Pointers as Integers

Pointers can be converted to integer types, but arbitrary conversions are implementation-defined and not generally portable.

For portable round-trip conversions, use:

```c
uintptr_t
```

from:

```c
#include <stdint.h>
```

Example:

```c
uintptr_t n = (uintptr_t)p;
```

Later:

```c
int *p2 = (int *)n;
```

`uintptr_t`, when provided by the implementation, can hold converted object-pointer values without losing the information needed to convert back.

There is also:

```c
intptr_t
```

for a signed integer type with the same purpose.

---

# Safe Pointer Conversions

Important permitted conversions include:

### `void *`

```c
int *p = &x;

void *vp = p;

int *p2 = vp;
```

`void *` can hold an object pointer.

---

# Character Pointers

You can convert an object pointer to:

```c
char *
signed char *
unsigned char *
```

This allows access to the object's byte representation.

Example:

```c
unsigned char *bytes = (unsigned char *)&x;
```

---

# Struct and First Member

A pointer to a structure and a pointer to its first member have a special relationship.

Example:

```c
struct foo {
    int x;
    int y;
};

struct foo f;
```

The address of:

```c
&f
```

corresponds to the address where the first member begins:

```c
&f.x
```

They have different pointer types, but C provides guarantees about this relationship when properly converted.

Conceptually:

```text
struct foo
│
├── x  ◄── starts at struct address
└── y
```

---

# Invalid Type Punning and Strict Aliasing

This is dangerous:

```c
int a = 1;

float *p = (float *)&a;
```

Reading:

```c
*p
```

as a `float` violates C's aliasing rules and can produce undefined behavior.

A cast does not magically make an object become another type.

---

# Strict Aliasing

The compiler can assume incompatible pointer types generally do not refer to the same object.

Example conceptually:

```c
int32_t *pv;
struct words *pw;
```

If both actually point to the same memory, the compiler may optimize based on the assumption that they do not alias.

This can produce unexpected results with optimization.

---

# Safe Type Punning

For examining raw bytes, use:

```c
unsigned char *
```

For converting an object's representation safely, `memcpy()` is commonly used.

Example conceptually:

```c
float f;
uint32_t bits;

memcpy(&bits, &f, sizeof bits);
```

This copies the representation without violating strict aliasing.

---

# Pointer Differences

You can subtract two pointers that point into the **same array**.

Example:

```c
int cats[100];

int *f = cats + 20;
int *g = cats + 60;

ptrdiff_t d = g - f;
```

Result:

```text
d = 40
```

The result counts **elements**, not bytes.

---

# `ptrdiff_t`

Pointer differences should be stored in:

```c
ptrdiff_t
```

defined in:

```c
#include <stddef.h>
```

Printing:

```c
printf("%td\n", d);
```

Example output:

```text
40
```

---

# Pointer Difference Mental Model

```text
cats

[0][1][2] ... [20] ................ [60]
                ▲                    ▲
                f                    g

g - f = 40 elements
```

It is not the raw difference in memory addresses.

---

# Pointers to Functions

A **function pointer** stores the address of a function.

Syntax:

```c
float (*p)(int, int);
```

Read it as:

> `p` is a pointer to a function that takes two `int` arguments and returns `float`.

---

# Why Parentheses Matter

This:

```c
float (*p)(int, int);
```

means:

```text
p
│
▼
pointer to function
│
▼
returns float
```

Without parentheses:

```c
float *p(int, int);
```

means something completely different:

> A function returning `float *`.

---

# Calling Through a Function Pointer

Example:

```c
#include <stdio.h>

void print_int(int n)
{
    printf("%d\n", n);
}

int main(void)
{
    void (*p)(int) = print_int;

    p(3490);
}
```

Output:

```text
3490
```

A function name in this context can be used to obtain a pointer to the function.

---

# Functions as Arguments

Function pointers allow functions to receive other functions as arguments.

Example:

```c
int add(int a, int b)
{
    return a + b;
}

int mult(int a, int b)
{
    return a * b;
}
```

A function that accepts an operation:

```c
void print_math(int (*op)(int, int), int x, int y)
{
    int result = op(x, y);

    printf("%d\n", result);
}
```

Usage:

```c
print_math(add, 5, 7);
print_math(mult, 5, 7);
```

Output:

```text
12
35
```

---

# Function Pointer Mental Model

```text
print_math
     │
     │ receives
     ▼
Function Pointer
     │
     ├──► add
     │
     └──► mult
```

Then:

```c
op(x, y);
```

calls whichever function was passed.

---

# Why Function Pointers Matter

Function pointers are used for:

- Callbacks
    
- Event handlers
    
- Sorting functions
    
- Plugin systems
    
- Dispatch tables
    
- Operating systems
    
- Reverse engineering
    

They allow behavior to be selected dynamically.

---

# Reverse Engineering Perspective

A normal function call might appear as:

```asm
call function
```

A function-pointer call is often an **indirect call**.

Conceptually:

```asm
call rax
```

or:

```asm
call qword ptr [address]
```

Meaning:

```text
Get function address
        │
        ▼
Call the address indirectly
```

Recognizing indirect calls is important when analyzing:

- Virtual-function-like systems
    
- Callback tables
    
- Windows APIs
    
- Malware
    
- Dispatch tables
    

---

# Mental Model

```text
Pointer Levels

int x
  ▲
  │
int *p
  ▲
  │
int **q
```

And for function pointers:

```text
Function Pointer
      │
      ▼
Function Address
      │
      ▼
Execute Function
```

---

# Long-Term Takeaways

- A pointer can point to another pointer.
    
- `&` adds one level of pointer indirection.
    
- `*` removes one level through dereferencing.
    
- `&*p == p` when `p` is valid to dereference.
    
- `const` can apply independently at every pointer level.
    
- Objects can occupy multiple bytes, and `&object` gives the address of the first byte.
    
- `unsigned char *` can inspect an object's byte representation.
    
- `memcpy()` fundamentally works by copying bytes.
    
- Structs can contain padding bytes for alignment.
    
- Little-endian systems store the least significant byte first.
    
- `NULL`, `0`, and `'\0'` are related but are not interchangeable concepts.
    
- `uintptr_t` and `intptr_t`, when provided, support pointer-to-integer round trips.
    
- `void *` is the generic pointer type for object pointers.
    
- Character pointers can safely inspect object representations.
    
- Arbitrarily casting incompatible pointer types can violate strict aliasing and cause undefined behavior.
    
- Use `memcpy()` when you need safe representation-level type punning.
    
- Subtracting pointers in the same array produces a difference in **elements**, stored as `ptrdiff_t`.
    
- A function pointer stores the address of a function.
    
- Parentheses are essential in function pointer declarations.
    
- Function pointers allow callbacks and dynamic behavior.
    
- In assembly, function-pointer calls often appear as indirect `call` instructions.