
An **enumeration** (`enum`) is an integer type that lets you give meaningful names to integer values.

Example:

```c
enum {
    ONE = 1,
    TWO = 2
};

printf("%d %d\n", ONE, TWO);
```

Output:

```text
1 2
```

Instead of writing:

```c
int resource = 3;
```

you can write:

```c
enum resource_type resource = BRICK;
```

This makes the code easier to understand.

---

# Why Use `enum`?

Enums improve:

- Readability
    
- Code clarity
    
- Maintainability
    
- Debugging
    

Example:

```c
if (resource == 3)
```

vs:

```c
if (resource == BRICK)
```

The second version clearly shows what `3` means.

---

# `enum` vs `#define`

Example with macros:

```c
#define SHEEP 0
#define WHEAT 1
#define WOOD  2
```

Example with an enum:

```c
enum {
    SHEEP,
    WHEAT,
    WOOD
};
```

Key difference:

```text
#define
    ↓
Preprocessor substitution

enum
    ↓
Integer constants belonging to the C language
```

---

# Important Differences

`enum`:

- Represents integer values
    
- Creates named integer constants
    
- Can define an enum type
    
- Symbolic values may be displayed meaningfully by debuggers
    

`#define`:

- Is handled by the preprocessor
    
- Can substitute almost anything
    
- Does not create a C type
    
- Usually disappears before compilation
    

---

# Basic `enum` Syntax

```c
enum {
    ONE,
    TWO,
    THREE
};
```

By default, the values are:

```text
ONE   = 0
TWO   = 1
THREE = 2
```

You can use them anywhere appropriate for integer values.

Example:

```c
int array[THREE];
```

Or:

```c
switch (value)
{
    case ONE:
        break;

    case TWO:
        break;
}
```

---

# Automatic Numbering

Enum constants are automatically numbered.

Example:

```c
enum {
    SHEEP,
    WHEAT,
    WOOD,
    BRICK,
    ORE
};
```

Values:

```text
SHEEP = 0
WHEAT = 1
WOOD  = 2
BRICK = 3
ORE   = 4
```

Example:

```c
printf("%d %d\n", SHEEP, BRICK);
```

Output:

```text
0 3
```

---

# Setting Explicit Values

You can manually assign values.

Example:

```c
enum {
    X = 2,
    Y = 18,
    Z = -2
};
```

Values:

```text
X = 2
Y = 18
Z = -2
```

Enum values are not required to start at `0`.

---

# Duplicate Values

Multiple enum constants can have the same value.

Example:

```c
enum {
    X = 2,
    Y = 2,
    Z = 2
};
```

This is valid.

Conceptually:

```text
X ──┐
Y ──┼──► 2
Z ──┘
```

The names are different even though their values are the same.

---

# Continuing Automatic Numbering

If you explicitly set a value, automatic numbering continues from that value.

Example:

```c
enum {
    A,      // 0
    B,      // 1
    C = 4,  // 4
    D,      // 5
    E,      // 6
    F = 3,  // 3
    G,      // 4
    H       // 5
};
```

Mental model:

```text
No value specified?
        │
        ▼
Previous value + 1
```

The first enumerator defaults to `0`.

---

# Trailing Commas

A trailing comma is allowed.

Example:

```c
enum {
    X = 2,
    Y = 18,
    Z = -2,
};
```

This is valid C and can make adding new values easier.

---

# Scope

Enum constants follow normal C scope rules.

Example:

```c
void func(void)
{
    enum {
        RED,
        GREEN,
        BLUE
    };

    printf("%d\n", RED);
}
```

Here, the enum constants are visible only within the appropriate block scope.

---

# File Scope

An enum declared outside functions has file scope.

Example:

```c
enum {
    DEBUG,
    RELEASE
};

int main(void)
{
    printf("%d\n", DEBUG);
}
```

These names can be used throughout the rest of that translation unit after their declaration.

Enums are commonly placed in header files when multiple source files need the same definitions.

---

# Your `enum` Is a Type

An enum can have a **tag**, allowing you to refer to its type later.

Example:

```c
enum resource {
    SHEEP,
    WHEAT,
    WOOD,
    BRICK,
    ORE
};
```

The type is:

```text
enum resource
```

You can declare variables:

```c
enum resource r = BRICK;
```

---

# Using an Enum Variable

Example:

```c
enum resource {
    SHEEP,
    WHEAT,
    WOOD,
    BRICK,
    ORE
};

enum resource r = BRICK;

if (r == BRICK)
{
    printf("I'll trade you a brick.\n");
}
```

The enum makes the intended meaning clearer.

---

# Why Not Just Use `int`?

You could write:

```c
int r = 3;
```

But:

```text
What does 3 mean?
```

Compare:

```c
enum resource r = BRICK;
```

Mental model:

```text
int
 │
 └── Generic number


enum resource
 │
 └── Number representing a resource
```

The main advantage in C is **expressing intent clearly**.

---

# Important: C Does Not Strictly Enforce Enum Values

Unlike what beginners sometimes expect, C does not generally guarantee that an enum variable can only contain one of its listed enumerators.

Example:

```c
enum resource r;

r = 100;
```

Code may compile.

Therefore:

```text
enum
≠
automatic runtime range checking
```

Enums primarily provide a named type and meaningful constants.

---

# `typedef` with `enum`

You can use `typedef` to create a shorter type name.

