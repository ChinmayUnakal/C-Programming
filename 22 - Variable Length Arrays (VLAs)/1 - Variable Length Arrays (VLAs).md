# Variable Length Arrays (VLAs)

A **Variable Length Array (VLA)** is an array whose size is determined **at runtime**.

Unlike a normal array:

```c
int v[10];
```

a VLA can use a runtime value:

```c
int n = 10;
int v[n];
```

The size is determined when execution reaches the declaration and **cannot be changed afterward**.

VLAs provide runtime-sized arrays without explicitly using `malloc()`/`free()`.

---

# VLA Support

VLAs were mandatory in **C99** but became **optional in C11**.

The macro:

```c
__STDC_NO_VLA__
```

is defined as `1` when VLA support is unavailable.

Example:

```c
#if __STDC_NO_VLA__ == 1
#error Sorry, need VLAs for this program!
#endif
```

Compiler support varies, so VLAs should not be assumed to be available everywhere.

---

# Basic VLA

Normal array:

```c
int v[10];
```

VLA:

```c
int n = 10;
int v[n];
```

The size can also be an expression:

```c
int v[x * 100];
```

Example:

```c
int n;

scanf("%d", &n);

int v[n];
```

Here, `n` determines the array size at runtime.

---

# VLA Lifetime

A VLA is normally allocated when execution reaches its declaration:

```c
int n = 10;

int v[n];
```

At this point:

```text
n = 10
 ↓
VLA created
 ↓
10 elements
```

Changing `n` later does **not** resize the array:

```c
n = 20;
```

`v` is still an array of 10 elements.

VLAs typically have **automatic storage duration**, meaning they are normally stored on the stack.

---

# VLA Restrictions

VLAs have some important restrictions:

- Cannot be declared at **file scope**.
    
- Cannot have `static` storage duration.
    
- Cannot use an initializer list.
    

For example:

```c
int n = 5;

int v[n] = {1, 2, 3, 4, 5}; // Not allowed
```

A negative or otherwise invalid runtime size can result in **undefined behavior**.

---

# sizeof() and VLAs

`sizeof` works with VLAs just like normal arrays.

Example:

```c
int n = 10;
int v[n];

printf("%zu\n", sizeof v);
```

The important difference is:

```text
Normal array with compile-time size
        ↓
sizeof usually determined at compile time

VLA
        ↓
sizeof evaluated at runtime
```

---

# Number of Elements

The usual array-size calculation works with VLAs:

```c
size_t num_elems = sizeof v / sizeof v[0];
```

For example:

```c
int n = 10;
int v[n];

size_t count = sizeof v / sizeof v[0];
```

`count` will be `10`.

---

# VLA Pointer Arithmetic

VLAs behave like regular arrays in memory.

Example:

```c
int n = 5;
int v[n];

int *p = v;

p[2] = 12;

printf("%d\n", v[2]); // 12

p[3] = 34;

printf("%d\n", v[3]); // 34
```

The array-to-pointer relationship is the same as with ordinary arrays.

```text
v
↓
┌────┬────┬────┬────┬────┐
│  0 │  0 │ 12 │ 34 │  0 │
└────┴────┴────┴────┴────┘
       ↑         ↑
      p+2       p+3
```

---

# sizeof() Without Declaring a VLA

You can use a VLA type directly inside `sizeof`:

```c
int x = 12;

printf("%zu\n", sizeof(int[x]));
```

For a typical 4-byte `int`:

```text
48
```

The actual result depends on `sizeof(int)` on the system.

---

# Multidimensional VLAs

Multiple dimensions can be runtime-sized:

```c
int w = 10;
int h = 20;

int x[h][w];
```

You can also mix fixed and variable dimensions:

```c
int y[5][w];

int z[10][w][20];
```

The dimensions are part of how the compiler calculates element addresses.

---

# Passing a VLA to a Function

A one-dimensional VLA can be passed to a function like a normal array.

Example:

```c
int sum(int count, int *v)
{
    int total = 0;

    for (int i = 0; i < count; i++)
        total += v[i];

    return total;
}
```

Calling it:

