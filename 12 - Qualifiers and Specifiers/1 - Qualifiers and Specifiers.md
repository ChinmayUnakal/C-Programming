
# Type Qualifiers

Type qualifiers tell the compiler **how a variable or object may be accessed or modified**.

The main type qualifiers in C are:

```text
const
restrict
volatile
_Atomic
```

The most important ones for C and reverse engineering are:

- `const` → don't modify through this access
    
- `restrict` → promise about pointer aliasing
    
- `volatile` → value may change outside normal program flow
    
- `_Atomic` → atomic access for multithreaded programs
    

---

# `const`

`const` tells the compiler that an object **must not be modified through that particular declaration**.

Example:

```c
const int x = 2;

x = 4;     // Compiler error
```

Conceptually:

```text
x
│
▼
+------+
|  2   |  ← cannot modify through x
+------+
```

The important point is:

> `const` is a restriction on modification through that particular access path.

It does **not** necessarily mean the underlying memory is physically impossible to change.

---

# `const` Function Parameters

You will often see `const` in function parameters.

```c
void foo(const int x)
{
    printf("%d\n", x + 30);
}
```

Inside the function:

```c
x = 50;     // Error
```

because `x` is declared `const`.

For a simple value parameter like `const int x`, the practical usefulness is relatively small because `x` is already a local copy.

`const` becomes much more important with **pointers**.

---

# `const` and Pointers

This is one of the most important parts to understand.

There are **two different things** you can make constant:

1. The data being pointed to
    
2. The pointer itself
    

---

# Pointer to `const`

```c
int x = 10;

const int *p = &x;
```

Read this as:

> `p` is a pointer to a `const int`.

You cannot modify `x` **through `p`**:

```c
*p = 20;       // Error
```

But you can change where `p` points:

```c
int y = 30;

p = &y;        // OK
```

Conceptually:

```text
       p
       │
       ▼
   +--------+
   |   10   |  ← cannot modify through p
   +--------+

p can point somewhere else
```

These are equivalent:

```c
const int *p;
int const *p;
```

Both mean:

```text
Pointer → const int
```

---

# `const` Pointer

Now move `const` **after the `*`**:

```c
int *const p = &x;
```

This means:

> `p` itself is constant.

Therefore:

```c
*p = 20;       // OK
```

But:

```c
p = &y;        // Error
```

The pointer cannot be changed to point somewhere else.

Conceptually:

```text
p
│
▼
+--------+
|   20   |  ← can modify
+--------+

p itself cannot change
```

---

# Pointer to `const` vs `const` Pointer

This is extremely important:

```c
const int *p;
```

means:

```text
Can't modify the data through p
Can modify p
```

While:

```c
int *const p;
```

means:

```text
Can modify the data through p
Can't modify p
```

Think:

```text
const int *p

     p
     │
     ▼
   const
    DATA
```

versus:

```text
int *const p

  const
    p
    │
    ▼
   DATA
```

---

# `const` Pointer to `const`

You can make **both** constant:

```c
const int *const p = &x;
```

Now:

```c
*p = 20;       // Error
p = &y;        // Error
```

Meaning:

```text
             p
             │
             ▼
        +---------+
        |  const  |
        |   int   |
        +---------+

      p cannot change
      data cannot change through p
```

---

# Reading Complex Pointer Declarations

A useful rule is:

> Start at the variable name and read outward.

For example:

```c
const int *const p;
```

Start with:

```text
p
```

Then:

```text
p → const pointer
```

Then:

```text
p → const pointer → to pointer-to-const int
```

The practical result:

```text
p cannot change
*p cannot change
```

---

# Multiple Levels of Indirection

With multiple pointers, `const` can apply to different levels.

Example:

```c
char **p;
```

There are three objects involved conceptually:

```text
p
│
▼
char *
│
▼
char
```

You can modify the pointer:

```c
p++;
```

You can modify the intermediate pointer:

```c
(*p)++;
```

And you can modify the character:

```c
**p = 'A';
```

---

## `char **const p`

```c
char **const p;
```

