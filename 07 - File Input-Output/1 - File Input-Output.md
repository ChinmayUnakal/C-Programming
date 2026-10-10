
## What is File I/O?

File I/O allows a program to:

- Open files
    
- Read data from files
    
- Write data to files
    
- Close files
    

C provides file I/O functions through:

```c
#include <stdio.h>
```

---

# The `FILE *` Data Type

`FILE` is a type used by the C standard library to represent an open file stream.

A program usually accesses a file using a pointer:

```c
FILE *fp;
```

Open a file:

```c
fp = fopen("file.txt", "r");
```

Always check if opening succeeded:

```c
if (fp == NULL)
{
    // Failed to open file
}
```

Close the file when finished:

```c
fclose(fp);
```

---

# File Modes

Common modes:

|Mode|Meaning|
|---|---|
|`"r"`|Read text file|
|`"w"`|Write text file|
|`"a"`|Append to text file|
|`"rb"`|Read binary file|
|`"wb"`|Write binary file|
|`"ab"`|Append binary file|

`"w"` creates a file or **truncates an existing file**.

`"a"` writes at the end of the file.

---

# Reading Text Files

A simple way to read characters is:

```c
int c;

while ((c = fgetc(fp)) != EOF)
{
    putchar(c);
}
```

`fgetc()` reads one character at a time.

Its return type should be `int` so it can also represent `EOF`.

---

# End of File (`EOF`)

`EOF` means **End Of File**.

It indicates that no more input is available.

Example:

```c
while ((c = fgetc(fp)) != EOF)
{
    // process character
}
```

Important:

```text
EOF is not a character stored in the file.
```

It is a special value returned by input functions.

---

# Reading a Line at a Time

Use:

```c
fgets()
```

Example:

```c
char line[100];

while (fgets(line, sizeof(line), fp) != NULL)
{
    printf("%s", line);
}
```

`fgets()`:

- Reads up to a specified limit
    
- Stops at a newline or end of input
    
- Adds `'\0'` when successful
    

---

# Formatted Input

Use:

```c
fscanf()
```

Example:

```c
int age;

fscanf(fp, "%d", &age);
```

For multiple values:

```c
char name[50];
int age;

fscanf(fp, "%49s %d", name, &age);
```

Always limit string input to prevent buffer overflow.

---

# Writing Text Files

Use:

```c
fprintf()
```

Example:

```c
FILE *fp = fopen("output.txt", "w");

if (fp != NULL)
{
    fprintf(fp, "Hello\n");
    fclose(fp);
}
```

`fprintf()` works like `printf()`, but writes to a file stream.

---

# Binary File I/O

Binary I/O reads and writes **raw bytes**.

Main functions:

```c
fread()
fwrite()
```

Example:

```c
int x = 123;

fwrite(&x, sizeof(x), 1, fp);
```

This writes:

```text
sizeof(x) bytes
```

directly from memory to the file.

Reading:

```c
int x;

fread(&x, sizeof(x), 1, fp);
```

This reads raw bytes directly into `x`.

---

# `fread()` and `fwrite()` Syntax

```c
fread(pointer, size, count, file);
```

```c
fwrite(pointer, size, count, file);
```

Example:

```c
fwrite(array, sizeof(array[0]), length, fp);
```

Meaning:

```text
Pointer → Where data is
Size    → Size of one item
Count   → Number of items
File    → File stream
```

---

# Writing Structs to Files

A struct can be written directly:

```c
struct Person
{
    int age;
    float height;
};

struct Person p = {23, 178.0f};

fwrite(&p, sizeof(p), 1, fp);
```

Read it back:

```c
struct Person p;

fread(&p, sizeof(p), 1, fp);
```

This is called writing the struct's **raw memory representation**.

---

# Struct Caveats

Writing raw structs directly to files can cause portability problems.

Reasons:

- Padding may differ
    
- Alignment may differ
    
- Type sizes may differ
    
- Endianness may differ
    
- Compiler and architecture may differ
    

For example:

```c
struct Person
{
    char grade;
    int age;
};
```

The compiler may insert padding:

```text
+-------+
| grade | 1 byte
+-------+
|padding| possible
+-------+
|  age  | 4 bytes
+-------+
```

The file contains this actual memory layout.

---

# Number Caveats

Binary numbers may not have the same representation on every system.

Important differences include:

```text
Endianness
Integer size
Alignment
Floating-point representation
```

Example:

```text
0x12345678
```

Little-endian memory:

```text
78 56 34 12
```

Big-endian memory:

```text
12 34 56 78
```

This matters when reading binary files across different systems.

---

# Reverse Engineering Perspective

## Files Are Just Bytes

A binary file is ultimately:

```text
Byte  Byte  Byte  Byte  Byte
```

A program interprets those bytes according to a format.

For example:

```text
Executable → PE / ELF format
Image      → PNG / JPEG format
Document   → PDF format
```

---

## Text vs Binary

Text file:

```text
Hello
```

Stored as character bytes:

```text
48 65 6C 6C 6F
```

Binary file:

```text
Raw bytes representing program-defined data
```

There is no automatic human-readable representation.

---

## File Reading in Assembly

A C function such as:

```c
fread(buffer, 1, size, fp);
```

usually becomes a function call following the platform calling convention.

On Windows x64:

```text
RCX → buffer
RDX → size
R8  → count
R9  → FILE pointer
```

Then:

```asm
call fread
```

Understanding arguments passed to file functions is useful when reversing programs.

---

## Buffers

File data is commonly read into a buffer:

```c
char buffer[1024];

fread(buffer, 1, sizeof(buffer), fp);
```

Memory:

```text
File
 │
 ▼
Raw Bytes
 │
 ▼
Buffer in Memory
 │
 ▼
Program Processes Data
```

When reversing a binary, follow:

```text
File Input
   ↓
Buffer
   ↓
Parsing
   ↓
Data Structure
```

This often reveals how a program handles a file format.

---

# Mental Model

```text
Program
   │
   ▼
fopen()
   │
   ▼
FILE *
   │
   ├──► Read → Buffer → Program
   │
   └──► Write → File
   │
   ▼
fclose()
```

Binary I/O:

```text
Memory
  │
  │ raw bytes
  ▼
File
```

---

# Long-Term Takeaways

- `FILE *` represents an open file stream.
    
- `fopen()` opens a file; `fclose()` closes it.
    
- Always check whether `fopen()` returned `NULL`.
    
- `EOF` is a special return value, not a character in the file.
    
- `fgets()` reads text line by line.
    
- `fprintf()` writes formatted text.
    
- `fread()` and `fwrite()` read/write raw bytes.
    
- Binary I/O often works directly with memory buffers.
    
- Writing raw structs can cause portability problems.
    
- Padding, alignment, type sizes, and endianness affect binary data.
    
- In reverse engineering, files are ultimately **bytes interpreted as data structures**.