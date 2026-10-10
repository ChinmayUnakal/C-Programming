# Compound Literals

A **compound literal** lets you create an unnamed object of a specified type directly inside an expression.

It is useful when you need a temporary array, `struct`, or other object **without declaring a separate variable**.

### Syntax

```c
(type){initializer}
```

Examples:

```c
(int){3490}

(int []){1, 2, 3, 4}

(struct coord){.x = 10, .y = 20}
```

The object has **no variable name**, but it still exists in memory and can have an address.

---

# Array Compound Literals

```c
int *p = (int []){1, 2, 3, 4};

printf("%d\n", p[1]);    // 2
```

Here:

```c
(int []){1, 2, 3, 4}
```

creates an unnamed array of four `int`s.

Conceptually:

```text
(int []){1, 2, 3, 4}

      ↓

+----+----+----+----+
| 1  | 2  | 3  | 4  |
+----+----+----+----+
  ^
  |
  p
```

Compare:

```c
int p[] = {1, 2, 3, 4};
```

vs.

```c
int *p = (int []){1, 2, 3, 4};
```

The first creates a **named array**.

The second creates an **unnamed array** and stores its address in `p`.

---

# Passing Compound Literals to Functions

One of the most useful applications is passing temporary objects directly to functions.

Normally:

```c
int sum(int p[], int count)
{
    int total = 0;

    for (int i = 0; i < count; i++)
        total += p[i];

    return total;
}
```

Without a compound literal:

```c
int a[] = {1, 2, 3, 4};

int s = sum(a, 4);
```

With a compound literal:

```c
int s = sum((int []){1, 2, 3, 4}, 4);
```

No separate array variable is required.

### Mental Model

```text
(int []){1,2,3,4}
        │
        ▼
 unnamed array
        │
        ▼
      pointer
        │
        ▼
       sum()
```

---

# Struct Compound Literals

Compound literals also work with `struct`s.

```c
#include <stdio.h>

struct coord {
    int x, y;
};

void print_coord(struct coord c)
{
    printf("%d, %d\n", c.x, c.y);
}

int main(void)
{
    print_coord((struct coord){.x = 10, .y = 20});
}
```

Output:

```text
10, 20
```

Normally you would write:

```c
struct coord t = {.x = 10, .y = 20};

print_coord(t);
```

The compound literal eliminates the temporary variable.

---

# Pointers to Compound Literals

You can take the address of a compound literal using `&`.

```c
#include <stdio.h>

struct coord {
    int x, y;
};

void print_coord(struct coord *c)
{
    printf("%d, %d\n", c->x, c->y);
}

int main(void)
{
    print_coord(&(struct coord){.x = 10, .y = 20});
}
```

The important part is:

```c
&(struct coord){.x = 10, .y = 20}
```

Meaning:

```text
create unnamed struct
        ↓
take its address
        ↓
pass pointer to function
```

The same idea works with simple types:

```c
foo(&(int){3490});
```

This creates an unnamed `int` containing `3490` and passes its address.

---

# Compound Literal Lifetime and Scope

A compound literal creates an actual object. It is **not just a constant value**.

Its lifetime depends on where it appears.

For a block-scope compound literal, its lifetime lasts until execution leaves that block.

Therefore, this is invalid:

```c
int *p;

{
    p = &(int){10};
}

printf("%d\n", *p);    // Undefined behavior
```

The compound literal's object no longer exists after leaving the block.

### Same Rule as Local Variables

Think of:

```c
int x = 10;
```

and:

```c
(int){10}
```

as objects with similar local lifetime rules.

You cannot safely do:

```c
int *get_value(void)
{
    return &(int){3490};    // WRONG
}
```

because the compound literal's lifetime ends when the function/block ends.

This is conceptually similar to:

```c
int *get_value(void)
{
    int x = 3490;

    return &x;              // WRONG
}
```

### Important

**Never return a pointer to a block-scope compound literal.**

---

# Simple Compound Literals

You can create a compound literal of almost any object type.

For example:

```c
int x = 3490;

printf("%d\n", x);
printf("%d\n", 3490);
printf("%d\n", (int){3490});
```

All produce:

```text
3490
```

The third one creates an actual unnamed `int` object, so it is usually unnecessary for simple values.

Compound literals become much more useful with:

- Arrays
    
- `struct`s
    
- Pointers
    
- Complex temporary objects
    

---

# Compound Literals — RE Perspective

Compound literals are important for understanding that **C expressions can create real objects in memory**.

For example:

```c
foo(&(struct coord){.x = 10, .y = 20});
```

Conceptually involves:

```text
stack/object storage
        │
        ├── x = 10
        └── y = 20
             │
             ▼
          pointer
             │
             ▼
            foo()
```

When reverse engineering, you may encounter code where the compiler creates a temporary structure/array, initializes it, and passes its address to another function.

At the assembly level, this may look like:

```asm
mov     DWORD PTR [rsp+...], 10
mov     DWORD PTR [rsp+...], 20
lea     rdi, [rsp+...]       ; pointer to temporary object
call    foo
```

So understanding compound literals helps you recognize **compiler-generated temporary objects**.

---

# Generic Selections — `_Generic`

`_Generic` is a C feature that allows code to select an expression based on its **type**.

The selection happens at **compile time**, not runtime.

### Syntax

```c
_Generic(
    expression,
    type1: result1,
    type2: result2,
    default: result_default
)
```

Think of it as a **compile-time type-based selection**.

---

# Basic Example