The outer pointer is constant.

```c
p++;          // Error
```

But:

```c
(*p)++;       // OK
**p = 'A';    // OK
```

Conceptually:

```text
       const p
          │
          ▼
       char *
          │
          ▼
        char
```

---

## `char *const *p`

```c
char *const *p;
```

Here the **middle pointer** is constant.

Therefore:

```c
p++;          // OK
```

but:

```c
(*p)++;       // Error
```

Conceptually:

```text
p
│
▼
const char *
│
▼
char
```

---

## `char *const *const p`

```c
char *const *const p;
```

Both the outer pointer and the intermediate pointer are constant.

```c
p++;          // Error
(*p)++;       // Error
**p = 'A';    // OK
```

So:

```text
p        → constant
*p       → constant pointer
**p      → modifiable char
```

---

# `const` Correctness

Consider:

```c
const int x = 20;

int *p = &x;
```

This is a problem.

The types are:

```text
x
│
▼
const int

&x
│
▼
const int *

p
│
▼
int *
```

You are trying to remove the `const` qualifier:

```text
const int *  →  int *
```

The compiler will normally warn or reject this conversion.

Why is this dangerous?

Because now you could try:

```c
*p = 40;
```

But `x` was declared `const`.

Modifying a `const` object through a non-const pointer results in **undefined behavior**.

---

# Important `const` Mental Model

Don't think:

> "`const` means this memory can never change."

Think:

> "`const` means this particular access path cannot be used to modify the object."

For example:

```c
int x = 10;

const int *p = &x;
```

`x` itself is not a `const` object.

You can still do:

```c
x = 20;       // OK
```

But:

```c
*p = 20;      // Error
```

because `p` provides a read-only access path.

---

# Reverse Engineering Perspective: `const`

`const` can affect how the compiler generates code.

Example:

```c
const int x = 10;
```

The compiler may treat `x` as a value that never changes and optimize accordingly.

With optimization enabled, you may see:

```asm
mov eax, 10
```

instead of an actual memory load.

Therefore:

> A source-level `const` variable might not exist as a recognizable memory variable in the final binary.

This is important when reverse engineering optimized programs.

---

# `restrict`

`restrict` is primarily a **pointer optimization promise**.

It tells the compiler:

> "For this access, I promise that this pointer is the primary way the pointed-to object is accessed."

In other words, it gives the compiler information about **aliasing**.

Example:

```c
void swap(int *restrict a, int *restrict b)
{
    int t;

    t = *a;
    *a = *b;
    *b = t;
}
```

When calling:

```c
int x = 10;
int y = 20;

swap(&x, &y);
```

`a` points to `x` and `b` points to `y`.

They refer to different objects.

That's fine.

But:

```c
swap(&x, &x);
```

violates the `restrict` contract because both restricted pointers refer to the same object in a way that conflicts with the promise.

The behavior is **undefined**.

---

# Why Does `restrict` Exist?

Consider:

```c
void add(int *restrict a, int *restrict b)
{
    *a = *a + *b;
}
```

Because of `restrict`, the compiler can make stronger assumptions about whether `a` and `b` overlap.

Without `restrict`, the compiler has to be more conservative.

With `restrict`, you are essentially telling the compiler:

```text
These pointers don't alias in a conflicting way.
```

That can allow more aggressive optimization.

---

# Aliasing

**Aliasing** occurs when multiple pointers can refer to the same memory.

Example:

```c
int x = 10;

int *a = &x;
int *b = &x;
```

Now:

```text
a ─────┐
       ▼
     +----+
     | 10 |
     +----+
       ▲
b ─────┘
```

Both `a` and `b` refer to the same object.

This is aliasing.

`restrict` allows you to promise that such conflicting aliasing won't happen for the relevant access.

---

# Reverse Engineering Perspective: `restrict`

`restrict` is particularly relevant when analyzing optimized code.

The compiler may use the no-aliasing promise to:

- eliminate unnecessary memory loads
    
- reorder operations
    
- keep values in registers
    
- perform vectorization
    
- reduce memory accesses
    

Therefore:

```c
restrict
```

can indirectly influence the assembly you see.

You usually **cannot reliably identify `restrict` from a stripped binary**, because it is primarily a compiler optimization contract rather than runtime metadata.

---

# `volatile`

`volatile` tells the compiler:

> "This value can change unexpectedly, so don't assume its value remains unchanged between accesses."

Example:

```c
volatile int *p;
```

Every access through `p` must be treated as potentially observable.

---

# Why Is `volatile` Needed?

Imagine hardware has a memory-mapped register:

```text
Hardware
   │
   ▼
Memory address
   │
   ▼
volatile int
```

The hardware might change the value without your program executing an instruction that writes to it.

Without `volatile`, the compiler might optimize repeated reads.

With:

```c
volatile int *p;
```

the compiler knows:

> "I need to actually perform the memory access."

---

# Example

```c
volatile int *status = (volatile int *)0x40000000;

while (*status == 0) {
    // wait
}
```

The hardware may change the value at:

```text
0x40000000
```

The compiler must not simply assume:

```text
*status
```

always has the same value.

---

# Where Is `volatile` Used?

You will commonly encounter it in:

- Embedded systems
    
- Memory-mapped hardware
    
- Hardware registers
    
- Signal handlers
    
- Low-level systems programming
    

For normal application programming, you rarely need it.

---

# Reverse Engineering Perspective: `volatile`

`volatile` can result in repeated memory accesses appearing in assembly.

C:

```c
volatile int x;

a = x;
b = x;
```

The compiler generally cannot simply assume both reads produce the same value.

You may therefore see separate memory loads.

This is especially relevant when reverse engineering:

- firmware
    
- drivers
    
- embedded software
    
- hardware interfaces
    

---

# Storage-Class Specifiers

Storage-class specifiers provide information about a variable's:

- **storage duration**
    
- **linkage**
    
- **scope-related behavior**
    
- **thread-local storage**
    

Common ones include:

```text
auto
static
extern
register
_Thread_local
```

---

# `auto`

`auto` is the default storage-class specifier for ordinary local variables.

These are equivalent:

```c
int x;
```

and:

```c
auto int x;
```

For example:

```c
void foo(void)
{
    int x = 10;
}
```

is effectively:

```c
void foo(void)
{
    auto int x = 10;
}
```

You almost never explicitly write `auto` in modern C.

> Don't confuse C's `auto` with C++'s `auto`, which has a completely different and much more commonly used purpose.

---

# `static`

`static` is extremely important because its meaning depends on **where it is used**.

There are two major cases:

```text
static at block scope
static at file scope
```

---

# `static` at Block Scope

Example:

```c
void counter(void)
{
    static int count = 1;

    printf("%d\n", count);

    count++;
}
```

Calling:

```c
counter();
counter();
counter();
```

produces:

```text
1
2
3
```

Why?

Normally:

```c
int count = 1;
```

would create a new local variable every time the function executes.

But:

```c
static int count = 1;
```

creates **one persistent object**.

Its value survives between function calls.

---

# Normal Local vs `static` Local

Normal local:

```c
void foo(void)
{
    int x = 0;
    x++;
    printf("%d\n", x);
}
```

Repeated calls:

```text
1
1
1
```

because `x` is recreated each time.

With `static`:

```c
void foo(void)
{
    static int x = 0;
    x++;
    printf("%d\n", x);
}
```

Repeated calls:

```text
1
2
3
```

because the same object persists.

---

# Lifetime of a `static` Local

A `static` local variable:

- has **static storage duration**
    
- exists for the entire execution of the program
    
- retains its value between function calls
    
- is only directly accessible within its block
    

Conceptually:

```text
Program starts
     │
     ▼
static x created
     │
     ├── function call
     │
     ├── function returns
     │
     ├── function call
     │
     └── program ends
```

The variable doesn't disappear when the function returns.

---

# Initialization of `static` Locals

A static local is initialized only once.

```c
void foo(void)
{
    static int x = 10;

    x++;
}
```

The `10` initialization does **not** happen every time `foo()` runs.

