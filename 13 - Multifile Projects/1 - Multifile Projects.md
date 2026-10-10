# Multifile Projects in C

Real C programs are rarely written in a single `.c` file.

Instead, a program is commonly split into multiple source files:

```text
project/
├── foo.c
├── bar.c
└── bar.h
```

For example:

```text
foo.c → contains main()
bar.c → contains add()
bar.h → tells other files that add() exists
```

This allows large programs to be divided into manageable pieces.

---

# Why Split a Program Into Multiple Files?

Imagine a large program containing:

```text
100,000 lines of C code
```

Putting everything into:

```text
main.c
```

would be difficult to:

- Read
    
- Debug
    
- Maintain
    
- Reuse
    
- Compile efficiently
    

Instead, functionality can be separated:

```text
main.c       → program entry point
network.c    → networking
crypto.c     → cryptography
parser.c     → parsing
utils.c      → utility functions
```

Each `.c` file can contain related functionality.

---

# Source Files vs Header Files

There are two important types of files:

### Source file

```text
.c
```

Usually contains:

- Function definitions
    
- Global variable definitions
    
- Actual implementation
    

Example:

```c
// bar.c

int add(int a, int b)
{
    return a + b;
}
```

### Header file

```text
.h
```

Usually contains:

- Function prototypes
    
- Type declarations
    
- Macros
    
- `struct` declarations
    
- `extern` declarations
    

Example:

```c
// bar.h

int add(int, int);
```

A useful mental model:

```text
bar.h
  │
  │ tells other files WHAT exists
  ▼
bar.c
  │
  │ contains HOW it works
  ▼
add()
```

---

# Function Prototypes

Suppose `add()` is implemented in `bar.c`:

```c
// bar.c

int add(int a, int b)
{
    return a + b;
}
```

And we want to call it from `foo.c`:

```c
// foo.c

#include <stdio.h>

int main(void)
{
    printf("%d\n", add(2, 3));
}
```

The compiler does not automatically know what `add()` looks like.

We need to provide a **function prototype**:

```c
int add(int, int);
```

So:

```c
// foo.c

#include <stdio.h>

int add(int, int);

int main(void)
{
    printf("%d\n", add(2, 3));
}
```

The prototype tells the compiler:

```text
There is a function called add
It returns int
It accepts two int arguments
```

It does **not** contain the implementation.

---

# Prototype vs Function Definition

These are different:

### Prototype

```c
int add(int, int);
```

Means:

> "A function called `add` exists with this interface."

### Definition

```c
int add(int a, int b)
{
    return a + b;
}
```

Means:

> "Here is the actual implementation of `add`."

Think:

```text
Prototype
   ↓
WHAT the function looks like

Definition
   ↓
HOW the function works
```

---

# Header Files

Instead of manually writing:

```c
int add(int, int);
```

in every file that needs `add()`, we can put it in a header:

```c
// bar.h

int add(int, int);
```

Then include it:

```c
// foo.c

#include <stdio.h>
#include "bar.h"

int main(void)
{
    printf("%d\n", add(2, 3));
}
```

This is much better for larger projects.

If the function prototype changes, we only need to update:

```text
bar.h
```

instead of changing it in multiple `.c` files.

---

# `#include "bar.h"` vs `#include <stdio.h>`

You will commonly see two forms:

```c
#include <stdio.h>
```

and:

```c
#include "bar.h"
```

### Angle brackets

```c
#include <stdio.h>
```

Normally used for system/library headers.

### Double quotes

```c
#include "bar.h"
```

Normally used for your own project headers.

The exact search rules are compiler-dependent, but the important practical distinction is:

```text
<...>  → system/library headers
"..."  → project/local headers
```

---

# What Does `#include` Actually Do?

This is extremely important.

`#include` does **not** mean:

> "Link this file."

Instead, the preprocessor essentially **inserts the contents of the header into the source file** before compilation.

For example:

```c
#include "bar.h"

int main(void)
{
    printf("%d\n", add(2, 3));
}
```

If `bar.h` contains:

```c
int add(int, int);
```

the preprocessor effectively produces something conceptually like:

```c
int add(int, int);

int main(void)
{
    printf("%d\n", add(2, 3));
}
```

So:

```text
#include
   ↓
Preprocessor
   ↓
Header contents inserted
   ↓
Compiler
```

---

# Include Guards

A header can accidentally be included multiple times.

For example:

```text
main.c
 │
 ├── foo.h
 │      │
 │      └── bar.h
 │
 └── bar.h
```

Now `bar.h` gets included twice.

This can cause problems, especially when the header contains declarations such as:

```c
struct Person {
    int age;
};
```

We can prevent repeated inclusion using an **include guard**.

```c
#ifndef BAR_H
#define BAR_H

int add(int, int);

#endif
```

---

# Understanding `#ifndef`

`#ifndef` means:

```text
IF NOT DEFINED
```

So:

```c
#ifndef BAR_H
```

means:

> "If `BAR_H` has NOT been defined..."

---

# Understanding `#define`

```c
#define BAR_H
```

creates a preprocessor macro called:

```text
BAR_H
```

It doesn't need a value.

We are using it simply as a **flag**:

```text
BAR_H = "this header has already been processed"
```

---

# How the Include Guard Works

First time `bar.h` is included:

```c
#ifndef BAR_H
```

Is `BAR_H` defined?

```text
NO
```

Therefore the contents are processed:

```c
#define BAR_H

int add(int, int);
```

Now:

```text
BAR_H = defined
```

If `bar.h` is included again:

```c
#ifndef BAR_H
```

Is `BAR_H` undefined?

```text
NO
```

Therefore everything until:

```c
#endif
```

is skipped.

Conceptually:

```text
First include
     │
     ▼
BAR_H defined?
     │
    NO
     │
     ▼
Process header
     │
     ▼
#define BAR_H
```

Second include:

```text
BAR_H defined?
     │
    YES
     │
     ▼
Skip header
```

---

# Include Guard Mental Model

Think of `BAR_H` as a **security checkpoint**:

```text
Has bar.h already entered the program?

        │
   ┌────┴────┐
   │         │
  NO        YES
   │         │
   ▼         ▼
Allow      Reject/
entry      skip
   │
   ▼
Mark BAR_H
```

The entire pattern:

```c
#ifndef BAR_H
#define BAR_H

// header contents

#endif
```

is called an:

> **Include guard**

---

# Static and Extern

C programs can have variables and functions that exist in different source files.

Two important keywords for controlling this are:

```c
extern
static
```

They are particularly important for understanding **linking and reverse engineering**.

---

# `extern`

`extern` tells the compiler:

> "This variable/function exists somewhere else."

Example:

```c
// bar.c

int counter = 100;
```

Another file:

```c
// foo.c

extern int counter;

int main(void)
{
    printf("%d\n", counter);
}
```

The `extern` declaration does **not create another `counter`**.

It tells the compiler:

```text
counter exists,
but its definition is somewhere else.
```

Conceptually:

```text
foo.c                         bar.c

extern int counter;           int counter = 100;
       │                              │
       └───────────┐    ┌─────────────┘
                   ▼    ▼
                linker
                   │
                   ▼
             one counter
```

---

# `static` at File Scope

`static` can restrict visibility to the current source file.

Example:

```c
// secret.c

static int secret = 123;
```

Another source file cannot access that variable through `extern`.

```c
extern int secret;
```

doesn't make it externally visible.

This is called **internal linkage**.

Without `static`, a file-scope object/function can potentially have **external linkage**.

---

# Why `static` Matters in Reverse Engineering

When analyzing binaries, symbol visibility can give you clues.

For example:

```c
static void helper(void)
{
    ...
}
```

is intended to be private to that source file.

Whereas:

```c
void helper(void)
{
    ...
}
```

can have external linkage.

So `static` affects how a symbol can be referenced across translation units and how it appears to the linker.

---

# Compilation Process

A `.c` file doesn't directly become an executable.

There are several stages:

```text
foo.c
 │
 ▼
Preprocessor
 │
 ▼
Expanded source
 │
 ▼
Compiler
 │
 ▼
Assembly
 │
 ▼
Assembler
 │
 ▼
foo.o
 │
 └──────────────┐
                │
bar.c           │
 │              │
 ▼              │
bar.o ──────────┤
                ▼
             Linker
                │
                ▼
            Executable
```

The important stages are:

```text
Preprocessing
      ↓
Compilation
      ↓
Assembly
      ↓
Linking
```

---

# Compiling Multiple Source Files Directly

Suppose we have:

```text
foo.c
bar.c
```

We can compile and link them together:

```bash
gcc -o foo foo.c bar.c
```

This tells GCC:

```text
Create executable → foo

using:
foo.c
bar.c
```

The linker combines the resulting pieces into one executable.

---

# Object Files

Instead of producing the executable immediately, we can compile each source file into an **object file**.

Use:

```bash
gcc -c foo.c
```

This produces:

```text
foo.o
```

And:

```bash
gcc -c bar.c
```

produces:

```text
bar.o
```

Now we have:

```text
foo.c → foo.o
bar.c → bar.o
```

The `.o` files contain compiled machine-code-related information, but they aren't normally complete standalone executables.

---

# Linking Object Files

Now we can combine them:

```bash
gcc -o foo foo.o bar.o
```

Conceptually:

```text
foo.o ─────┐
           │
           ▼
         Linker
           │
bar.o ─────┘
           │
           ▼
        foo executable
```

---

# Why Does the Linker Matter?

Consider:

```c
// foo.c

int add(int, int);

int main(void)
{
    return add(2, 3);
}
```

