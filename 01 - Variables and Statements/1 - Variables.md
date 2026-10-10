
## What is a Variable?

A **variable** is a **human-readable name** that refers to a location in memory where data is stored.

People often say **"variables hold values."** While this is a useful simplification, a more accurate way to think about it is:

> A variable is a name that allows programmers to access data stored somewhere in memory.

The compiler eventually replaces these names with actual memory addresses or CPU registers.

---

# Why Do Variables Exist?

Computers store data using **memory addresses**.

For example, a value might be stored at an address like:

```
0x7FFD3A91B8C4
```

Using these addresses directly would make programs almost impossible for humans to read and maintain.

Instead, we give meaningful names:

```c
int age = 23;
```

Now we can write:

```c
printf("%d", age);
```

instead of remembering a long memory address.

Variables improve:

- Readability
    
- Maintainability
    
- Debugging
    
- Code organization
    

---

# Variable Declaration

A variable is declared using:

```c
DataType VariableName;
```

Example:

```c
int age;
```

or

```c
int age = 23;
```

The second example both **declares** and **initializes** the variable.

---

# Common Data Types

## int

Stores whole numbers.

Example:

```c
int score = 100;
```

Typical size:

```
4 bytes
```

---

## float

Stores decimal numbers.

Example:

```c
float temperature = 36.5f;
```

Typical size:

```
4 bytes
```

---

## double

Stores decimal numbers with greater precision.

Example:

```c
double pi = 3.1415926535;
```

Typical size:

```
8 bytes
```

---

## char

Stores a **single character**.

Example:

```c
char grade = 'A';
```

Typical size:

```
1 byte
```

Characters are actually stored using their ASCII (or compatible) integer value.

```
'A' = 65
```

---

## char *

A **pointer** to the first character of a character array.

Used to represent C strings.

Example:

```c
char *name = "Alice";
```

The variable does **not** contain the string itself.

Instead, it stores the address of the first character.

```
name
 │
 ▼
'A''l''i''c''e''\0'
```

---

# Variable Naming Rules

Allowed characters:

```
A-Z
a-z
0-9
_
```

Rules:

- Cannot begin with a number.
    
- Cannot contain spaces.
    
- Cannot use C keywords.
    
- Avoid names beginning with underscores, especially:
    
    - `__name`
        
    - `_Name`
        

These are reserved for the compiler and standard library.

Good:

```c
count
studentAge
total_sum
```

Bad:

```c
1count
my age
int
```

---

# Memory Representation

Example:

```c
int age = 23;
```

Conceptually:

```
 Variable

   age
    │
    ▼
  Memory
+---------+
|   23    |
+---------+
```

The variable name exists only for the programmer and compiler.

The CPU only sees:

- registers
    
- memory addresses
    
- instructions
    

---

# Variable Lifetime

A variable exists only while its lifetime allows.

For example:

```c
void func()
{
    int x = 5;
}
```

After the function returns, `x` no longer exists.

(Static and global variables behave differently.)

---

# Key Ideas

Remember:

- Variables are names.
    
- Data lives in memory.
    
- Variables make code readable.
    
- The compiler translates variable names into addresses or registers.
    
- The CPU never knows variable names.
    

---

# Reverse Engineering Perspective (x86-64 Assembly)

When a program is compiled:

- Variable names are usually removed.
    
- Only memory locations and registers remain.
    
- Reverse engineers must infer what each variable represents.
    

---

## Local Variables

C:

```c
int x = 10;
```

Possible assembly:

```asm
mov DWORD PTR [rbp-4], 10
```

Meaning:

```
[rbp-4]
```

is simply the storage location for `x`.

The compiler knows it as `x`.

The CPU only knows:

> "Store 10 at rbp minus 4."

---

## Variables in Registers

The compiler may avoid memory entirely.

C:

```c
int a = 5;
int b = a + 2;
```

Possible assembly:

```asm
mov eax, 5
add eax, 2
```

Here:

```
eax
```

temporarily holds the value.

No memory access is necessary.

This is called **register allocation**.

---

## Stack Variables

Inside functions, local variables are commonly stored on the stack.

Example:

```c
void func()
{
    int a = 5;
    int b = 10;
}
```

Stack layout (conceptual):

```
Higher Addresses
-----------------
Return Address
Saved RBP
b
a
-----------------
Lower Addresses
```

Assembly often accesses them like:

```asm
[rbp-4]
[rbp-8]
```

---

## Global Variables

Global variables live in the program's data sections rather than on the stack.

C:

```c
int counter = 0;
```

Assembly may reference it using RIP-relative addressing:

```asm
mov eax, DWORD PTR [rip+counter]
```

---

## Optimized Builds

With compiler optimizations enabled:

- Variables may never exist in memory.
    
- Multiple variables may reuse the same register.
    
- Some variables disappear completely.
    
- Constant values may be substituted directly into instructions.
    

This is why optimized binaries are much harder to reverse engineer.

---

## Decompiled Code

A decompiler such as Ghidra or IDA Pro invents variable names because the original names are usually gone.

Example:

```c
int local_10;
char local_18[16];
```

or

```c
v1
v2
v3
```

These names did **not** exist in the original program.

They are placeholders generated by the decompiler.

---

# Mental Model

When writing C:

```
Variable Name
      │
      ▼
   Compiler
      │
      ▼
Memory Address / Register
      │
      ▼
CPU Instructions
```

When reverse engineering:

```
CPU Instructions
      │
      ▼
Registers + Memory Addresses
      │
      ▼
Decompiler guesses variable names
      │
      ▼
Reverse engineer figures out the real meaning
```

---

# Long-Term Takeaways

- A variable is a **name**, not the data itself.
    
- Data is stored in **memory** or **CPU registers**.
    
- The compiler maps names to addresses or registers.
    
- Assembly contains addresses and registers—not source-level variable names.
    
- Reverse engineering is the process of reconstructing the programmer's intent from machine code.