It happens once.

Also:

```c
static int x;
```

is automatically initialized to:

```c
0
```

So:

```c
static int x;
```

and:

```c
static int x = 0;
```

have the same initial value.

---

# Reverse Engineering Perspective: `static` Local

A static local variable is not normally stored on the stack.

It generally resides in a static/data area of the executable.

For example:

```c
void foo(void)
{
    static int count = 0;
    count++;
}
```

You may see assembly accessing a fixed memory location rather than:

```asm
[rbp-4]
```

Conceptually:

```asm
mov eax, DWORD PTR [rip+count]
add eax, 1
mov DWORD PTR [rip+count], eax
```

This distinction is useful when identifying variables in a binary.

---

# `static` at File Scope

Now consider:

```c
static int counter = 0;
```

outside every function.

Here, `static` means:

> This variable has **internal linkage** and cannot be accessed directly from another source file.

For example:

```text
file1.c
----------------
static int x = 10;
```

Another file:

```text
file2.c
----------------
extern int x;
```

cannot access that `x`.

The `static` variable belongs only to `file1.c`.

---

# Global vs File-Private Global

Without `static`:

```c
int x = 10;
```

the object can have external linkage and can be referenced from another source file using:

```c
extern int x;
```

With:

```c
static int x = 10;
```

the object has internal linkage.

Conceptually:

```text
int x
 │
 ├── file1.c
 └── potentially accessible from other files

static int x
 │
 └── file1.c only
```

---

# `static` Functions

`static` can also be used with functions.

Example:

```c
static void helper(void)
{
    printf("Hello");
}
```

This function has **internal linkage**.

It can only be referenced from that source file.

This is useful for creating private implementation functions.

---

# Reverse Engineering Perspective: File-Scope `static`

In a compiled binary, file-scope `static` variables/functions may not be exported as externally visible symbols.

In stripped binaries, however, even ordinary names may disappear, so don't assume:

```text
static = impossible to find
```

Instead, think:

> `static` affects linkage at the source/compiler level; reverse engineers infer the resulting behavior from the binary.

---

# `extern`

`extern` tells the compiler:

> "This object is defined somewhere else; I want to refer to it here."

Suppose:

```c
// bar.c

int a = 37;
```

Another file:

```c
// foo.c

extern int a;

int main(void)
{
    printf("%d\n", a);
}
```

The `a` in `foo.c` refers to the object defined in `bar.c`.

Conceptually:

```text
bar.c
    │
    ▼
+---------+
| a = 37  |
+---------+
    ▲
    │
 extern
    │
foo.c
```

---

# `extern` Does Not Create a New Object

This is important.

```c
extern int a;
```

does **not** normally allocate a new `int`.

It declares that an `int` named `a` exists elsewhere.

The actual definition is:

```c
int a = 37;
```

---

# `extern` and Multiple Source Files

Typical project:

```text
main.c
globals.c
globals.h
```

`globals.c`:

```c
int counter = 10;
```

`globals.h`:

```c
extern int counter;
```

`main.c`:

```c
#include "globals.h"

printf("%d\n", counter);
```

The linker connects the reference to the actual definition.

---

# `extern` Functions

Functions already have external linkage by default unless declared `static`.

Therefore:

```c
extern void foo(void);
```

is usually redundant.

Normally you simply write:

```c
void foo(void);
```

---

# Reverse Engineering Perspective: `extern`

`extern` is closely related to **linking and symbols**.

When multiple source files are compiled:

```text
foo.c ──┐
        │
bar.c ──┼──> Compiler ──> Linker ──> Executable
        │
baz.c ──┘
```

The linker resolves references between translation units.

This is important when reverse engineering:

- ELF binaries
    
- PE binaries
    
- symbol tables
    
- imports/exports
    
- shared libraries
    
- DLLs
    
- dynamic linking
    

---

# `register`

`register` was historically a hint to the compiler:

> "This variable is accessed frequently; consider keeping it in a CPU register."

Example:

```c
register int i;

for (i = 0; i < 10; i++)
    printf("%d\n", i);
```

