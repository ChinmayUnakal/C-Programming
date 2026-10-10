
## What is a String?

C has **no built-in string data type**.

A C string is a sequence of characters ending with a special character:

```c
'\0'
```

This is called the **null terminator**.

Example:

```c
"Hello"
```

Memory:

```text
'H' 'e' 'l' 'l' 'o' '\0'
```

The null terminator marks where the string ends.

---

# Why is `'\0'` Important?

C does not automatically store a string's length with the string.

Functions such as `printf("%s")` and `strlen()` continue reading characters until they find:

```c
'\0'
```

Without a null terminator, the program may read past the intended memory.

---

# String Literals

A string literal is written using double quotes:

```c
"Hello"
```

A string literal includes an implicit null terminator.

```text
"Hello"

H e l l o \0
```

The literal `"Hello"` therefore requires **6 bytes**.

---

# String Variables

Strings are commonly represented using:

```c
char name[] = "Alice";
```

Memory:

```text
'A' 'l' 'i' 'c' 'e' '\0'
```

The compiler determines the required array size.

Equivalent to:

```c
char name[] = {'A', 'l', 'i', 'c', 'e', '\0'};
```

---

# String Variables as Arrays

A string stored in a character array can be accessed using indexes.

```c
char name[] = "Alice";
```

```c
name[0];    // 'A'
name[1];    // 'l'
name[4];    // 'e'
name[5];    // '\0'
```

Example:

```c
name[0] = 'M';
```

The string becomes:

```text
Mlice
```

---

# String Initializers

Let the compiler determine the size:

```c
char name[] = "Alice";
```

Explicit size:

```c
char name[6] = "Alice";
```

Because:

```text
A l i c e \0
```

requires 6 characters.

You can also allocate more space:

```c
char name[20] = "Alice";
```

The remaining elements are initialized to zero.

---

# Character Array vs String Pointer

Character array:

```c
char name[] = "Alice";
```

Creates an array containing a copy of the characters.

Its characters can be modified:

```c
name[0] = 'M';
```

---

Pointer to a string literal:

```c
const char *name = "Alice";
```

`name` stores an address pointing to the string literal.

The characters should **not** be modified.

```c
name[0] = 'M';    // Undefined behavior
```

---

# Getting String Length

Use:

```c
strlen()
```

Example:

```c
#include <string.h>

char name[] = "Alice";

size_t length = strlen(name);
```

Result:

```text
length = 5
```

`strlen()` does **not** count the null terminator.

Conceptually:

```text
A l i c e \0
1 2 3 4 5 STOP
```

---

# String Termination

A valid C string must end with:

```c
'\0'
```

Correct:

```c
char name[] = "Hello";
```

Incorrect as a C string:

```c
char name[5] = {'H', 'e', 'l', 'l', 'o'};
```

This array contains characters but **is not a null-terminated C string**.

Using it with:

```c
printf("%s", name);
```

causes **undefined behavior** because `printf` continues reading memory until it eventually finds `'\0'`.

Correct:

```c
char name[6] = {'H', 'e', 'l', 'l', 'o', '\0'};
```

---

# Copying a String

You cannot assign one character array directly to another:

```c
char a[10] = "Hello";
char b[10];

b = a;    // Invalid
```

Use:

```c
strcpy()
```

Example:

```c
#include <string.h>

char a[] = "Hello";
char b[10];

strcpy(b, a);
```

Result:

```text
b = "Hello"
```

`strcpy()` also copies the null terminator.

### Important

The destination must have enough space.

```c
char b[3];

strcpy(b, "Hello");    // Buffer overflow
```

This causes undefined behavior.

---

# Reverse Engineering Perspective (x86-64 Assembly)

## Strings in Memory

A string such as:

```c
"Hello"
```

appears in memory as:

```text
48 65 6C 6C 6F 00
 H  e  l  l  o \0
```

Each character is usually one byte.

---

## Accessing String Characters

C:

```c
char c = name[i];
```

Assembly may look like:

```asm
mov al, BYTE PTR [rbx + rcx]
```

Where:

```text
rbx = string base address
rcx = index
al  = character
```

Unlike an `int` array, no scaling is needed because:

```text
sizeof(char) = 1
```

---

## String Length

Conceptually, `strlen()` performs:

```text
Start
  │
  ▼
Read Character
  │
  ▼
Is it '\0'?
 │       │
No      Yes
 │       │
 ▼       ▼
Next    Return
Byte
```

At the assembly level, this often appears as a loop checking each byte for:

```text
0x00
```

---

## String Comparisons

String comparisons often involve checking characters one byte at a time.

You may see instructions such as:

```asm
cmp al, 0
je end
```

or comparisons against individual ASCII values.

For example:

```text
'A' = 0x41
'B' = 0x42
'a' = 0x61
```

Recognizing ASCII values in disassembly is useful during reverse engineering.

---

## String References

In a binary, code usually does not contain a whole string inside every instruction.

Instead, code loads the **address of a string**.

Conceptually:

```asm
lea rcx, [rip + string]
call printf
```

This means:

```text
RCX ───► "Hello\0"
```

---

# Mental Model

```text
C String

H  e  l  l  o  \0
│              │
└── Characters ┘
               │
               └── End Marker
```

String pointer:

```text
Pointer
   │
   ▼
 H  e  l  l  o  \0
```

---

# Long-Term Takeaways

- C has no built-in string type.
    
- A C string is a sequence of `char` values ending with `'\0'`.
    
- String literals automatically include a null terminator.
    
- `strlen()` does not count `'\0'`.
    
- `char name[]` creates an array; `const char *name` points to a string literal.
    
- Always ensure enough space for characters **plus `'\0'`**.
    
- `strcpy()` requires a sufficiently large destination buffer.
    
- In memory, strings are sequences of bytes ending in `0x00`.
    
- In reverse engineering, strings are commonly identified by their ASCII bytes and references to their addresses.