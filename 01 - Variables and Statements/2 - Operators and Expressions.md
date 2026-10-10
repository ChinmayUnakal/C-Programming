
## What are Operators?

Operators are **symbols** that perform operations on one or more operands (values or variables).

Example:

```c
int x = 5 + 3;
```

`+` is the operator, while `5` and `3` are operands.

---

## What is an Expression?

An **expression** is any valid combination of variables, constants, and operators that produces a value.

Examples:

```c
5 + 3
```

```c
a * b
```

```c
x > 10
```

```c
i += 2
```

---

# Arithmetic Operators

Used for mathematical operations.

|Operator|Meaning|
|---|---|
|`+`|Addition|
|`-`|Subtraction|
|`*`|Multiplication|
|`/`|Division|
|`%`|Modulus (remainder)|

Example:

```c
i = i + 3;
i = i - 8;
i = i * 9;
i = i / 2;
i = i % 5;
```

---

## Compound Assignment Operators

A shorthand for updating variables.

```c
i += 3;
i -= 8;
i *= 9;
i /= 2;
i %= 5;
```

Equivalent to:

```c
i = i + 3;
```

etc.

---

# Ternary Operator (`?:`)

A compact form of `if-else`.

Syntax:

```c
condition ? value_if_true : value_if_false;
```

Example:

```c
printf("%s", x % 2 == 0 ? "Even" : "Odd");
```

Equivalent to:

```c
if (x % 2 == 0)
    printf("Even");
else
    printf("Odd");
```

---

# Increment and Decrement

Increase or decrease a variable by one.

## Post

```c
i++;
i--;
```

The expression uses the **old value**, then updates it.

```c
i = 10;
j = i++;
```

Result:

```text
i = 11
j = 10
```

---

## Pre

```c
++i;
--i;
```

The variable is updated **before** the expression is evaluated.

```c
i = 10;
j = ++i;
```

Result:

```text
i = 11
j = 11
```

---

# Comma Operator

Evaluates expressions **left to right**.

Returns the value of the **last expression**.

Example:

```c
x = (1, 2, 3);
```

Result:

```text
x = 3
```

Rarely used in modern C.

---

# Comparison (Relational) Operators

Used to compare values.

|Operator|Meaning|
|---|---|
|`==`|Equal|
|`!=`|Not equal|
|`<`|Less than|
|`>`|Greater than|
|`<=`|Less than or equal|
|`>=`|Greater than or equal|

Example:

```c
if (age >= 18)
```

Comparison operators return:

```text
1 (true)
0 (false)
```

---

# Boolean (Logical) Operators

Used to combine or invert conditions.

|Operator|Meaning|
|---|---|
|`&&`|AND|
|`||
|`!`|NOT|

Examples:

```c
if (x < 10 && y > 20)
```

```c
if (x == 1 || y == 2)
```

```c
if (!found)
```

---

# sizeof Operator

Returns the size (in bytes) of a data type or object.

Examples:

```c
sizeof(int)
```

```c
sizeof a
```

```c
sizeof(2 + 7)
```

Typical sizes (x86-64):

|Type|Size|
|---|---|
|`char`|1 byte|
|`int`|4 bytes|
|`float`|4 bytes|
|`double`|8 bytes|
|Pointer|8 bytes|

Example:

```c
printf("%zu\n", sizeof(int));
```

---

# Reverse Engineering Perspective (x86-64 Assembly)

## Arithmetic

C:

```c
a = b + c;
```

Assembly:

```asm
mov eax, [b]
add eax, [c]
mov [a], eax
```

Arithmetic operators become instructions like:

```text
add
sub
imul
idiv
```

---

## Comparisons

C:

```c
if (a == b)
```

Assembly:

```asm
cmp eax, ebx
je label
```

Common instructions:

```text
cmp
test
je
jne
jg
jl
jge
jle
```

---

## Boolean Logic

C:

```c
if (x && y)
```

Usually becomes multiple comparisons and conditional jumps.

Example:

```asm
cmp eax, 0
je fail

cmp ebx, 0
je fail
```

---

## Ternary Operator

C:

```c
x = cond ? a : b;
```

May compile to:

```asm
cmp eax, 0
je else

mov ecx, a
jmp end

else:
mov ecx, b
```

or, in optimized builds:

```asm
cmov
```

(Conditional Move)

---

## Increment / Decrement

C:

```c
i++;
```

Assembly:

```asm
inc eax
```

or

```asm
add eax, 1
```

Modern compilers often prefer `add`.

---

## sizeof

`sizeof` is evaluated **at compile time**.

Example:

```c
sizeof(int)
```

becomes the constant:

```text
4
```

No runtime instruction is generated.

---

# Mental Model

```text
Operator
    │
    ▼
Performs an Operation
    │
    ▼
Produces a Value
    │
    ▼
Compiler
    │
    ▼
Assembly Instructions
```

---

# Long-Term Takeaways

- Operators perform operations on operands.
    
- Expressions always evaluate to a value.
    
- `==` compares values, `=` assigns values.
    
- Pre-increment updates first; post-increment updates later.
    
- `sizeof` is usually computed at compile time.
    
- In assembly, operators become CPU instructions like `add`, `sub`, `cmp`, `test`, and conditional jumps.