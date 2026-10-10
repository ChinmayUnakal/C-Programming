
A **variadic function** is a function that accepts a variable number of arguments.

Example:

```c
void func(int a, ...)
```

The three dots:

```c
...
```

are called an **ellipsis**.

They mean:

> Additional arguments may be passed after the named parameters.

---

# Basic Example

```c
#include <stdio.h>

void func(int a, ...)
{
    printf("a is %d\n", a);
}

int main(void)
{
    func(2, 3, 4, 5, 6);
}
```

Output:

```text
a is 2
```

The function receives all the arguments, but to access the unnamed arguments, we need the facilities in:

```c
#include <stdarg.h>
```

---

# Why Use Variadic Functions?

Variadic functions are useful when the number of arguments is not known in advance.

Common examples:

- `printf()`
    
- Logging functions
    
- Wrapper functions
    
- Functions accepting arbitrary numbers of values
    

Example:

```c
printf("%d %d %d\n", 10, 20, 30);
```

`printf()` uses the format string to determine how to interpret the additional arguments.

---

# Required Tools — `<stdarg.h>`

Variadic functions use:

```c
#include <stdarg.h>
```

The main tools are:

```c
va_list
va_start()
va_arg()
va_end()
va_copy()
```

---

# The Variadic Function Process

The standard process is:

```text
va_start()
    │
    ▼
va_arg()
va_arg()
va_arg()
    │
    ▼
va_end()
```

In general:

1. Initialize the argument list with `va_start()`.
    
2. Retrieve arguments with `va_arg()`.
    
3. Clean up with `va_end()`.
    

---

# `va_list`

`va_list` is a type used to keep track of the current position in the variable argument list.

Example:

```c
va_list va;
```

Conceptually:

```text
Arguments

count | 10 | 20 | 30
          ▲
          │
      va_list tracks
      the next argument
```

Its actual implementation is **opaque** and implementation-dependent.

You should only manipulate it using the standard `va_*` macros.

---

# `va_start()`

`va_start()` initializes a `va_list`.

Example:

```c
va_start(va, count);
```

Syntax:

```c
va_start(va_list_variable, last_named_parameter);
```

The second argument must be the last named parameter before:

```c
...
```

Example:

```c
int add(int count, ...)
```

Here:

```c
count
```

is the last named parameter.

So:

```c
va_start(va, count);
```

initializes access to the arguments after `count`.

---

# `va_arg()`

`va_arg()` retrieves the next variable argument.

Example:

```c
int n = va_arg(va, int);
```

Syntax:

```c
va_arg(va_list_variable, type)
```

It:

1. Gets the next argument.
    
2. Interprets it as the specified type.
    
3. Advances the `va_list`.
    

Conceptually:

```text
va ───► 10
         │
         ▼
va_arg(va, int)
         │
         ▼
va ───► 20
```

You must know the correct type.

---

# `va_end()`

When finished, call:

```c
va_end(va);
```

This ends use of that particular `va_list`.

Important:

> After using `va_start()` or `va_copy()`, the corresponding `va_list` must eventually be passed to `va_end()`.

---

# Example — Adding Variable Arguments

```c
#include <stdio.h>
#include <stdarg.h>

int add(int count, ...)
{
    int total = 0;

    va_list va;

    va_start(va, count);

    for (int i = 0; i < count; i++) {
        int n = va_arg(va, int);
        total += n;
    }

    va_end(va);

    return total;
}

int main(void)
{
    printf("%d\n", add(4, 6, 2, -4, 17));
    printf("%d\n", add(2, 22, 44));
}
```

Output:

```text
21
66
```

---

# How Does the Function Know How Many Arguments Exist?

The ellipsis itself provides **no information** about:

- Number of arguments
    
- Argument types
    

The programmer must provide a way to determine them.

Common approaches:

### Count

```c
add(4, 6, 2, -4, 17);
```

`4` tells `add()` to read four additional arguments.

### Format String

```c
printf("%d %f", x, y);
```

The format string tells `printf()` what arguments to read and their types.

### Sentinel Value

Some functions use a special terminating value, such as `NULL`.

---

# Important — You Must Request the Correct Type

This is dangerous:

```c
double x = 3.14;

int n = va_arg(va, int);
```

if the next argument is actually a `double`.

`va_arg()` does not magically determine the type.

You must correctly know what type was passed.

Getting the wrong type can cause **undefined behavior**.

---

# Default Argument Promotions

Arguments passed through `...` undergo the **default argument promotions**.

Important examples:

```text
float
  │
  ▼
double
```

and integer types smaller than `int`, such as:

```text
char
short
```

are promoted to:

```text
int
```

or sometimes `unsigned int`.

So this is wrong:

```c
float x = va_arg(va, float);
```

For an argument originally passed as `float`, you usually retrieve it as:

```c
double x = va_arg(va, double);
```

This is a very important variadic-function rule.

---

# `va_copy()`

`va_copy()` copies the state of one `va_list` into another.

Example:

```c
va_list va1, va2;

va_start(va1, count);
va_copy(va2, va1);
```

Conceptually:

```text
va1 ───► current argument
va2 ───► same argument
```

If `va1` has already consumed five arguments, the copy begins at the same position.

---

# Why Use `va_copy()`?

It is useful when you need to process the same arguments more than once.

Example conceptually:

```text
Original va_list
      │
      ├──► First pass
      │
      └──► Copy for second pass
```

Each copied `va_list` must also be ended:

```c
va_end(va1);
va_end(va2);
```

---

# Variadic Function Mental Model

```text
Named Arguments

count = 4
     │
     ▼
va_start()
     │
     ▼
Variable Arguments

6 ───► 2 ───► -4 ───► 17
▲
│
va_list
```

Each call to:

```c
va_arg()
```

moves to the next argument.

---

# Passing Variadic Arguments to Another Function

The C library provides `v` versions of many variadic functions.

Examples include:

```c
vprintf()
vfprintf()
vsnprintf()
```

These accept a:

```c
va_list
```

instead of `...`.

---

# Example — Wrapper Around `printf()`

```c
#include <stdio.h>
#include <stdarg.h>

int my_printf(int serial, const char *format, ...)
{
    va_list va;
    int rv;

    printf("The serial number is: %d\n", serial);

    va_start(va, format);

    rv = vprintf(format, va);

    va_end(va);

    return rv;
}

int main(void)
{
    int x = 10;
    float y = 3.2;

    my_printf(3490, "x is %d, y is %f\n", x, y);
}
```

Output:

```text
The serial number is: 3490
x is 10, y is 3.200000
```

---

# Why `vprintf()`?

You cannot directly pass a `va_list` to ordinary `printf()`.

This:

```c
printf(format, va);
```

is wrong.

Instead:

```c
vprintf(format, va);
```

is designed specifically to accept a `va_list`.

Conceptually:

```text
my_printf(...)

     │
     ▼

va_list

     │
     ▼

vprintf(format, va)
```

---

# Variadic Functions and the Preprocessor

The `va_*` facilities are commonly implemented as macros.

This means they may behave differently internally across implementations.

For example:

```c
va_start(va, format);
```

might expand into more complex implementation-specific code.

Therefore, avoid making unsafe assumptions about their implementation.

---

# Macro Scope Gotcha

Avoid declaring important variables in a place where a macro expansion could theoretically create a scope problem.

Safer:

```c
va_list va;
int rv;

va_start(va, format);

rv = vprintf(format, va);

va_end(va);

return rv;
```

The variable:

```c
rv
```

is declared before the variadic macros are used.

This avoids depending on implementation details of macro expansion.

---

# Function Signature Requirements

A traditional variadic function has at least one named parameter before:

```c
...
```

Example:

```c
int add(int count, ...)
```

The named parameter allows:

```c
va_start()
```

to initialize the variable argument processing.

The last named parameter is passed to `va_start()`.

---

# Reverse Engineering Perspective

Consider:

```c
printf("%d %f\n", x, y);
```

At the source level:

```text
Format String
     │
     ├── %d ───► int
     │
     └── %f ───► double
```

But at the machine-code level, the CPU does not know about `...`.

It simply follows the platform's **calling convention**.

Arguments may be passed through:

- Registers
    
- Stack
    
- Special register areas depending on the ABI
    

The called function uses ABI rules and information such as the format string to interpret them.

---

# Variadic Functions in x86-64

On x86-64, the implementation of:

```c
va_list
```

depends on the platform ABI.

This means a `va_list` is **not necessarily just a simple pointer**.

For example, it may track:

```text
Register arguments
        │
        ▼
Current position
        │
        ├── General-purpose registers
        ├── Floating-point registers
        └── Stack arguments
```

This is why you should never manually assume what a `va_list` looks like.

Use:

```c
va_start()
va_arg()
va_end()
va_copy()
```

---

# Mental Model

```text
Function Call

add(4, 6, 2, -4, 17)

     │
     ▼

Named Argument

count = 4

     │
     ▼

Initialize

va_start(va, count)

     │
     ▼

Read Arguments

va_arg() ───► 6
va_arg() ───► 2
va_arg() ───► -4
va_arg() ───► 17

     │
     ▼

Finish

va_end(va)
```

---

# Long-Term Takeaways

- A variadic function accepts a variable number of arguments using `...`.
    
- Include `<stdarg.h>` to process variable arguments.
    
- `va_list` tracks the current position in the argument list.
    
- `va_start()` initializes a `va_list`.
    
- The second argument to `va_start()` is the last named parameter.
    
- `va_arg()` retrieves the next argument and requires its correct type.
    
- `va_arg()` advances the `va_list`.
    
- `va_end()` must be called when finished with a `va_list`.
    
- The ellipsis does not tell the function how many arguments exist or what their types are.
    
- Use a count, format string, or sentinel to determine the arguments.
    
- Requesting an argument with the wrong type causes undefined behavior.
    
- Default argument promotions apply to arguments passed through `...`.
    
- `float` is promoted to `double`.
    
- Types smaller than `int` are promoted to `int` or `unsigned int`.
    
- `va_copy()` copies the current state of a `va_list`.
    
- Every initialized or copied `va_list` must eventually have `va_end()` called on it.
    
- `vprintf()` and similar functions accept a `va_list`.
    
- Variadic wrapper functions commonly use `v` library functions.
    
- `va_*` facilities may be implemented as macros, so avoid relying on their internal expansion.
    
- At the machine-code level, variadic arguments follow the platform's calling convention.
    
- `va_list` can be ABI-specific and should always be accessed through the standard `va_*` facilities.