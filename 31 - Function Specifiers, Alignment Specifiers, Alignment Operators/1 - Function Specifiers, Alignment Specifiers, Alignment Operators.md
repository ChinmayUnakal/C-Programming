# Function Specifiers

C provides function specifiers that can give the compiler information about how a function is expected to behave or be used. This can allow certain optimizations.

---

# `inline` — Possibly Faster

A function can be declared `inline`:

```c
static inline int add(int x, int y)
{
    return x + y;
}
```

`inline` is intended to encourage the compiler to optimize the function call.

One possible optimization is **function inlining**, where the compiler replaces a call with the function's body instead of performing a normal call/return sequence.

Example:

```c
int result = add(1, 2);
```

Conceptually, the compiler could replace the call with:

```c
int result = 1 + 2;
```

This can avoid function-call overhead, but it may increase the size of the generated code because the function body can be duplicated at multiple call sites.

### Important points

- Modern compilers usually decide themselves when inlining is beneficial.
    
- `inline` is therefore not normally required just for performance.
    
- `static inline` is generally the simpler form because the function has **internal linkage**.
    

---

# `inline` Without `static`

Consider:

```c
inline int add(int x, int y)
{
    return x + y;
}
```

A non-`extern` inline definition has special linkage rules.

Simply defining it this way may result in a linker error because an additional externally linked definition can be required.

Therefore, when using `inline`, using:

```c
static inline
```

avoids many of these linkage complications.

You can also use:

```c
extern inline
```

which allows an inline definition in the current translation unit while also providing external linkage.

When both inline and external definitions exist, the compiler may choose which version to use, so keeping their behavior identical is important.

---

# `noreturn` and `_Noreturn`

`noreturn` tells the compiler that a function is not expected to return to its caller.

Examples include functions that terminate the program:

```c
exit()
abort()
```

A non-returning function can look like:

```c
#include <stdlib.h>
#include <stdnoreturn.h>

noreturn void foo(void)
{
    printf("Happy days\n");
    exit(1);
}
```

The built-in keyword is:

```c
_Noreturn
```

The `<stdnoreturn.h>` header provides the easier-to-read:

```c
noreturn
```

### Why use it?

It can:

- Help the compiler optimize control flow.
    
- Make the function's behavior clear to other programmers.
    

If a function declared `noreturn` actually reaches the end and returns, the behavior is **undefined**.

A compiler may warn when it detects such a possibility.

Example:

```c
noreturn void foo(void)
{
    printf("This is supposed to return");
}
```

may generate a warning because the function can actually reach the end.

---

# Alignment

**Alignment** determines which memory addresses are valid starting locations for objects of a particular type.

For example, an object might require an address aligned to:

```text
1-byte boundary
2-byte boundary
4-byte boundary
8-byte boundary
16-byte boundary
```

For low-level code, alignment matters because incorrect assumptions about object placement can cause problems.

It is especially relevant when dealing with things such as:

- Memory allocators
    
- Hardware interfaces
    
- Low-level system code
    

---

# Alignment Specifiers

C provides an alignment specifier that can be used in a declaration.

The built-in form is:

```c
_Alignas
```

The `<stdalign.h>` header provides:

```c
alignas
```

Example:

```c
char alignas(int) c;
```

This requests that `c` use the alignment required for `int`.

---

# Specifying Alignment Directly

An alignment value can also be given:

```c
char alignas(8) c;
```

This requests an alignment on an 8-byte boundary.

The requested alignment must be supported by the implementation.

Small powers of two such as:

```text
1
2
4
8
16
```

are generally common alignment values, but the C standard does not require every implementation to accept every value.

---

# Maximum Alignment

`<stddef.h>` provides the type:

```c
max_align_t
```

It represents a type whose alignment is suitable for the maximum fundamental alignment requirement supported by the implementation.

Example:

```c
#include <stddef.h>
#include <stdalign.h>

char alignas(max_align_t) c;
```

An alignment greater than that provided by `max_align_t` is an **over-alignment**.

Whether such alignment is supported is implementation-dependent.

---

# Alignment Operator

C provides an operator for querying the alignment requirement of a type.

Built-in form:

```c
_Alignof
```

The `<stdalign.h>` header provides:

```c
alignof
```

Example:

```c
alignof(int)
```

The result tells you the alignment requirement of that type on the current implementation.

---

# Example of `alignof()`

```c
#include <stdalign.h>
#include <stdio.h>
#include <stddef.h>

struct t
{
    int a;
    char b;
    float c;
};

int main(void)
{
    printf("char       : %zu\n", alignof(char));
    printf("short      : %zu\n", alignof(short));
    printf("int        : %zu\n", alignof(int));
    printf("long       : %zu\n", alignof(long));
    printf("long long  : %zu\n", alignof(long long));
    printf("double     : %zu\n", alignof(double));
    printf("long double: %zu\n", alignof(long double));
    printf("struct t   : %zu\n", alignof(struct t));
    printf("max_align_t: %zu\n", alignof(max_align_t));
}
```

