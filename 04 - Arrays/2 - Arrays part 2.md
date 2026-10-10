# Arrays — Miscellaneous Features

This section covers three additional array features:

- Type qualifiers in array parameters
    
- `static` in array parameters
    
- Partial/multidimensional array initializers
    

---

# Type Qualifiers for Array Parameters

Recall that these function parameters are equivalent:

```c
int func(int *p)
{
    // ...
}
```

```c
int func(int p[])
{
    // ...
}
```

In a function parameter list, an array parameter is adjusted to a **pointer parameter**.

So:

```c
int p[]
```

is essentially:

```c
int *p
```

### Adding Type Qualifiers

Pointer qualifiers such as:

```c
int *const p;
int *volatile p;
int *const volatile p;
```

can also be written using array parameter notation.

The qualifiers go **inside the brackets**:

```c
int func(int p[const])
{
    // ...
}
```

is equivalent to:

```c
int func(int *const p)
{
    // ...
}
```

Similarly:

```c
int func(int p[const volatile])
{
    // ...
}
```

is equivalent to:

```c
int func(int *const volatile p)
{
    // ...
}
```

You can also include an optional size:

```c
int func(int p[const volatile 10])
{
    // ...
}
```

The `10` does **not** make the parameter an actual array of 10 elements; it remains pointer-based.

### Important

For multidimensional arrays, the qualifiers belong in the **first set of brackets**.

---

# `static` in Array Parameters

A special form of array parameter is:

```c
int func(int p[static 4])
{
    // ...
}
```

This means:

> The caller promises that `p` points to the first element of an array containing **at least 4 elements**.

For example:

```c
int func(int p[static 4])
{
    return p[3];
}
```

Valid calls:

```c
int a[] = {11, 22, 33, 44};
int b[] = {11, 22, 33, 44, 55};

func(a);    // OK
func(b);    // OK
```

But:

```c
int c[] = {11, 22};

func(c);    // Undefined behavior
```

because `c` doesn't satisfy the minimum requirement.

---

# What Does `static` Actually Do?

It does **not**:

- allocate an array
    
- make the array static
    
- enforce the size at runtime
    

Instead, it is essentially a **minimum-size contract between the programmer and compiler**.

```c
int func(int p[static 4])
```

means:

```text
Caller:
"I promise p points to at least 4 elements."

Compiler:
"Okay. I can assume that."
```

The compiler may use this information for optimization.

### Important

The compiler generally doesn't need to insert a runtime check that verifies the array has 4 elements.

Breaking the promise results in **undefined behavior**.

---

# `static` vs Normal Array Parameter

```c
int func(int p[4])
```

does **not** mean the caller must provide exactly 4 elements.

Remember:

```c
int p[4]
```

in a parameter list is adjusted to a pointer.

But:

```c
int func(int p[static 4])
```

adds a minimum-size requirement:

```text
p[4]        → pointer parameter; 4 is not a runtime size requirement

p[static 4] → pointer parameter + caller promises ≥ 4 elements
```

This distinction is important.

---

# RE Perspective — `static` Array Parameters

Consider:

```c
int func(int p[static 4])
{
    return p[3];
}
```

The compiler knows that accessing:

```c
p[3]
```

is expected to be valid.

When reverse engineering optimized binaries, this kind of source-level information may influence:

- generated memory accesses
    
- bounds assumptions
    
- optimization
    
- removal of checks
    

You won't necessarily see anything corresponding directly to the keyword `static`.

---

# Partial Array Initializers

C allows you to provide only part of an array's initializer.

For example:

```c
int a[5] = {1, 2};
```

produces:

```text
1  2  0  0  0
```

Any elements not explicitly initialized are initialized to **zero**.

### Zero Initialization

A common idiom is:

```c
int a[5] = {0};
```

which initializes the entire array to zero:

```text
0  0  0  0  0
```

---

# Multidimensional Array Initializers

Consider:

```c
int a[3][2];
```

This represents:

```text
3 rows × 2 columns
```

A complete initializer is:

```c
int a[3][2] = {
    {1, 2},
    {3, 4},
    {5, 6}
};
```

Memory conceptually contains:

```text
1  2
3  4
5  6
```

---

# Partial Row Initializers

You can leave elements out of an individual row:

```c
int a[3][2] = {
    {1, 2},
    {3},
    {5, 6}
};
```

The missing element becomes zero:

```text
1  2
3  0
5  6
```

So:

```c
{3}
```

is effectively:

```c
{3, 0}
```

---

# Omitting an Entire Row

Consider:

```c
int a[3][2] = {
    {1, 2},
    {5, 6}
};
```

The result is:

```text
1  2
5  6
0  0
```

C fills the provided initializers **in order**.

It doesn't know that you intended `{5, 6}` to be the third row.

It simply uses it for the next available elements.

---

# Flat Initializers for Multidimensional Arrays

C also allows:

```c
int a[3][2] = {1, 2, 3, 4, 5, 6};
```

Even though the array is two-dimensional, the initializer can be written as a flat list.

The values are assigned in row-major order:

```text
1  2
3  4
5  6
```

Conceptually:

```text
a[0][0] = 1
a[0][1] = 2

a[1][0] = 3
a[1][1] = 4

a[2][0] = 5
a[2][1] = 6
```

---

# Partial Flat Initializers

You can also provide fewer values:

```c
int a[3][2] = {1, 2, 3};
```

Result:

```text
1  2
3  0
0  0
```

The remaining elements are zero-initialized.

---

# Initializing Everything to Zero

For multidimensional arrays:

```c
int a[3][2] = {0};
```

produces:

```text
0  0
0  0
0  0
```

This is a very common C idiom.

---

# Recommended Style

Although this is legal:

```c
int a[3][2] = {1, 2, 3, 4, 5, 6};
```

prefer:

```c
int a[3][2] = {
    {1, 2},
    {3, 4},
    {5, 6}
};
```

for multidimensional arrays.

It makes the structure of the data obvious.

### Exception

For zero-initialization, this is idiomatic:

```c
int a[3][2] = {0};
```

---

# Memory / x86-64 Perspective

A multidimensional C array is stored **contiguously**.

For:

```c
int a[3][2] = {
    {1, 2},
    {3, 4},
    {5, 6}
};
```

the memory layout is essentially:

```text
a[0][0]  → 1
a[0][1]  → 2
a[1][0]  → 3
a[1][1]  → 4
a[2][0]  → 5
a[2][1]  → 6
```

Assuming `int` is 4 bytes:

```text
+------+------+------+------+------+------+
|  1   |  2   |  3   |  4   |  5   |  6   |
+------+------+------+------+------+------+
   4B     4B     4B     4B     4B     4B
```

This is important in reverse engineering because a 2D array often appears in assembly simply as **a contiguous block of memory with calculated offsets**.

For example, conceptually:

```c
a[row][col]
```

maps to an offset like:

```text
(row × number_of_columns + col) × sizeof(int)
```

For this array:

```c
a[row][col]
```

becomes conceptually:

```text
(row × 2 + col) × 4
```

Understanding this makes compiler-generated address calculations much easier to recognize.

---

# Mental Model

```text
ARRAY PARAMETERS

int p[]
     ↓
int *p

int p[const]
     ↓
int *const p

int p[static 4]
     ↓
pointer + "at least 4 elements" promise
```

```text
PARTIAL INITIALIZATION

int a[5] = {1, 2};

        ↓

1  2  0  0  0
       ↑
   missing elements
   become zero
```

```text
2D ARRAY

int a[3][2] = {1,2,3,4,5,6};

        ↓

1  2
3  4
5  6

        ↓

contiguous memory
```

# Long-Term Takeaways

- Array parameters are adjusted to **pointer parameters**.
    
- Array parameter qualifiers go **inside `[]`**.
    
- `int p[const]` is equivalent to `int *const p`.
    
- `static` inside an array parameter specifies a **minimum number of elements** the caller promises to provide.
    
- `int p[static 4]` means `p` must point to at least 4 elements; violating this is **undefined behavior**.
    
- `static` here does **not** mean static storage duration.
    
- Missing array initializer elements become **zero**.
    
- Multidimensional arrays can use nested initializers or a flat initializer list.
    
- Flat multidimensional initializers fill elements **in order**.
    
- `int a[3][2] = {0};` is the idiomatic way to zero the entire array.
    
- Multidimensional arrays are stored **contiguously**, which is highly relevant to x86-64 reverse engineering.