```c
int a = 5;
int v[a];

printf("%d\n", sum(a, v));
```

The VLA itself is not copied into the function.

The array is effectively passed as a **pointer to its first element**.

---

# Specifying VLA Size in Parameters

You can explicitly specify the VLA dimension:

```c
int sum(int count, int v[count])
{
    int total = 0;

    for (int i = 0; i < count; i++)
        total += v[i];

    return total;
}
```

Here:

```text
count → specifies the VLA size
v     → array parameter
```

The size parameter should appear before the VLA parameter.

---

# VLA Function Prototypes

A VLA parameter can be written in a prototype as:

```c
void do_something(int count, int v[count]);
```

Or without naming the size parameter:

```c
void do_something(int, int v[*]);
```

Important:

```c
int v[*]
```

is valid for a **function prototype**, not the actual function definition.

The definition must specify the size expression:

```c
void do_something(int count, int v[count])
{
    // ...
}
```

---

# Passing Multidimensional VLAs

Multidimensional VLAs can also be passed to functions.

Example:

```c
void print_matrix(int h, int w, int m[h][w])
{
    for (int row = 0; row < h; row++) {
        for (int col = 0; col < w; col++)
            printf("%2d ", m[row][col]);

        printf("\n");
    }
}
```

Calling:

```c
int rows = 4;
int cols = 7;

int matrix[rows][cols];

print_matrix(rows, cols, matrix);
```

The dimensions allow the compiler to correctly calculate:

```c
m[row][col]
```

---

# Partial Multidimensional VLAs

Some dimensions can be variable while others are fixed.

Example:

```c
void print_records(int count, int record[count][5])
{
    for (int i = 0; i < count; i++) {
        for (int j = 0; j < 5; j++)
            printf("%2d ", record[i][j]);

        printf("\n");
    }
}
```

Here:

```text
Rows    → variable
Columns → fixed at 5
```

The array:

```c
int records[count][5];
```

contains `count` records, with exactly 5 integers per record.

---

# VLAs and Regular Arrays

VLAs and normal arrays have compatible memory layouts.

Therefore, they can generally be passed to the same functions when their dimensions are compatible.

Example:

```c
void foo(int h, int w, int m[h][w])
{
    // ...
}
```

A regular array can be passed:

```c
int matrix[3][5];

foo(3, 5, matrix);
```

A VLA can also be passed to a function expecting compatible dimensions.

### Important

The dimensions must be **compatible**.

Incorrect dimensions can lead to incorrect memory access and **undefined behavior**.

---

# typedef and VLAs

You can use `typedef` with a VLA:

```c
int w = 10;

typedef int goat[w];
```

At the point where the `typedef` is evaluated, `w` is `10`.

Therefore:

```c
goat x;
```

is effectively an array of 10 `int`s.

Changing `w` later:

```c
w = 20;
```

does **not** change `goat`.

```text
typedef executed
      ↓
w = 10
      ↓
goat = array of 10 int
      ↓
w = 20
      ↓
goat is STILL 10 int
```

Also, an initializer list cannot be used for this array type.

---

# VLAs and goto / longjmp

VLAs have special control-flow rules.

Be careful when using:

- `goto`
    
- `longjmp()`
    

around VLA declarations and lifetimes.

Incorrect control flow can violate VLA lifetime rules, and `longjmp()` can also result in resources associated with VLAs being lost.

These cases are advanced and should be handled carefully.

---

# VLA Memory Model

VLAs are normally allocated on the **stack**.

Conceptually:

```text
Higher Addresses
────────────────────
Function arguments
Return address
Local variables
VLA
────────────────────
Lower Addresses
```

For example:

```c
int n = 100;
int v[n];
```

creates space for 100 integers in the function's automatic storage area.

---

# Why VLAs Can Be Dangerous

VLAs have several practical disadvantages.

### 1. Limited Stack Size

Because VLAs normally use the stack:

```c
int n = user_input;

int v[n];
```

a very large `n` can cause **stack exhaustion/overflow**.

This is especially important when `n` comes from untrusted input.

---

### 2. Runtime Overhead

VLA allocation requires runtime work.