However, modern compilers are much better at register allocation.

Therefore:

```c
register
```

is rarely useful today.

---

# Important Restriction of `register`

You cannot take the address of a `register` object.

```c
register int x = 10;

int *p = &x;     // Error
```

Why?

The compiler is allowed to place `x` in a CPU register rather than memory.

A register doesn't have a normal C memory address that you can take.

---

# `register` and Arrays

This also applies to arrays.

```c
register int a[] = {10, 20, 30};
```

You cannot treat the array as though you can simply obtain the address of its first element.

For example:

```c
int *p = a;      // Constraint violation
```

because this requires the address of `a[0]`.

Similarly, operations that require an address can be prohibited.

---

# Reverse Engineering Perspective: `register`

The keyword itself is mostly irrelevant when reverse engineering modern binaries.

The compiler already performs **register allocation**.

You may see:

```asm
mov eax, 10
add eax, 5
```

without any memory access.

That doesn't mean the source code necessarily contained:

```c
register int x;
```

The compiler may simply have decided that `x` belongs in `eax`.

Therefore:

> Never assume a register variable in assembly came from the C `register` keyword.

---

# `_Thread_local`

`_Thread_local` is used in multithreaded programs.

It gives each thread its **own separate instance** of a variable.

Example:

```c
_Thread_local int counter;
```

Conceptually:

```text
Thread 1
   │
   └── counter = 10

Thread 2
   │
   └── counter = 50

Thread 3
   │
   └── counter = 100
```

They are not sharing one `counter`.

Each thread gets its own copy.

---

# Why Is `_Thread_local` Useful?

Without thread-local storage:

```text
Thread 1 ──┐
           │
           ▼
        counter
           ▲
           │
Thread 2 ──┘
```

Multiple threads can access the same variable, potentially causing synchronization problems.

With:

```c
_Thread_local int counter;
```

you get:

```text
Thread 1 → counter₁

Thread 2 → counter₂

Thread 3 → counter₃
```

Each thread works with its own object.

This can avoid certain forms of shared-variable races.

---

# `_Thread_local` Scope Rules

At file scope:

```c
_Thread_local int x;
```

creates thread-local storage.

At block scope, `_Thread_local` must be combined with either:

```c
static
```

or:

```c
extern
```

For example:

```c
static _Thread_local int x;
```

---

# `thread_local`

If you include:

```c
#include <threads.h>
```

C provides:

```c
thread_local
```

as a more convenient spelling for:

```c
_Thread_local
```

So:

```c
thread_local int counter;
```

can be used instead of:

```c
_Thread_local int counter;
```

---

# Type Qualifiers vs Storage-Class Specifiers

This distinction is important.

## Type Qualifiers

They describe **how an object can be accessed**.

```text
const
restrict
volatile
_Atomic
```

Think:

> "What restrictions or guarantees apply to accessing this data?"

---

## Storage-Class Specifiers

They describe things such as **storage duration, linkage, and thread-local storage**.

```text
auto
static
extern
register
_Thread_local
```

Think:

> "How does this object exist and how is it connected to the rest of the program?"

---

# Quick Comparison

|Keyword|Main idea|
|---|---|
|`const`|Don't modify through this access|
|`restrict`|Promise about pointer aliasing|
|`volatile`|Value may change unexpectedly|
|`_Atomic`|Atomic access to an object|
|`auto`|Default local storage class|
|`static`|Persistent storage or internal linkage|
|`extern`|Object/function is defined elsewhere|
|`register`|Historical register-allocation hint|
|`_Thread_local`|Separate object for each thread|

---

# Reverse Engineering Perspective

These keywords can leave very different fingerprints in compiled code.

### `const`

May allow the compiler to:

```text
Constant propagation
Constant folding
Elimination of memory accesses
```

Example:

```c
const int x = 10;
```

might become:

```asm
mov eax, 10
```

---

### `restrict`

May allow:

```text
Better alias analysis
Instruction reordering
Vectorization
Fewer memory loads
```

You generally cannot identify `restrict` directly from stripped assembly.

