# `typedef` — Making Type Aliases

## What is `typedef`?

`typedef` creates a new **name (alias)** for an existing type.

Syntax:

```c
typedef existing_type new_name;
```

Example:

```c
typedef int number;
```

Now:

```c
number x = 10;
```

is equivalent to:

```c
int x = 10;
```

> `typedef` creates an alias, not a new underlying type.

---

# `typedef` in Theory

General form:

```c
typedef old_type new_name;
```

Example:

```c
typedef unsigned long ulong;
```

Now:

```c
ulong x;
```

means:

```c
unsigned long x;
```

The compiler treats both as the same underlying type.

---

# Scoping

A `typedef` name follows normal C scope rules.

Example:

```c
void function(void)
{
    typedef int number;

    number x = 10;
}
```

Here, `number` exists only inside the function.

A file-scope `typedef` can be used throughout that source file after its declaration.

---

# `typedef` in Practice

`typedef` is commonly used to:

- Make long type names shorter
    
- Improve readability
    
- Simplify complex declarations
    
- Create convenient names for structs and pointers
    

Example:

```c
typedef unsigned long long ull;
```

Then:

```c
ull value;
```

---

# `typedef` and Structs

Without `typedef`:

```c
struct animal
{
    char *name;
    int leg_count;
    int speed;
};

struct animal y;
```

With `typedef`:

```c
typedef struct animal animal;

animal z;
```

Both:

```c
struct animal y;
```

and:

```c
animal z;
```

refer to the same struct type.

---

# Defining a Struct and `typedef` Together

You can define the struct and its alias at the same time:

```c
typedef struct animal
{
    char *name;
    int leg_count;
    int speed;
} animal;
```

Now:

```c
struct animal y;
animal z;
```

Both work.

---

# Anonymous Struct with `typedef`

A struct does not need a tag name:

```c
typedef struct
{
    char *name;
    int leg_count;
    int speed;
} animal;
```

Now:

```c
animal z;
```

works.

But:

```c
struct animal y;
```

does **not** work because no `animal` struct tag exists.

---

# Common Struct Pattern

Example:

```c
typedef struct
{
    int x;
    int y;
} point;
```

Usage:

```c
point p =
{
    .x = 20,
    .y = 40
};

printf("%d, %d\n", p.x, p.y);
```

Output:

```text
20, 40
```

This is a very common C pattern.

---

# `typedef` and Other Types

Example:

```c
typedef int score;
typedef char character;
typedef unsigned int uint;
```

Usage:

```c
score x = 100;
character c = 'A';
uint n = 50;
```

These remain aliases for their original types.

---

# `typedef` and Pointers

`typedef` can make pointer declarations easier to read.

Example:

```c
typedef int *int_ptr;
```

Now:

```c
int_ptr p;
```

means:

```c
int *p;
```

Example:

```c
int x = 10;

int_ptr p = &x;
```

---

# Why `typedef` Can Be Useful with Pointers

Consider:

```c
int *a, b;
```

This means:

```text
a → int pointer
b → int
```

But:

```c
typedef int *int_ptr;

int_ptr a, b;
```

means:

```text
a → int pointer
b → int pointer
```

Because both variables use the alias `int_ptr`.

---

# `typedef` and Capitalization

A common convention is to use capitalization for type aliases.

Example:

```c
typedef unsigned int UINT;
typedef struct Person PERSON;
```

However, capitalization is only a **naming convention**.

C does not require it.

Choose a consistent style.

---

# Arrays and `typedef`

`typedef` can simplify array declarations.

Example:

```c
typedef int array5[5];
```

Now:

```c
array5 numbers;
```

means:

```c
int numbers[5];
```

You can initialize it normally:

```c
array5 numbers = {1, 2, 3, 4, 5};
```

---

# Important: `typedef` Does Not Create a New Type

Example:

```c
typedef int number;

number x = 10;
int y = 20;
```

`number` and `int` are still the same underlying type.

Think:

```text
typedef
   │
   ▼
Existing Type ───► Another Name
```

Not:

```text
typedef
   │
   ▼
Completely New Type
```

---

# Reverse Engineering Perspective

## Type Aliases Do Not Exist at Runtime

Consider:

```c
typedef int score;

score x = 10;
```

At runtime, there is no special `score` type.

The CPU only sees an integer value.

```text
Source Code:

score x

        │

        ▼

Compiled Code:

int-sized value in memory/register
```

`typedef` exists mainly to help the programmer and compiler understand the source code.

---

## `typedef` Information Is Usually Lost During Compilation

These:

```c
typedef int score;
typedef int age;
```

are both simply integers in the final machine code.

Assembly does not know:

```text
score
age
```

It only operates on bytes and registers.

---

## Structs and Reverse Engineering

Source:

```c
typedef struct
{
    int x;
    int y;
} point;
```

Assembly may access:

```asm
mov eax, [rcx]
mov edx, [rcx + 4]
```

The reverse engineer may reconstruct it as:

```c
struct point
{
    int x;      // offset 0x0
    int y;      // offset 0x4
};
```

The original `typedef` name is usually unavailable unless debugging symbols exist.

---

# Mental Model

```text
Original Type
     │
     ▼
typedef
     │
     ▼
Alias
```

Example:

```text
int
 │
 └────── typedef ──────► number


struct {...}
 │
 └────── typedef ──────► point
```

---

# Long-Term Takeaways

- `typedef` creates a type alias.
    
- It does not create a new underlying type.
    
- `typedef` names follow normal C scope rules.
    
- It is commonly used to simplify complex declarations.
    
- `typedef` is especially useful with structs and pointers.
    
- Anonymous structs are commonly used with `typedef`.
    
- `typedef` can also simplify array types.
    
- Type aliases generally do not exist in compiled machine code.
    
- In reverse engineering, typedef names are often reconstructed manually from context.