The compiler sees:

```c
int add(int, int);
```

So it knows:

```text
add exists.
```

But `foo.c` doesn't contain the actual implementation.

The implementation is in:

```c
// bar.c

int add(int a, int b)
{
    return a + b;
}
```

After compilation:

```text
foo.o
  │
  └── references add()

bar.o
  │
  └── defines add()
```

The **linker** connects them.

```text
foo.o
  │
  │ "I need add()"
  ▼
LINKER
  ▲
  │ "bar.o provides add()"
  │
bar.o
```

This is one of the most important concepts for understanding C binaries.

---

# Undefined vs Defined Symbols

This leads to an important reverse-engineering concept.

`foo.o` may contain a reference to:

```text
add
```

but not contain the actual implementation.

So conceptually:

```text
foo.o
└── undefined reference: add

bar.o
└── defined symbol: add
```

The linker resolves the reference:

```text
undefined add
       │
       ▼
defined add
       │
       ▼
resolved executable
```

This relationship becomes very useful when examining:

- ELF binaries
    
- PE files
    
- Symbol tables
    
- Imports
    
- Exports
    
- Relocations
    
- Linker errors
    

---

# Why Object Files Are Useful

Suppose you have:

```text
1000 C source files
```

Compiling everything from scratch can take a long time.

But imagine changing only:

```text
network.c
```

There is no reason to recompile:

```text
crypto.c
parser.c
database.c
...
```

if they haven't changed.

Instead:

```text
network.c
   ↓
network.o
   ↓
link
```

The existing object files can be reused.

Conceptually:

```text
Before:

a.c → a.o ─┐
b.c → b.o ─┤
c.c → c.o ─┤
d.c → d.o ─┤ → executable
e.c → e.o ─┘


Change only c.c:

a.o ─┐
b.o ─┤
c.c → c.o ─┤ → executable
d.o ─┤
e.o ─┘
```

Only `c.c` needs recompilation.

---

# `make`

The `make` utility automates this process.

It tracks dependencies and timestamps.

For example:

```text
a.c → a.o
b.c → b.o
c.c → c.o
```

If only:

```text
b.c
```

changes:

```text
a.o  → reuse
b.o  → rebuild
c.o  → reuse
```

Then the linker creates a new executable.

This can dramatically reduce build times for large projects.

---

# Important Reverse Engineering Connection

When you reverse engineer a compiled program, remember:

```text
Original C project

foo.c ──┐
bar.c ──┤
util.c ─┤
main.c ─┘
    │
    ▼
Compiler + Assembler
    │
    ▼
Object files
    │
    ▼
Linker
    │
    ▼
Executable
```

The final executable usually **doesn't preserve the original `.c` file structure in an obvious way**.

You may instead see:

```text
functions
symbols
addresses
sections
imports
exports
strings
machine instructions
```

Your job as a reverse engineer is often to reconstruct the original organization and intent.

---

# C Source → Binary Mental Model

When writing C:

```text
foo.c
   │
   ├── variables
   ├── functions
   └── prototypes
          │
          ▼
     Preprocessor
          │
          ▼
    Translation unit
          │
          ▼
       Compiler
          │
          ▼
       foo.o
          │
          ▼
        Linker
          │
          ▼
     Executable
```

When reverse engineering:

```text
Executable
    │
    ▼
Machine instructions
    │
    ├── functions
    ├── symbols
    ├── memory accesses
    ├── calls
    └── references
    │
    ▼
Decompiler / Disassembler
    │
    ▼
Reconstruct likely C structure
```

---

# Key Takeaways

Remember these relationships:

|Concept|Meaning|
|---|---|
|`.c`|Source/implementation file|
|`.h`|Header containing declarations/interfaces|
|Prototype|Tells compiler a function's signature|
|Definition|Contains the actual implementation|
|`#include`|Preprocessor inserts header contents|
|`#define`|Defines a preprocessor macro|
|`#ifndef`|"If this isn't defined"|
|Include guard|Prevents a header from being processed multiple times|
|`extern`|Refers to something defined elsewhere|
|`static`|Can restrict file-scope visibility to the current source file|
|`.o`|Object file produced before final linking|
|Compiler|Converts C into lower-level code/object code|
|Linker|Combines object files and resolves references|
|`make`|Automates incremental rebuilding|

### The most important mental model

```text
HEADER (.h)
    │
    │ declarations
    ▼
SOURCE (.c)
    │
    │ implementation
    ▼
OBJECT (.o)
    │
    │ symbols + machine code + unresolved references
    ▼
LINKER
    │
    │ resolves references
    ▼
EXECUTABLE
```

And from a **reverse-engineering perspective**:

```text
C source
  ↓
Compiler
  ↓
Object files
  ↓
Linker
  ↓
Binary
  ↓
Machine code + symbols + data
  ↓
YOU reconstruct the original logic
```

