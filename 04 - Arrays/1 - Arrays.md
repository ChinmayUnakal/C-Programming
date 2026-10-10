
## What is an Array?

An **array** is a collection of elements of the **same type** stored in **contiguous memory locations**.

Example:

```c
int numbers[5];
```

This creates space for 5 `int` values.

---

# Why Use Arrays?

Arrays allow you to store multiple related values under one name.

Instead of:

```c
int score1, score2, score3;
```

you can use:

```c
int scores[3];
```

---

# Easy Example

```c
int numbers[5] = {10, 20, 30, 40, 50};
```

Access elements using an **index**:

```c
numbers[0];    // 10
numbers[1];    // 20
numbers[4];    // 50
```

Arrays are **zero-indexed**.

```text
Index:    0    1    2    3    4
        ┌────┬────┬────┬────┬────┐
Value:  │ 10 │ 20 │ 30 │ 40 │ 50 │
        └────┴────┴────┴────┴────┘
```

---

# Memory Layout

Array elements are stored **next to each other** in memory.

For:

```c
int numbers[3] = {10, 20, 30};
```

Conceptually:

```text
Address     Value
0x1000  →    10
0x1004  →    20
0x1008  →    30
```

Each `int` typically uses 4 bytes.

---

# Getting the Length of an Array

For an actual array in the same scope:

```c
int numbers[] = {10, 20, 30, 40, 50};

size_t length = sizeof(numbers) / sizeof(numbers[0]);
```

`sizeof(numbers)` gives the total array size.

`sizeof(numbers[0])` gives the size of one element.

Example:

```text
Total size:       20 bytes
One element:       4 bytes
--------------------------
Length:            5
```

---

# Array Initializers

Full initialization:

```c
int a[3] = {10, 20, 30};
```

Let the compiler determine the size:

```c
int a[] = {10, 20, 30};
```

Partial initialization:

```c
int a[5] = {10, 20};
```

Remaining elements are initialized to `0`.

```text
{10, 20, 0, 0, 0}
```

Initialize all elements to zero:

```c
int a[5] = {0};
```

---

# Out of Bounds

Valid indices for:

```c
int a[5];
```

are:

```text
0, 1, 2, 3, 4
```

This is invalid:

```c
a[5] = 10;
```

C does **not** automatically check array bounds.

Accessing outside an array causes **undefined behavior**.

This can cause:

- Crashes
    
- Memory corruption
    
- Security vulnerabilities
    

---

# Multidimensional Arrays

An array can contain other arrays.

Example:

```c
int matrix[2][3] =
{
    {1, 2, 3},
    {4, 5, 6}
};
```

Access:

```c
matrix[0][1];    // 2
matrix[1][2];    // 6
```

Memory is still contiguous.

```text
1  2  3  4  5  6
```

C stores multidimensional arrays in **row-major order**.

---

# Arrays and Pointers

In most expressions, an array name converts ("decays") to a pointer to its first element.

```c
int a[3] = {10, 20, 30};
```

Then:

```c
a
```

usually behaves like:

```c
&a[0]
```

Therefore:

```c
a[0] == *(a + 0)
a[1] == *(a + 1)
a[2] == *(a + 2)
```

Array indexing is based on pointer arithmetic.

---

# Important: Array ≠ Pointer

An array is **not actually a pointer**.

```c
int a[5];
int *p;
```

They are different types.

However, the array name often converts to a pointer to its first element.

A useful mental model:

```text
Array name
    │
    ▼
First element
```

But remember:

> **Array decay does not mean an array is a pointer.**

---

# Getting a Pointer to an Array

For:

```c
int a[5];
```

These are different:

```c
a       // Usually converts to int *
&a[0]   // Pointer to first int: int *
&a      // Pointer to entire array: int (*)[5]
```

Example:

```c
int (*p)[5] = &a;
```

Here `p` points to the entire array.

---

# Passing Single-Dimensional Arrays to Functions

These declarations are equivalent as function parameters:

```c
void print_array(int a[])
```

```c
void print_array(int *a)
```

The array decays to a pointer when passed to the function.

Example:

```c
void print_array(int a[], int length)
{
    for (int i = 0; i < length; i++)
    {
        printf("%d\n", a[i]);
    }
}
```

Call:

```c
int numbers[] = {10, 20, 30};

print_array(numbers, 3);
```

---

# Important: sizeof in Functions

This does **not** give the array length:

```c
void function(int a[])
{
    sizeof(a);
}
```

Inside the function, `a` is treated as a pointer.

Therefore, pass the length separately:

```c
void function(int a[], size_t length)
```

---

# Changing Arrays in Functions

Because the function receives access to the original array elements, changes affect the original array.

Example:

```c
void set_first(int a[])
{
    a[0] = 100;
}
```

```c
int numbers[] = {10, 20, 30};

set_first(numbers);
```

Result:

```text
{100, 20, 30}
```

---

# Passing Multidimensional Arrays to Functions

The compiler must know the size of each row to calculate element addresses.

Example:

```c
void print_matrix(int rows, int a[][3])
{
    for (int i = 0; i < rows; i++)
    {
        for (int j = 0; j < 3; j++)
        {
            printf("%d ", a[i][j]);
        }
    }
}
```

The column size `3` must be known.

Equivalent pointer form:

```c
void print_matrix(int rows, int (*a)[3])
```

---

# Reverse Engineering Perspective (x86-64 Assembly)

## Array Access

C:

```c
x = a[i];
```

The CPU calculates:

```text
Address of a + (i × element size)
```

For an `int`:

```text
address = base + (i × 4)
```

Possible assembly:

```asm
mov eax, [rbx + rcx*4]
```

Where:

```text
rbx = array base address
rcx = index
4   = size of int
```

---

## Array Indexing

C:

```c
a[i]
```

is equivalent to:

```c
*(a + i)
```

This is why arrays and pointers are closely connected in C.

---

## Multidimensional Arrays

For:

```c
int matrix[2][3];
```

Accessing:

```c
matrix[i][j]
```

requires calculating:

```text
base + ((i × columns) + j) × sizeof(int)
```

For 3 columns:

```text
base + ((i × 3) + j) × 4
```

This address calculation is commonly visible in assembly.

---

## Out-of-Bounds Access

The CPU does **not know array boundaries**.

It simply accesses the address it is given.

This is why an instruction such as:

```asm
mov eax, [rbx + rcx*4]
```

does not check whether `rcx` is a valid index.

The programmer or compiler must ensure bounds are valid.

---

# Mental Model

```text
Array
  │
  ▼
Contiguous Memory

[10][20][30][40][50]
  │
  │
  └──► Access using index

a[i]
=
*(a + i)
```

For reverse engineering:

```text
Base Address + Index × Element Size
```

---

# Long-Term Takeaways

- An array stores multiple elements of the same type.
    
- Array elements are stored contiguously in memory.
    
- Arrays start at index `0`.
    
- `a[i]` is equivalent to `*(a + i)`.
    
- Arrays often decay to pointers, but arrays and pointers are not the same.
    
- C does not perform automatic bounds checking.
    
- Passing an array to a function effectively passes access to its first element.
    
- `sizeof` cannot determine the original array length after it has decayed to a pointer.
    
- Multidimensional arrays use row-major memory layout.
    
- In assembly, array access is usually **base address + index × element size**.