Possible output:

```text
char       : 1
short      : 2
int        : 4
long       : 8
long long  : 8
double     : 8
long double: 16
struct t   : 16
max_align_t: 16
```

These values are **implementation-dependent**, so another system may produce different results.

---

# Alignment and Structures

A structure's alignment can be greater than the alignment of any individual member because the compiler may need to insert **padding** to satisfy alignment requirements.

For example:

```c
struct t
{
    int a;
    char b;
    float c;
};
```

The structure can have padding between members or at the end so that objects of `struct t` are correctly aligned.

Therefore:

```text
sizeof(struct t)
```

and:

```text
alignof(struct t)
```

are related but are not the same thing.

---

# `memalignment()` — Querying Pointer Alignment

C23 provides the `memalignment()` function for determining the maximum alignment satisfied by the address represented by a pointer. It is declared in `<stdlib.h>`.

```
#include <stdlib.h>

size_t memalignment(const void *p);
```

The function examines the **address of the object** pointed to by `p` and returns the greatest alignment that the address satisfies.

For example, if a pointer's address is suitably aligned for:

```
1 byte
2 bytes
4 bytes
8 bytes
16 bytes
```

then `memalignment()` can return:

```
16
```

### Comparing with `alignof`

`memalignment()` is useful together with `alignof`.

For example:

```
if (memalignment(p) >= alignof(int))
{
    // p has sufficient alignment for int
}
```

If:

```
memalignment(p) >= alignof(T)
```

then the pointer's alignment is sufficient for accessing an object of type `T`.

### Null Pointer

If `p` is a null pointer:

```
memalignment(NULL)
```

returns:

```
0
```

A result of `0` indicates that the pointer cannot be used to access an object.

### Example

```
#include <stdio.h>
#include <stdlib.h>
#include <stdalign.h>

int main(void)
{
    alignas(32) int x = 10;

    printf("%zu\n", memalignment(&x));
}
```

Possible output:

```
32
```

The exact result depends on the actual address and its alignment.

### `memalignment()` vs `alignof()`

These two answer different questions:

```
alignof(T)
    │
    └── How must an object of type T be aligned?

memalignment(p)
    │
    └── How well aligned is the address contained in p?
```

For example:

```
alignof(int)
```

queries the alignment requirement of the **type** `int`, while:

```
memalignment(p)
```

queries the alignment satisfied by a **particular pointer value**. This makes `memalignment()` a dynamic counterpart to `alignof()`.

### Reverse Engineering Perspective

`memalignment()` is useful when reasoning about actual pointer values and memory addresses.

For example, when analyzing a binary, you may encounter a pointer whose type is unclear. `memalignment()` provides a standard way to determine what alignment that address satisfies, which can help when reasoning about:

- Dynamically allocated objects
- SIMD/vector data
- Structure layouts
- Allocator behavior
- Low-level memory manipulation

`memalignment()` was added in **C23** and is specifically useful because pointer representations are not required to expose alignment through simple integer address calculations on every architecture.

---

# Alignment Mental Model

```text
Type
 │
 └── Alignment requirement
          │
          ▼
Valid starting addresses

Example: alignment = 8

... 0x1000
    0x1008
    0x1010
    0x1018
    ...
```

An object requiring 8-byte alignment should begin at an address satisfying that alignment requirement.

---

# Reverse Engineering Perspective

Alignment is important when reconstructing memory layouts from binaries.

For example, when reverse engineering:

```c
struct Example
{
    uint32_t a;
    uint64_t b;
};
```

you cannot determine the structure layout by simply adding member sizes.

You must also consider:

```text
member alignment
padding
structure alignment
```

This affects:

- Field offsets
    
- Structure size
    
- Memory allocation
    
- Pointer arithmetic
    
- How instructions access structure members
    

Understanding `_Alignas`, `_Alignof`, and `max_align_t` therefore helps when mapping C structures to their actual memory representation.

---

# Long-Term Takeaways

- `inline` can encourage function inlining, but the compiler ultimately decides whether to perform it.
    
- `static inline` avoids many external-linkage complications.
    
- `extern inline` can provide inline behavior together with external linkage.
    
- `noreturn` / `_Noreturn` indicates that a function should never return to its caller.
    
- Returning from a function declared `noreturn` causes undefined behavior.
    
- Alignment determines which addresses can be used as the starting address of an object.
    
- `_Alignas` is the built-in alignment specifier.
    
- `alignas` is the `<stdalign.h>` version.
    
- Alignment can be specified using another type:
    

```c
alignas(int)
```

- A numeric alignment can also be requested:
    

```c
alignas(8)
```

- `max_align_t` represents the implementation's maximum fundamental alignment requirement.
    
- `_Alignof` obtains a type's alignment requirement.
    
- `alignof` is the `<stdalign.h>` spelling.
    
- Alignment values differ between implementations.
    
- Structure padding is closely connected to alignment.
    
- Alignment is especially important in **memory allocators, low-level programming, structure layout, and reverse engineering**.