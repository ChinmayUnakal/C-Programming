
## What is Pointer Arithmetic?

Pointer arithmetic allows you to move through objects in memory.

For a pointer of type `T *`:

```text
p + 1
```

moves by:

```text
sizeof(T)
```

bytes, **not necessarily 1 byte**.

---

# Adding to Pointers

Example:

```c
int a[5] = {11, 22, 33, 44, 55};

int *p = a;
```

`p` points to `a[0]`.

```c
*p       // 11
*(p + 1) // 22
*(p + 2) // 33
```

If `int` is 4 bytes:

```text
p       → address + 0
p + 1   → address + 4
p + 2   → address + 8
```

The compiler automatically scales the pointer by the size of its pointed-to type.

---

# Pointer Arithmetic and Arrays

These are equivalent:

```c
a[i]
```

```c
*(a + i)
```

Example:

```c
int a[] = {11, 22, 33};

printf("%d\n", a[1]);
printf("%d\n", *(a + 1));
```

Both print:

```text
22
```

---

# Changing a Pointer

A pointer itself can be moved.

```c
int a[] = {11, 22, 33, 44};

int *p = a;

p++;    // now points to a[1]
p++;    // now points to a[2]
```

Example:

```c
while (*p != 44)
{
    printf("%d\n", *p);
    p++;
}
```

`p++` moves the pointer to the **next element**, not necessarily the next byte.

---

# Pointer Increment

For:

```c
int *p;
```

```c
p++;
```

means:

```text
p = p + sizeof(int)
```

For:

```c
char *p;
```

```c
p++;
```

moves by one byte because:

```text
sizeof(char) == 1
```

---

# Pointer Subtraction

Two pointers to elements of the same array can be subtracted.

Example:

```c
int a[] = {10, 20, 30, 40, 50};

int *p = &a[4];
int *q = &a[1];

printf("%td\n", p - q);
```

Result:

```text
3
```

The result represents the **number of elements between the pointers**, not the number of bytes.

The result type is `ptrdiff_t`.

---

# Example: `strlen()` Using Pointer Arithmetic

```c
size_t my_strlen(char *s)
{
    char *p = s;

    while (*p != '\0')
        p++;

    return (size_t)(p - s);
}
```

If:

```text
s → H e l l o \0
```

then `p` stops at `'\0'`.

```text
p - s = number of characters
```

---

# Pointer Comparison

Pointers can be compared.

Common comparisons:

```c
p == q
p != q
p < q
p > q
p <= q
p >= q
```

Ordering comparisons such as `<` and `>` are meaningful when the pointers refer to elements of the same array (or one past it).

---

# Array / Pointer Equivalence

For an array:

```c
int a[] = {11, 22, 33, 44, 55};
```

These all access the same elements:

```c
a[i]
```

```c
*(a + i)
```

If:

```c
int *p = a;
```

then:

```c
p[i]
```

and:

```c
*(p + i)
```

also access the same elements.

Therefore:

```text
a[i] == *(a + i)
     == p[i]
     == *(p + i)
```

---

# Array Name vs Pointer

An array name usually **decays to a pointer to its first element**.

```c
int a[5];

a
```

usually becomes:

```c
&a[0]
```

But the array itself is **not a pointer**.

Important difference:

```c
p++;    // valid if p is a pointer
a++;    // invalid: array name cannot be incremented
```

---

# Pointer Arithmetic in Assembly

C:

```c
x = a[i];
```

For an `int` array, assembly may look like:

```asm
mov eax, [rbx + rcx*4]
```

Conceptually:

```text
Address = Base + Index × Element Size
```

For:

```text
int → 4 bytes
```

the scale is `4`.

For:

```text
double → 8 bytes
```

the scale is `8`.

This is a **very important reverse-engineering pattern**.

---

# Array Arguments to Functions

These function declarations are equivalent:

```c
int my_strlen(char *s);
```

```c
int my_strlen(char s[]);
```

When used as a function parameter, an array parameter is adjusted to a pointer.

Therefore:

```c
char s[] = "Hello";
char *p = s;

my_strlen(s);
my_strlen(p);
```

Both work.

---

# `void *` — Generic Pointer

A `void *` is a pointer to an object of **unknown or unspecified type**.

Example:

```c
int x = 10;

void *p = &x;
```

`p` contains the address of `x`, but its type does not specify what `x` is.

---

# Why Use `void *`?

`void *` allows functions to work with different data types.

Common examples:

```c
memcpy()
qsort()
bsearch()
```

Instead of writing:

```text
memcpy_int()
memcpy_float()
memcpy_double()
memcpy_struct()
...
```

one generic function can operate on raw memory.

---

# `void *` and Dereferencing

You cannot directly dereference a `void *` in standard C because the compiler does not know the pointed-to type.

```c
void *p = &x;

*p;    // Invalid
```