---

### `volatile`

Often forces actual memory accesses.

You may see:

```asm
mov eax, DWORD PTR [address]
```

repeatedly instead of the compiler keeping the value in a register.

---

### `static` Local

Often corresponds to storage outside the stack:

```text
.data
.bss
```

or equivalent sections.

Instead of:

```asm
[rbp-4]
```

you may see:

```asm
[rip+offset]
```

---

### `static` File-Scope

Creates internal linkage.

The symbol may not be externally visible.

---

### `extern`

Creates references that the linker resolves.

This becomes important when examining:

```text
Imports
Exports
Symbols
Relocations
PLT/GOT
DLL references
```

depending on the platform and binary format.

---

### `register`

Usually has no special reverse-engineering significance today.

The compiler performs register allocation automatically.

---

### `_Thread_local`

May result in accesses through **thread-local storage mechanisms** rather than ordinary global memory.

On x86-64, you may encounter mechanisms involving:

```text
FS / GS
TLS
thread-local storage offsets
```

depending on the operating system and ABI.

---

# Variable Storage Mental Model

Putting everything together:

```text
                         C VARIABLE
                             │
              ┌──────────────┴──────────────┐
              │                             │
       Type Qualifiers              Storage Class
              │                             │
      ┌───────┼────────┐          ┌─────────┼─────────┐
      │       │        │          │         │         │
    const restrict volatile      static    extern   _Thread_local
      │       │        │          │
      │       │        │          ├── persistent local
      │       │        │          │
      │       │        │          └── file-private
      │       │        │
      │       │        └── externally changing value
      │       │
      │       └── aliasing promise
      │
      └── read-only access
```

---

# Reverse Engineering Mental Model

When you see a variable-like value in assembly, ask:

```text
Where is it stored?
      │
      ├── Stack?
      │      └── [rbp-offset]
      │
      ├── Register?
      │      └── eax, ebx, rdi, etc.
      │
      ├── Global/data section?
      │      └── RIP-relative memory access
      │
      └── Thread-local storage?
             └── TLS / FS / GS mechanisms
```

Then ask:

```text
Can it change?
      │
      ├── const?
      ├── volatile?
      ├── normal variable?
      └── modified through another pointer?
```

Then:

```text
Can multiple pointers refer to it?
      │
      └── Think about aliasing / restrict
```

And finally:

```text
Who can access it?
      │
      ├── local scope
      ├── file scope
      ├── external linkage
      └── thread-local instance
```

---

# Final Mental Model

When writing C:

```text
Source Code
    │
    ▼
Variable Declaration
    │
    ├── Type
    ├── Qualifiers
    └── Storage Class
    │
    ▼
Compiler
    │
    ▼
Storage + Access Rules
    │
    ├── Register
    ├── Stack
    ├── Global/Data
    └── Thread-Local Storage
    │
    ▼
Machine Code
```

When reverse engineering:

```text
Machine Code
     │
     ▼
Registers + Memory Accesses
     │
     ▼
Storage Location
     │
     ▼
Infer Lifetime / Scope / Aliasing
     │
     ▼
Infer Original Variable
     │
     ▼
Infer Programmer's Intent
```

---

# Long-Term Takeaways

- `const` → **don't modify through this access path**.
    
- `const int *p` → data is read-only **through `p`**.
    
- `int *const p` → pointer itself cannot change.
    
- `const int *const p` → neither pointer nor pointed-to data can be modified through `p`.
    
- `restrict` → **aliasing promise** that enables optimization.
    
- `volatile` → compiler must treat accesses as potentially changing externally.
    
- `static` local → **same variable survives between function calls**.
    
- `static` file-scope → **internal linkage; private to that source file**.
    
- `extern` → refers to an object/function defined elsewhere.
    
- `register` → old optimization hint; modern compilers generally handle registers themselves.
    
- `_Thread_local` → **each thread gets its own copy**.
    
- In reverse engineering, these keywords usually aren't visible directly; you infer their effects from **memory accesses, registers, sections, symbols, TLS, and compiler behavior**.