
The **C preprocessor** is a program that runs **before the compiler**.

It processes special instructions called **preprocessor directives**, which begin with:

```c
#
```

Examples:

```c
#include <stdio.h>
#define MAX 100
#ifdef DEBUG
#endif
```

The preprocessor mainly performs:

- Header inclusion
    
- Macro expansion
    
- Conditional compilation
    
- Source-code modification
    

---

# Compilation Pipeline

A simplified C compilation process is:

```text
C Source Code
     │
     ▼
Preprocessor
     │
     ▼
Expanded C Code
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
Object File
     │
     ▼
Linker
     │
     ▼
Executable
```

For example:

```c
#include <stdio.h>

#define MAX 100

int main(void)
{
    printf("%d\n", MAX);
}
```

The preprocessor expands the source before the compiler sees it.

---

# `#include`

`#include` tells the preprocessor to **insert the contents of another file** into the current source file.

Example:

```c
#include <stdio.h>
```

Conceptually:

```text
main.c
   │
   ├── includes stdio.h
   │
   ▼
expanded source
```

The compiler then compiles the combined result.

---

# System Headers

Angle brackets are normally used for system headers:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
```

The compiler searches its configured system include directories.

---

# User Headers

Double quotes are normally used for your own headers:

```c
#include "config.h"
```

The compiler generally searches the source file's directory first.

Example:

```text
project/
│
├── main.c
├── functions.c
└── functions.h
```

Then:

```c
#include "functions.h"
```

---

# Reverse Engineering Perspective

`#include` itself does **not** exist in the final executable.

It is a **preprocessor feature**.

For example:

```c
#include <stdio.h>
```

does not become an `#include` instruction in assembly.

Instead, declarations from the header are made available to the compiler.

The resulting executable contains machine code and data, not C preprocessor directives.

---

# Simple Macros

A macro is a name that the preprocessor replaces with something else.

Syntax:

```c
#define NAME replacement
```

Example:

```c
#define MAX 100
```

Code:

```c
int x = MAX;
```

After preprocessing, conceptually:

```c
int x = 100;
```

---

# Macros Are Text Substitution

Macros are primarily **preprocessor substitutions**, not variables.

Example:

```c
#define PI 3.14159
```

This does not create a variable called `PI`.

The preprocessor replaces occurrences of `PI` before compilation.

---

# Macro Constants

Example:

```c
#define BUFFER_SIZE 1024

char buffer[BUFFER_SIZE];
```

Conceptually becomes:

```c
char buffer[1024];
```

---

# Macro vs `const`

Macro:

```c
#define MAX 100
```

`const` variable:

```c
const int max = 100;
```

Important difference:

```text
#define
    ↓
Preprocessor substitution

const
    ↓
Actual C object with a type
```

For ordinary typed constants, `const` is often preferable.

---

# Conditional Compilation

Conditional compilation allows sections of source code to be included or excluded **before compilation**.

Example:

```c
#ifdef DEBUG

printf("Debug mode\n");

#endif
```

If `DEBUG` is defined, the code is included.

If it isn't defined, the code is removed during preprocessing.

---

# `#ifdef`

`#ifdef` means:

> Compile this code if this macro is defined.

Example:

```c
#define DEBUG

#ifdef DEBUG
printf("Debugging enabled\n");
#endif
```

Because `DEBUG` exists, the `printf()` remains after preprocessing.

---

# `#endif`

Every conditional block must eventually be closed.

Example:

```c
#ifdef DEBUG

printf("Debug");

#endif
```

Think of:

```text
#ifdef
   │
   ├── code
   │
#endif
```

as:

> Start conditional section → end conditional section.

---

# `#ifndef`

`#ifndef` means:

> If this macro is **not defined**.

Example:

```c
#ifndef MAX
#define MAX 100
#endif
```

Meaning:

```text
Is MAX undefined?
       │
      YES
       │
       ▼
#define MAX 100
```