Example:

```c
typedef enum {
    SHEEP,
    WHEAT,
    WOOD,
    BRICK,
    ORE
} RESOURCE;
```

Now:

```c
RESOURCE r = BRICK;
```

instead of:

```c
enum resource r = BRICK;
```

---

# Tagged `enum` vs `typedef`

Tagged enum:

```c
enum resource {
    SHEEP,
    WHEAT,
    WOOD,
    BRICK,
    ORE
};

enum resource r;
```

With `typedef`:

```c
typedef enum {
    SHEEP,
    WHEAT,
    WOOD,
    BRICK,
    ORE
} RESOURCE;

RESOURCE r;
```

Both approaches are valid.

---

# Declaring Variables with an `enum`

You can declare variables at the same time as the enum.

Example:

```c
enum {
    SHEEP,
    WHEAT,
    WOOD,
    BRICK,
    ORE
} r = BRICK, s = WOOD;
```

Here:

```text
Enum type
    │
    ├── r = BRICK
    └── s = WOOD
```

This is legal, but usually less readable than defining the type separately.

---

# Named `enum` with Variables

Example:

```c
enum resource {
    SHEEP,
    WHEAT,
    WOOD,
    BRICK,
    ORE
} r = BRICK, s = WOOD;
```

Now:

```text
Type:
enum resource

Variables:
r
s
```

The enum type can also be used later:

```c
enum resource x = SHEEP;
```

---

# `enum` Values Are Integer Constants

Enum constants can be used where integer constant expressions are allowed.

Example:

```c
enum {
    BUFFER_SIZE = 1024
};

char buffer[BUFFER_SIZE];
```

They can also be used in:

```c
case labels
```

Example:

```c
switch (r)
{
    case SHEEP:
        printf("Sheep\n");
        break;

    case BRICK:
        printf("Brick\n");
        break;
}
```

---

# Reverse Engineering Perspective

C source:

```c
enum resource {
    SHEEP,
    WHEAT,
    WOOD,
    BRICK,
    ORE
};

enum resource r = BRICK;
```

At the machine-code level, the CPU generally sees an integer value.

Conceptually:

```text
C Source

BRICK
  │
  ▼
Integer Value
  │
  ▼
Machine Code
```

If:

```text
BRICK = 3
```

assembly might effectively compare against:

```asm
cmp eax, 3
```

The CPU does not know the symbolic name:

```text
BRICK
```

---

# Enums in a Debugger

Debug information can preserve the relationship between:

```text
3
│
▼
BRICK
```

So a debugger with suitable symbols may display:

```text
r = BRICK
```

instead of:

```text
r = 3
```

This makes enums especially useful when debugging.

Without debug information, reverse engineers may only see the raw value and need to infer its meaning.

---

# `enum` in Reverse Engineering

Suppose you see:

```asm
cmp eax, 3
je  some_function
```

You might eventually reconstruct:

```c
enum command {
    START = 0,
    STOP = 1,
    PAUSE = 2,
    EXIT = 3
};
```

Then the assembly:

```asm
cmp eax, 3
```

may correspond to:

```c
if (command == EXIT)
```

Recognizing repeated comparisons against a small set of constants can help identify possible enum values.

---

# `switch` and Enums

Enums are commonly used with `switch`.

C:

```c
switch (r)
{
    case SHEEP:
        break;

    case WHEAT:
        break;

    case WOOD:
        break;
}
```

Assembly may contain:

```text
Compare value
    │
    ├── Case 0
    ├── Case 1
    ├── Case 2
    └── ...
```

Or, depending on optimization and the values involved, a jump table.

This is useful when reverse engineering state machines and command handlers.

---

# `enum` vs Macro in Reverse Engineering

Macro:

```c
#define BRICK 3
```

Enum:

```c
enum {
    BRICK = 3
};
```

Both can ultimately result in:

```asm
cmp eax, 3
```

So in a stripped binary:

```text
Source distinction
        ↓
often lost
        ↓
Raw integer constant
```

Debug symbols can provide additional information about enum types and names.

---

# Mental Model

```text
Meaning
   │
   ▼
Symbolic Name

BRICK
   │
   ▼
Integer Value

3
   │
   ▼
CPU / Machine Code
```

An enum connects meaningful source-level names to integer values.

---

# Long-Term Takeaways

- An `enum` creates named integer constants.
    
- Enumerators start at `0` by default.
    
- Values automatically increase by `1`.
    
- You can explicitly assign positive or negative values.
    
- Multiple enumerators can have the same value.
    
- Automatic numbering continues from the previously assigned value.
    
- A trailing comma after the final enumerator is valid.
    
- Enum constants follow normal C scope rules.
    
- A tagged enum creates a type such as `enum resource`.
    
- `typedef` can create a shorter enum type name.
    
- Enum values can be used where integer constant expressions are allowed, including `case` labels.
    
- In C, enum variables are not generally strict runtime range restrictions.
    
- Enums provide better code clarity than unexplained integer values.
    
- Compared with `#define`, enums are part of the C language rather than simple preprocessor substitution.
    
- Debuggers with symbols can often display enum values using their symbolic names.
    
- At the machine-code level, enum values are generally represented as integers.
    
- The CPU does not know enum names such as `BRICK`; it only sees their numeric values.
    
- When reverse engineering, repeated comparisons against meaningful integer constants may represent enum values.