Fixed-size arrays can often be handled more efficiently by the compiler.

The overhead is usually small, but it matters in low-level/system software.

---

### 3. Compiler Support

VLAs are optional since C11 and are not supported equally by all C compilers.

Therefore, code using VLAs can have portability issues.

---

### 4. No Convenient Allocation-Failure Handling

Unlike:

```c
malloc()
```

where you can check:

```c
if (p == NULL)
```

there is no equivalent standard C return value for VLA allocation failure.

A sufficiently large VLA can instead lead to serious runtime failure, such as stack exhaustion.

---

### 5. Security Implications

If a VLA size comes from untrusted input:

```c
int n = user_input;

int buffer[n];
```

an attacker-controlled value could cause excessive stack usage.

This makes VLA-related code worth paying attention to during **vulnerability research and reverse engineering**.

---

# Why the Linux Kernel Avoids VLAs

The Linux kernel banned VLAs because of several concerns:

- Many uses could simply be fixed-size arrays.
    
- Runtime VLA allocation has additional overhead.
    
- Compiler support is inconsistent.
    
- VLAs consume limited kernel stack space.
    
- An unexpectedly large VLA can cause stack exhaustion.
    
- VLA sizes may come from values that are difficult to safely control.
    

The stack is particularly important in kernel code because kernel stacks are limited.

---

# Reverse Engineering Perspective

VLAs are especially interesting because their size is determined **at runtime**.

A compiler may generate code that:

```text
Calculate size
     ↓
Align size
     ↓
Adjust stack pointer
     ↓
Use stack space as array
```

Conceptually:

```asm
; simplified conceptual example

mov     eax, [n]
imul    eax, 4          ; n * sizeof(int)
sub     rsp, rax        ; allocate stack space
```

The exact assembly depends on the compiler, architecture, optimization level, and ABI.

When reversing a function, runtime stack-pointer adjustments can therefore be a clue that the function contains a **dynamic-sized local object**, such as a VLA.

---

# VLA vs malloc()

|Feature|VLA|`malloc()`|
|---|---|---|
|Size|Runtime|Runtime|
|Typical storage|Stack|Heap|
|Manual `free()`|No|Yes|
|Allocation failure handling|Limited|`NULL` can be checked|
|Lifetime|Automatic|Until `free()`|
|Large allocations|Risky|Generally more suitable|
|C11 support|Optional|Standard|
|Can use initializer list|No|No|

---

# Mental Model

```text
                VLA
                 │
        Runtime size determined
                 │
                 ▼
          Stack allocation
                 │
                 ▼
          Array is created
                 │
                 ▼
        Size cannot be changed
                 │
                 ▼
       Function/block ends
                 │
                 ▼
        Storage is released
```

---

# Long-Term Takeaways

- A **VLA** is an array whose size is determined at runtime.
    
- VLAs were mandatory in **C99** but became optional in **C11**.
    
- `__STDC_NO_VLA__ == 1` indicates that VLA support is unavailable.
    
- A VLA's size is determined when execution reaches its declaration.
    
- The size **cannot be changed** afterward.
    
- VLAs normally have automatic storage duration and are typically allocated on the **stack**.
    
- VLAs cannot be declared at file scope or with `static` storage duration.
    
- VLAs cannot use initializer lists.
    
- `sizeof` on a VLA is evaluated at **runtime**.
    
- Pointer arithmetic works with VLAs just like regular arrays.
    
- One-dimensional VLAs can be passed to functions as array/pointer parameters.
    
- Multidimensional VLAs can specify runtime dimensions in function parameters.
    
- `[*]` can be used for VLA dimensions in **function prototypes**.
    
- Fixed and variable dimensions can be combined.
    
- Regular arrays and VLAs can be passed interchangeably when their dimensions are compatible.
    
- `typedef` captures the dimension values at the point where the `typedef` is evaluated.
    
- `goto` and `longjmp()` require special care around VLAs.
    
- VLAs can cause **stack exhaustion** when their size is too large.
    
- Runtime-sized stack allocations are important to recognize during **reverse engineering and vulnerability research**.