---

# Header Guards

One of the most important uses of `#ifndef` is preventing a header file from being included multiple times.

Example:

```c
#ifndef FUNCTIONS_H
#define FUNCTIONS_H

int add(int a, int b);
void print_message(void);

#endif
```

The first time the header is included:

```text
FUNCTIONS_H undefined
        ↓
define FUNCTIONS_H
        ↓
include contents
```

If it is included again:

```text
FUNCTIONS_H already defined
        ↓
skip contents
```

---

# Why Header Guards Exist

Suppose:

```c
#include "functions.h"
#include "functions.h"
```

Without protection, declarations or definitions in the header could be processed multiple times.

Header guards prevent this.

---

# `#else`

`#else` provides an alternative section.

Example:

```c
#ifdef DEBUG

printf("Debug mode");

#else

printf("Release mode");

#endif
```

Conceptually:

```text
DEBUG defined?
      │
   ┌──┴──┐
  YES    NO
   │      │
Debug   Release
```

---

# `#elifdef`

`#elifdef` means:

> Else, if this macro is defined.

Example:

```c
#ifdef WINDOWS

printf("Windows");

#elifdef LINUX

printf("Linux");

#else

printf("Unknown OS");

#endif
```

This allows multiple definition checks.

---

# `#elifndef`

`#elifndef` means:

> Else, if this macro is not defined.

Example:

```c
#ifdef DEBUG

printf("Debug");

#elifndef RELEASE

printf("Not release");

#endif
```

These directives are available in modern C standards/implementations; older C code often uses equivalent `#elif defined(...)` forms.

---

# General Conditional: `#if`

`#if` evaluates a **preprocessor expression**.

Example:

```c
#define VERSION 3

#if VERSION >= 3

printf("Version 3 or newer");

#endif
```

The preprocessor evaluates the condition.

---

# `#elif`

`#elif` means:

> Else, if this condition is true.

Example:

```c
#if VERSION == 1

printf("Version 1");

#elif VERSION == 2

printf("Version 2");

#else

printf("Other version");

#endif
```

---

# `defined()`

The `defined()` operator checks whether a macro exists.

Example:

```c
#if defined(DEBUG)

printf("Debug");

#endif
```

This is similar to:

```c
#ifdef DEBUG
```

You can also combine conditions:

```c
#if defined(DEBUG) && defined(WINDOWS)

printf("Windows debug build");

#endif
```

---

# Losing a Macro: `#undef`

`#undef` removes a macro definition.

Example:

```c
#define SIZE 100

#undef SIZE
```

After:

```c
#undef SIZE
```

`SIZE` is no longer defined.

You can redefine it:

```c
#define SIZE 100

#undef SIZE

#define SIZE 200
```

---

# Built-in Macros

The compiler automatically provides certain predefined macros.

Example:

```c
printf("%s\n", __FILE__);
```

These are useful for:

- Debugging
    
- Logging
    
- Conditional compilation
    
- Compiler/platform detection
    
- Diagnostics
    

---

# Mandatory Macros

Common predefined macros include:

```c
__FILE__
__LINE__
__DATE__
__TIME__
__STDC__
```

Example:

```c
printf("File: %s\n", __FILE__);
printf("Line: %d\n", __LINE__);
```

Possible output:

```text
File: main.c
Line: 10
```

---

# `__FILE__`

Contains the name of the current source file.

Example:

```c
printf("File: %s\n", __FILE__);
```

Useful for debugging.

---

# `__LINE__`

Contains the current source-code line number.

Example:

```c
printf("Line: %d\n", __LINE__);
```

This is especially useful in debugging macros.

---

# `__DATE__`

Contains the compilation date as a string.

Example:

```c
printf("Built: %s\n", __DATE__);
```

---

# `__TIME__`

Contains the compilation time as a string.

Example:

```c
printf("Built at: %s\n", __TIME__);
```

---

# Optional / Implementation-Defined Macros