Convert it to the correct pointer type first:

```c
int *q = p;

printf("%d\n", *q);
```

---

# `void *` and Pointer Arithmetic

Standard C does not define pointer arithmetic on `void *`.

This is invalid:

```c
void *p;

p++;
```

The compiler does not know the size of the object.

Convert it to a character pointer when working byte-by-byte:

```c
char *p = data;

p++;
```

Because:

```text
sizeof(char) == 1
```

---

# `void *` and Other Operations

Because the pointed-to type is unknown, you cannot directly use:

```c
*p
p->member
p[i]
p++
```

with a `void *` in standard C.

First convert it to an appropriate pointer type.

---

# `memcpy()`

Prototype:

```c
void *memcpy(void *dest, const void *src, size_t n);
```

It copies exactly `n` **bytes** from `src` to `dest`.

Example:

```c
int a[] = {11, 22, 33};
int b[3];

memcpy(b, a, sizeof(a));
```

Now:

```text
b = {11, 22, 33}
```

For an array:

```c
sizeof(a)
```

gives the total number of bytes.

---

# Copying Strings with `memcpy()`

```c
char s[] = "Goats!";
char t[100];

memcpy(t, s, sizeof(s));
```

`sizeof(s)` includes the terminating `'\0'`.

For `"Goats!"`:

```text
6 characters + 1 null = 7 bytes
```

For strings, however, `strcpy()` is usually more appropriate.

---

# Copying Structs with `memcpy()`

Example:

```c
struct Animal a;
struct Animal b;

memcpy(&b, &a, sizeof(a));
```

This copies the struct's raw bytes.

For ordinary structs, direct assignment is usually clearer:

```c
b = a;
```

---

# Implementing `memcpy()` Conceptually

A simplified byte-copying implementation:

```c
void *my_memcpy(void *dest, void *src, size_t n)
{
    char *d = dest;
    char *s = src;

    while (n--)
        *d++ = *s++;

    return dest;
}
```

Why `char *`?

Because we want to move through memory **one byte at a time**.

---

# `qsort()` and `void *`

`qsort()` can sort arrays of different types.

Conceptually:

```c
qsort(array, count, element_size, comparison_function);
```

Example:

```c
qsort(a, 4, sizeof(struct animal), compar);
```

The comparison function receives generic pointers:

```c
int compar(const void *a, const void *b)
```

Inside the function, convert them to the correct type:

```c
const struct animal *animal1 = a;
const struct animal *animal2 = b;
```

Now the members can be accessed:

```c
animal1->leg_count
animal2->leg_count
```

---

# Generic Memory Model

`void *` allows a function to think in terms of:

```text
Address
   ↓
Raw Bytes
   ↓
Interpret as required type
```

The generic function does not need to know the type itself.

The caller provides enough information for the operation to work.

---

# Important `void *` Limitation

A `void *` tells the compiler:

> "There is an object here, but I am not telling you its type."

Therefore, before doing type-dependent operations, convert it:

```text
void *
  │
  ▼
Correctly Typed Pointer
  │
  ├── Dereference
  ├── Pointer arithmetic
  ├── Array indexing
  └── Struct member access
```

---

# Reverse Engineering Perspective

Pointer arithmetic is extremely important when reconstructing data structures.

If you see:

```asm
mov eax, [rcx + 0x10]
```

you may be looking at:

```text
object + 0x10
      ↓
    member
```

If you see:

```asm
mov eax, [rcx + rdx*4]
```

you may be looking at:

```text
array_base + index × 4
```

which strongly suggests an array of 4-byte elements.

---

# Mental Model

For typed pointers:

```text
p + 1
   │
   ▼
Move by sizeof(*p)
```

For arrays:

```text
a[i]
   │
   ▼
*(a + i)
   │
   ▼
Base Address + i × Element Size
```

For generic memory:

```text
void *
  │
  ▼
Unknown Object
  │
  ▼
Cast to Correct Type
```

---

# Long-Term Takeaways

- Pointer arithmetic is **scaled by the pointed-to type's size**.
    
- `p + 1` means "next element", not "next byte".
    
- `p++` moves to the next element.
    
- Pointer subtraction gives the **number of elements** between two pointers.
    
- `a[i] == *(a + i)`.
    
- An array name usually decays to a pointer to its first element.
    
- An array is **not** itself a pointer.
    
- Array parameters are adjusted to pointer parameters.
    
- `void *` is a generic object pointer.
    
- You cannot directly dereference or perform standard pointer arithmetic on `void *`.
    
- `memcpy()` operates on raw bytes and therefore uses `void *`.
    
- `qsort()` uses `void *` and callbacks to operate on arbitrary data types.
    
- In reverse engineering, pointer arithmetic often reveals **arrays, structures, buffers, and object layouts**.
    
- A very important assembly pattern is:
    

```text
Base + Index × Element Size
```