```c
#include <stdio.h>

int main(void)
{
    int i;
    float f;
    char c;

    char *s = _Generic(i,
        int: "that variable is an int",
        float: "that variable is a float",
        default: "that variable is some type"
    );

    printf("%s\n", s);
}
```

Output:

```text
that variable is an int
```

Why?

The first argument is:

```c
i
```

Its type is:

```c
int
```

Therefore:

```c
int: "that variable is an int"
```

is selected.

Conceptually:

```c
_Generic(i,
    int: "that variable is an int",
    float: "that variable is a float",
    default: "that variable is some type"
)
```

becomes:

```c
"that variable is an int"
```

at compile time.

---

# `default`

If no type matches, `_Generic` uses `default`.

```c
_Generic(c,
    int: "int",
    float: "float",
    default: "something else"
)
```

If `c` is a `char`, the result is:

```text
something else
```

If there is **no matching type and no `default`**, the compiler produces an error.

---

# `_Generic` Is Compile-Time

This distinction is extremely important.

`_Generic` does **not** do something like:

```text
run program
    ↓
inspect variable
    ↓
determine type
    ↓
choose branch
```

Instead:

```text
C source
   ↓
compiler examines type
   ↓
selects matching expression
   ↓
machine code
```

There is normally **no runtime type-checking operation** corresponding to `_Generic`.

---

# `_Generic` with Macros

Because writing `_Generic` repeatedly is inconvenient, it is commonly placed inside a macro.

Example:

```c
#define TYPESTR(x) _Generic((x), \
    int: "int", \
    long: "long", \
    float: "float", \
    double: "double", \
    default: "something else")
```

Then:

```c
int i;
long l;
float f;
double d;
char c;

printf("i is type %s\n", TYPESTR(i));
printf("l is type %s\n", TYPESTR(l));
printf("f is type %s\n", TYPESTR(f));
printf("d is type %s\n", TYPESTR(d));
printf("c is type %s\n", TYPESTR(c));
```

Output:

```text
i is type int
l is type long
f is type float
d is type double
c is type something else
```

Conceptually, the compiler sees the selected expressions rather than performing runtime type detection.

---

# `_Generic` + Macros + `#x`

A more advanced use is building macros that print both the **variable name** and its value.

```c
#define FMTSPEC(x) _Generic((x), \
    int: "%d", \
    long: "%ld", \
    float: "%f", \
    double: "%f", \
    char *: "%s")

#define PRINT_VAL(x) do { \
    char fmt[512]; \
    snprintf(fmt, sizeof fmt, #x " = %s\n", FMTSPEC(x)); \
    printf(fmt, (x)); \
} while (0)
```

Then:

```c
int i = 10;
float f = 3.14159;
char *s = "Hello, world!";

PRINT_VAL(i);
PRINT_VAL(f);
PRINT_VAL(s);
```

Produces:

```text
i = 10
f = 3.141590
s = Hello, world!
```

---

# The `#x` Macro Operator

This part is especially important:

```c
#x
```

Inside a macro, `#` converts the argument into a **string literal**.

For example:

```c
#define NAME(x) #x
```

Then:

```c
printf("%s\n", NAME(foo));
```

acts conceptually like:

```c
printf("%s\n", "foo");
```

Therefore:

```c
PRINT_VAL(i);
```

can obtain:

```text
i
```

from:

```c
#x
```

while `_Generic` determines the appropriate format specifier.

---

# Compound Literals vs `_Generic`

These are separate features but both are useful for understanding advanced C.

|Feature|Purpose|When|
|---|---|---|
|Compound literal|Creates an unnamed object|Runtime object/lifetime|
|`_Generic`|Selects expression based on type|Compile time|
|`#x`|Converts macro argument to string|Preprocessing|
|Macro|Generates/reuses source code|Preprocessing|

---

# Reverse Engineering Perspective

These features can show up differently in compiled code.

### Compound literal

```c
foo(&(struct coord){10, 20});
```

May result in temporary storage:

```asm
mov     DWORD PTR [...], 10
mov     DWORD PTR [...], 20
lea     rdi, [...]
call    foo
```

Look for:

- stack-based temporary objects
    
- `lea` taking the address of temporary storage
    
- consecutive stores initializing structures/arrays
    
- function calls receiving pointers to those objects
    

### `_Generic`

Because `_Generic` is resolved during compilation, it generally **does not create runtime type-checking code**.

For example:

```c
PRINT_VAL(i);
```

may ultimately become ordinary code equivalent to:

```c
printf("i = %d\n", i);
```

The type-selection machinery has disappeared by the time you analyze the machine code.

---

# Mental Model

```text
COMPOUND LITERAL

(type){initializer}
       │
       ▼
 unnamed object
       │
       ├── use directly
       ├── take address
       └── pass to function
```

```text
_GENERIC

expression
    │
    ▼
 compiler examines type
    │
    ├── int    → expression A
    ├── float  → expression B
    ├── double → expression C
    └── default → expression D
    │
    ▼
 selected expression
    │
    ▼
 machine code
```

# Long-Term Takeaways

- **Compound literals create unnamed objects.**
    
- Syntax: `(type){initializer}`.
    
- They work with arrays, `struct`s, scalar types, etc.
    
- You can take their address with `&`.
    
- Their lifetime follows their scope; **don't return pointers to block-scope compound literals**.
    
- `_Generic` performs **compile-time type selection**.
    
- `_Generic` is not runtime type inspection.
    
- `_Generic` is particularly useful inside macros.
    
- `#x` converts a macro argument into a string literal.
    
- In RE, compound literals can appear as **temporary stack objects**, while `_Generic` usually leaves **no obvious runtime trace** because the compiler resolves it beforehand.