Compilers can provide additional predefined macros.

Common examples include:

```c
_WIN32
__linux__
__GNUC__
__clang__
```

These can be used for platform/compiler-specific code.

Example:

```c
#ifdef _WIN32

printf("Windows");

#elif defined(__linux__)

printf("Linux");

#endif
```

---

# Macros with Arguments

Macros can accept arguments.

Syntax:

```c
#define NAME(parameter) replacement
```

Example:

```c
#define SQUARE(x) ((x) * (x))
```

Usage:

```c
int x = SQUARE(5);
```

Conceptually becomes:

```c
int x = ((5) * (5));
```

---

# Why Parentheses Matter

Bad macro:

```c
#define SQUARE(x) x * x
```

Consider:

```c
SQUARE(2 + 3)
```

Expansion:

```c
2 + 3 * 2 + 3
```

Because of operator precedence, this does **not** produce `25`.

Better:

```c
#define SQUARE(x) ((x) * (x))
```

Now:

```c
SQUARE(2 + 3)
```

becomes:

```c
((2 + 3) * (2 + 3))
```

Result:

```text
25
```

---

# Macro Arguments Are Not Function Arguments

Consider:

```c
#define SQUARE(x) ((x) * (x))
```

and:

```c
int square(int x)
{
    return x * x;
}
```

They look similar but behave differently.

A function:

```text
argument
   ↓
evaluated
   ↓
passed to function
```

A macro:

```text
argument
   ↓
textually substituted
   ↓
expanded
```

---

# Macro Side Effects

This is a major danger.

Example:

```c
#define SQUARE(x) ((x) * (x))
```

Now:

```c
int x = 5;

SQUARE(x++);
```

Expands approximately to:

```c
((x++) * (x++))
```

The expression modifies `x` more than once without an appropriate sequencing relationship.

This can result in **undefined behavior**.

For operations involving expressions with side effects, a real function is often safer.

---

# Macros with More Than One Argument

Example:

```c
#define MAX(a, b) ((a) > (b) ? (a) : (b))
```

Usage:

```c
int x = MAX(10, 20);
```

Conceptually:

```c
int x = ((10) > (20) ? (10) : (20));
```

---

# Macros with Variable Arguments

Macros can accept a variable number of arguments using:

```c
...
```

Example:

```c
#define LOG(fmt, ...) printf(fmt, __VA_ARGS__)
```

Usage:

```c
LOG("Value = %d\n", x);
```

Conceptually becomes something similar to:

```c
printf("Value = %d\n", x);
```

Variable-argument macros are useful for:

- Logging
    
- Debugging
    
- Assertions
    
- Wrapper macros
    

---

# Stringification

The `#` operator inside a macro converts an argument into a **string literal**.

Example:

```c
#define STRINGIFY(x) #x
```

Usage:

```c
STRINGIFY(hello)
```

Produces:

```c
"hello"
```

Example:

```c
printf("%s\n", STRINGIFY(hello));
```

Output:

```text
hello
```

---

# Stringification Example

```c
#define PRINT_NAME(x) printf("%s = %d\n", #x, x)
```

Usage:

```c
int age = 23;

PRINT_NAME(age);
```

Conceptually becomes:

```c
printf("%s = %d\n", "age", age);
```

Output:

```text
age = 23
```

---

# Concatenation

The `##` operator joins two tokens together.

Example:

```c
#define MAKE_VAR(a, b) a##b
```

Usage:

```c
int MAKE_VAR(my, age) = 23;
```

Becomes:

```c
int myage = 23;
```

---

# Why Token Concatenation Is Useful

It can generate identifiers automatically.

Example:

```c
#define REGISTER(name) register_##name
```

Then:

```c
REGISTER(foo)
```

becomes:

```c
register_foo
```

This is commonly used in:

- Generated code
    
- Registration systems
    
- Generic macros
    
- Platform-specific code
    

---

