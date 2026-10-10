
## What is a Pointer?

A **pointer** is a variable that stores the **memory address** of another variable.

Instead of storing a value, it stores **where that value is located** in memory.

---

# Why Use Pointers?

Pointers allow a program to:

- Access memory directly
    
- Modify variables through their addresses
    
- Pass large objects efficiently
    
- Allocate memory dynamically
    
- Build data structures (arrays, linked lists, trees, etc.)
    

---

# Pointer Syntax

Declaration:

```c
int *p;
```

Initialization:

```c
int i = 10;
int *p = &i;
```

or

```c
int i = 10;

int *p;

p = &i;
```

`&` is the **address-of operator**.

It returns the memory address of a variable.

---

# Memory Representation

```text
Variable

i = 10
Address: 0x1000

Pointer

p = 0x1000
```

Diagram:

```text
+-------+          +-------+
|   p   | -------> |   i   |
|0x1000 |          |  10   |
+-------+          +-------+
```

---

# Printing a Pointer

Use the `%p` format specifier.

Example:

```c
printf("%p\n", (void *)p);
```

The cast to `(void *)` is the recommended portable way to print pointers.

---

# Dereferencing

The `*` operator accesses the value stored at the address held by a pointer.

Example:

```c
int i = 10;
int *p = &i;

printf("%d\n", *p);
```

Output:

```text
10
```

`*p` means:

> "Go to the address stored in `p` and access the value there."

---

# Modifying Data Through a Pointer

Example:

```c
int i = 10;
int *p = &i;

*p = 20;
```

Result:

```text
i = 20
```

Memory:

```text
Before

p ───► 10

After

*p = 20

p ───► 20
```

The pointer did **not** change.

The value at the address changed.

---

# Passing Pointers to Functions

Normally, C passes arguments **by value**.

Passing a pointer allows a function to modify the original variable.

Example:

```c
void increment(int *p)
{
    (*p)++;
}
```

Calling:

```c
int i = 10;

increment(&i);
```

Result:

```text
i = 11
```

Equivalent:

```c
int *ptr = &i;

increment(ptr);
```

Both pass the same address.

---

# Passing by Value vs Pointer

Passing by value:

```text
Main

i = 10
 │
 │ copy
 ▼

Function

x = 10
```

Changing `x` does **not** affect `i`.

---

Passing a pointer:

```text
Main

i = 10
 │
 ▼
Address
 │
 ▼

Function

*p
```

Both refer to the **same memory location**.

Changing `*p` changes `i`.

---

# NULL Pointer

`NULL` represents a pointer that points to **nothing**.

Example:

```c
int *p = NULL;
```

Use `NULL` when a pointer should not point to a valid object yet.

Always check before dereferencing:

```c
if (p != NULL)
{
    printf("%d\n", *p);
}
```

Dereferencing a `NULL` pointer causes **undefined behavior** (commonly a crash).

---

# Reverse Engineering Perspective (x86-64 Assembly)

## Addresses

Pointers are simply **memory addresses**.

Example:

```c
int *p = &i;
```

Assembly may load an address:

```asm
lea rax, [rbp-4]
```

`lea` (Load Effective Address) computes an address without reading memory.

---

## Dereferencing

C:

```c
x = *p;
```

Assembly:

```asm
mov eax, [rax]
```

The CPU reads the value stored at the address in `rax`.

---

## Writing Through a Pointer

C:

```c
*p = 20;
```

Assembly:

```asm
mov DWORD PTR [rax], 20
```

The CPU writes directly to memory.

---

## Passing Pointers

C:

```c
increment(&i);
```

Assembly:

```asm
lea rcx, [rbp-4]
call increment
```

The function receives the address of `i`, not its value.

---

# Mental Model

```text
Variable
   │
   ▼
Memory Address
   ▲
   │
Pointer
```

Dereferencing:

```text
Pointer
   │
   ▼
Address
   │
   ▼
Value
```

---

# Long-Term Takeaways

- A pointer stores an **address**, not a value.
    
- `&` gets the address of a variable.
    
- `*` accesses the value stored at that address.
    
- Changing `*p` changes the original variable.
    
- Passing pointers allows functions to modify caller data.
    
- `NULL` means the pointer points to nothing.
    
- In assembly, pointers are simply memory addresses manipulated with instructions like `lea` and `mov`.