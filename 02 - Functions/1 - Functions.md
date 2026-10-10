
## What is a Function?

A **function** is a reusable block of code that performs a specific task.

Instead of writing the same code multiple times, you write it once and call it whenever needed.

---

# Why Use Functions?

Functions improve:

- Code reuse
    
- Readability
    
- Maintainability
    
- Modularity
    

---

# Function Syntax

```c
return_type function_name(parameters)
{
    // code
    return value;
}
```

Example:

```c
int add(int a, int b)
{
    return a + b;
}
```

Calling the function:

```c
int result = add(5, 3);
```

---

# Function Components

Every function has:

- Return type
    
- Function name
    
- Parameters (optional)
    
- Function body
    
- Return statement (if required)
    

Example:

```c
int multiply(int x, int y)
{
    return x * y;
}
```

---

# Return Type

Specifies what the function returns.

Examples:

```c
int getAge();
```

```c
char getGrade();
```

```c
float average();
```

```c
void printMessage();
```

`void` means the function returns nothing.

---

# Parameters

Parameters are variables that receive values from the caller.

Example:

```c
int square(int x)
{
    return x * x;
}
```

Calling:

```c
square(5);
```

`5` is the **argument**.

`x` is the **parameter**.

---

# Passing by Value

C passes arguments **by value**.

A **copy** of the variable is given to the function.

Example:

```c
void increment(int x)
{
    x++;
}
```

```c
int a = 5;

increment(a);
```

Result:

```text
a = 5
```

The original variable is unchanged because only a copy was modified.

---

# Passing by Value (Memory View)

```text
Main

a = 5
 │
 │ copy
 ▼

Function

x = 5
```

Changing `x` does **not** affect `a`.

To modify the original variable, C uses **pointers** (pass by address).

---

# Function Prototype

A function prototype tells the compiler:

- Function name
    
- Return type
    
- Parameter types
    

before the function is used.

Syntax:

```c
return_type function_name(parameters);
```

Example:

```c
int add(int, int);
```

Later:

```c
int add(int a, int b)
{
    return a + b;
}
```

Function prototypes are commonly placed at the top of a source file or inside header (`.h`) files.

---

# Empty Parameter Lists

A function that takes **no arguments** should be declared as:

```c
void hello(void)
{
    printf("Hello");
}
```

Calling:

```c
hello();
```

### Important

```c
hello(void);
```

means:

> Takes **no parameters**.

Whereas

```c
hello();
```

in C means:

> Parameter list is **unspecified** (old C style).

Modern C prefers:

```c
hello(void);
```

---

# Reverse Engineering Perspective (x86-64 Assembly)

## Function Calls

C:

```c
result = add(5, 3);
```

Assembly:

```asm
mov ecx, 5
mov edx, 3
call add
```

The `call` instruction transfers execution to another function.

---

## Returning from Functions

C:

```c
return x + y;
```

Assembly:

```asm
add eax, edx
ret
```

The return value is commonly placed in:

```text
EAX (32-bit)
RAX (64-bit)
```

---

## Function Parameters

Parameters are passed according to the calling convention.

Windows x64:

|Parameter|Register|
|---|---|
|1|RCX|
|2|RDX|
|3|R8|
|4|R9|

Additional parameters are passed on the stack.

---

## Stack Frame

Every function typically creates its own stack frame.

```text
Higher Addresses
-------------------
Function Arguments
Return Address
Saved RBP
Local Variables
-------------------
Lower Addresses
```

Local variables usually appear as:

```asm
[rbp-4]
[rbp-8]
```

---

## Returning

Every function eventually ends with:

```asm
ret
```

which returns execution to the calling function.

---

# Mental Model

```text
Caller
   │
   ▼
Pass Arguments
   │
   ▼
Function Executes
   │
   ▼
Return Value
   │
   ▼
Caller Continues
```

---

# Long-Term Takeaways

- Functions break programs into reusable blocks.
    
- Parameters receive values from the caller.
    
- Arguments are copied in **pass-by-value**.
    
- Function prototypes tell the compiler a function exists before its definition.
    
- Use `void` for functions that take no parameters.
    
- In assembly, functions are implemented using `call` and `ret`.
    
- Function arguments are passed through registers and/or the stack, depending on the calling convention.