# Multiline Macros

A macro can span multiple lines using a backslash:

```c
#define PRINT_VALUES(a, b) \
    printf("A = %d\n", a); \
    printf("B = %d\n", b);
```

The `\` tells the preprocessor:

> Continue this macro onto the next line.

---

# Multiline Macro with `do while`

A safer pattern for statement-like macros is:

```c
#define PRINT_VALUES(a, b) \
    do { \
        printf("A = %d\n", a); \
        printf("B = %d\n", b); \
    } while (0)
```

Now it behaves more like a single statement.

---

# Example: An Assert Macro

Assertions are commonly implemented using macros.

Example:

```c
#define ASSERT(condition) \
    do { \
        if (!(condition)) { \
            fprintf(stderr, "Assertion failed\n"); \
            abort(); \
        } \
    } while (0)
```

Usage:

```c
int x = 10;

ASSERT(x > 0);
```

If:

```c
x > 0
```

is false, the assertion triggers.

---

# Debug Assertion Macro

A more useful debugging macro can include file and line information:

```c
#define ASSERT(condition) \
    do { \
        if (!(condition)) { \
            fprintf(stderr, \
                "Assertion failed: %s (%s:%d)\n", \
                #condition, __FILE__, __LINE__); \
            abort(); \
        } \
    } while (0)
```

Example:

```c
ASSERT(x != NULL);
```

The `#condition` converts:

```c
x != NULL
```

into:

```text
"x != NULL"
```

---

# The `#error` Directive

`#error` forces the compilation process to produce an error.

Example:

```c
#ifndef API_KEY
#error "API_KEY must be defined"
#endif
```

If `API_KEY` isn't defined, compilation stops with an error.

---

# Why Use `#error`?

Useful for detecting configuration problems:

```c
#if BUFFER_SIZE < 64
#error "BUFFER_SIZE is too small"
#endif
```

This catches invalid configurations during compilation rather than at runtime.

---

# The `#embed` Directive

`#embed` is a modern C feature for embedding binary resources directly into a C program.

Example conceptually:

```c
const unsigned char image[] = {
    #embed "image.bin"
};
```

The preprocessor obtains the contents of the specified resource and produces preprocessing tokens representing the data.

This can be useful for embedding:

- Images
    
- Fonts
    
- Firmware
    
- Binary data
    
- Other resources
    

---

# Why `#embed` Is Interesting for Reverse Engineering

It allows binary data to become part of the compiled program.

For example:

```text
image.bin
    │
    ▼
#embed
    │
    ▼
C source
    │
    ▼
compiler
    │
    ▼
Executable
```

In reverse engineering, you may later encounter that embedded data inside:

- `.rodata`
    
- `.data`
    
- Resource sections
    
- Other binary sections
    

---

# `#embed` Parameters

`#embed` supports parameters that control how the embedded data is produced.

Important parameters include:

```text
limit()
if_empty
prefix()
suffix()
```

There are also implementation/compiler-specific or standardized additional options depending on the C version and implementation.

---

# The `limit()` Parameter

`limit()` limits how many bytes are embedded.

Example:

```c
#embed "file.bin" limit(16)
```

Conceptually:

```text
Embed only the first 16 bytes
```

This can be useful when only part of a resource is required.

---

# The `if_empty` Parameter

`if_empty` specifies what should happen when the embedded resource contains no data.

Conceptually:

```c
#embed "file.bin" if_empty(...)
```

This allows the source code to handle an empty resource explicitly.

---

# `prefix()` and `suffix()`

These parameters allow tokens to be inserted before or after the embedded data.

Conceptually:

```c
#embed "file.bin" prefix(...) suffix(...)
```

Think:

```text
prefix
   │
   ▼
embedded data
   │
   ▼
suffix
```

---

# The `__has_embed()` Identifier

`__has_embed()` can be used to determine whether a resource can be embedded.

Conceptually:

```c
#if __has_embed("image.bin")
    /* Resource available */
#endif
```

This allows code to conditionally use `#embed`.

---

# Other `#embed` Parameters

Depending on the C implementation and standard version, additional parameters can affect how the embedded bytes are represented.

The important idea is:

```text
#embed
   │
   ├── resource
   ├── size limits
   ├── empty-resource handling
   └── surrounding tokens
```

---

# Embedding Multi-Byte Values

Binary data can contain values larger than one byte.

When embedding binary data, the representation needs to account for the target implementation's integer types and representation.

This matters when embedding:

- Images
    
- Audio
    
- Executable data
    
- Network packets
    
- Firmware
    
- Cryptographic material
    

From a reverse-engineering perspective, embedded data may appear as a sequence of bytes in the executable rather than as recognizable C source.

---

# The `#pragma` Directive

`#pragma` provides instructions to the compiler or implementation.

Example:

```c
#pragma once
```

Unlike many standard C constructs, pragmas can be **implementation-specific**.

---

# Non-Standard Pragmas

Compilers can define their own pragmas.

Examples commonly seen in real-world code include:

```c
#pragma once
#pragma pack(push, 1)
#pragma pack(pop)
```

Their exact behavior depends on the compiler.

---

# `#pragma once`

A common compiler-supported pragma is:

```c
#pragma once
```

It tells the compiler to process the header only once per translation unit.

Example:

```c
#pragma once

int add(int a, int b);
```

It is commonly used instead of traditional header guards.

---

# Standard Pragmas

The C standard also defines certain pragmas.

Their purpose is to provide standardized compiler-control mechanisms where specified by the standard.

However, compiler support and behavior can vary depending on the particular pragma and implementation.

---

# `_Pragma` Operator

`_Pragma` allows pragma directives to be generated through macro expansion.

Example:

```c
#define DO_PRAGMA(x) _Pragma(#x)
```

Then:

```c
DO_PRAGMA(message("Building program"))
```

can generate a pragma after macro expansion.

This is useful when a pragma needs to be controlled through a macro.

---

# `#line`

`#line` changes the line number and optionally the filename that the compiler reports for subsequent source lines.

Example:

```c
#line 100 "generated.c"
```

The compiler will subsequently treat the next source line as line `100` of:

```text
generated.c
```

---

# Why `#line` Exists

It is particularly useful for:

- Generated C code
    
- Code generators
    
- Debugging generated source
    
- Compiler diagnostics
    

Example:

```text
Generated source
       │
       ▼
#line
       │
       ▼
Compiler reports original source location
```

---

# Reverse Engineering Perspective: `#line`

`#line` generally does **not** directly create runtime instructions.

Instead, it affects compiler metadata and diagnostics.

However, source-level debugging information may reflect these manipulated filenames and line numbers.

This can matter when analyzing:

- Debug builds
    
- Generated code
    
- PDB/DWARF information
    
- Compiler-generated binaries
    

---

# The Null Directive

A line containing only:

```c
#
```

is called the **null directive**.

It performs no preprocessing action.

Example:

```c
#
```

It can effectively be used as an empty preprocessor directive.

---

# Preprocessor vs Compiler

This distinction is extremely important.

### Preprocessor

Handles:

```c
#include
#define
#ifdef
#ifndef
#if
#else
#elif
#undef
#error
#embed
#pragma
#line
```

### Compiler

Handles actual C language constructs:

```c
int
if
for
while
struct
function
pointer
expression
```

---

# What Happens to Macros?

Consider:

```c
#define SIZE 10

int array[SIZE];
```

The preprocessor produces something conceptually like:

```c
int array[10];
```

The compiler then compiles:

```c
int array[10];
```

The executable does **not** contain a macro called `SIZE`.

---

# What Happens to Conditional Compilation?

Consider:

```c
#ifdef DEBUG
printf("Debug\n");
#endif
```

If `DEBUG` is not defined:

```text
Source
  │
  ▼
Preprocessor
  │
  ▼
printf removed
  │
  ▼
Compiler never sees it
```

This is fundamentally different from:

```c
if (debug)
    printf("Debug\n");
```

because an ordinary `if` is processed by the compiler and becomes runtime code.

---

# Conditional Compilation vs Runtime `if`

Preprocessor:

```c
#ifdef WINDOWS
    windows_function();
#endif
```

The code may completely disappear before compilation.

Runtime:

```c
if (windows)
    windows_function();
```

The compiler still sees both paths.

---

# Reverse Engineering Perspective

This distinction is very important.

Suppose the source contains:

```c
#ifdef DEBUG
    printf("Password = %s\n", password);
#endif
```

If the program was compiled **without** `DEBUG`, the debugging code may not exist in the binary at all.

Therefore, when reverse engineering:

```text
Source Code
    │
    ▼
Preprocessor
    │
    ├── removed code
    │
    ▼
Compiler
    │
    ▼
Binary
```

You are analyzing the **post-preprocessor program**, not the original source.

---

# Preprocessor Macros in Reverse Engineering

Macros can make source code look very different from the generated assembly.

Example:

```c
#define READ_REG(x) (*(volatile unsigned int *)(x))
```

Source:

```c
value = READ_REG(0x40000000);
```

After expansion, conceptually:

```c
value = (*(volatile unsigned int *)(0x40000000));
```

The compiler then generates the actual memory access.

So when reversing a binary, you don't see:

```text
READ_REG()
```

You see the machine instructions generated from its expansion.

---

# Macro vs Function in Reverse Engineering

Macro:

```c
#define SQUARE(x) ((x) * (x))
```

Function:

```c
int square(int x)
{
    return x * x;
}
```

A macro may be completely **inlined through preprocessing and optimization**.

A function may produce a recognizable function boundary:

```asm
square:
    ...
    ret
```

Although optimization can also inline ordinary functions.

Therefore, the binary usually cannot tell you directly:

> "This code came from a macro."

You infer it from behavior and surrounding code.

---

# Mental Model

Think of the preprocessor as a **source-code transformation stage**:

```text
Original C
    │
    ▼
┌─────────────────┐
│  PREPROCESSOR   │
│                 │
│ #include        │
│ #define         │
│ #if/#ifdef      │
│ #undef          │
│ #pragma         │
│ #embed          │
└─────────────────┘
    │
    ▼
Expanded C
    │
    ▼
Compiler
    │
    ▼
Assembly
    │
    ▼
Machine Code
```

---

# Long-Term Takeaways

- The **preprocessor runs before the compiler**.
    
- Preprocessor directives begin with `#`.
    
- `#include` inserts another file's contents.
    
- `#define` creates a macro.
    
- Macros are primarily **preprocessor substitutions**, not variables.
    
- `#ifdef` checks whether a macro is defined.
    
- `#ifndef` checks whether a macro is not defined.
    
- `#if` evaluates a preprocessor expression.
    
- `#else` and `#elif` provide alternative conditional sections.
    
- `#undef` removes a macro definition.
    
- `#` inside a macro performs **stringification**.
    
- `##` performs **token concatenation**.
    
- `...` allows variable-argument macros.
    
- Multiline macros use `\` at the end of each continued line.
    
- `#error` deliberately produces a preprocessing error.
    
- `#embed` can embed binary resources into C source.
    
- `#pragma` provides compiler/implementation-specific instructions.
    
- `_Pragma` allows pragmas to be generated through macros.
    
- `#line` changes reported source-file and line information.
    
- A line containing only `#` is the **null directive**.
    
- Conditional compilation can completely remove code **before the compiler sees it**.
    
- In reverse engineering, you analyze the **compiled result**, not the original preprocessor directives.
    
- Macros generally leave **no identifiable macro name** in the final binary.
    
- Understanding preprocessing helps explain why source code and disassembly can look very different.