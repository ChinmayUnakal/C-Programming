# Program Exit & Termination

These functions are provided by:

```c
#include <stdlib.h>
```

They control how a C program terminates.

---

# Normal Exit

A **normal exit** allows the C runtime to perform normal cleanup before the program terminates.

Common ways:

- Returning from `main()`
    
- `exit()`
    

---

# Returning From `main()`

Returning from `main()` terminates the program normally.

Example:

```c
int main(void)
{
    return 0;
}
```

Equivalent to:

```c
exit(0);
```

### Exit Status

The returned value is the program's **exit status**.

```text
0       → success
non-zero → error/failure
```

The operating system or parent process can inspect this value.

---

# exit()

```c
#include <stdlib.h>

exit(status);
```

Immediately terminates the program with the specified exit status.

Example:

```c
#include <stdlib.h>

int main(void)
{
    exit(0);
}
```

Before terminating, `exit()` performs normal C runtime cleanup, including:

- Flushes open output streams
    
- Closes/cleans up standard I/O streams
    
- Calls functions registered with `atexit()`
    

---

# atexit()

`atexit()` registers a function to be called automatically when the program performs a **normal exit**.

Syntax:

```c
int atexit(void (*function)(void));
```

Example:

```c
#include <stdio.h>
#include <stdlib.h>

void on_exit_1(void)
{
    printf("Exit handler 1 called!\n");
}

void on_exit_2(void)
{
    printf("Exit handler 2 called!\n");
}

int main(void)
{
    atexit(on_exit_1);
    atexit(on_exit_2);

    printf("About to exit...\n");
}
```

Output:

```text
About to exit...
Exit handler 2 called!
Exit handler 1 called!
```

### Important

Exit handlers are executed in **reverse order of registration**.

```text
Registered:

1 → on_exit_1
2 → on_exit_2

Executed:

2 → on_exit_2
1 → on_exit_1
```

This is **LIFO**:

> Last In, First Out

`atexit()` handlers are called when the program exits normally through:

- `return` from `main()`
    
- `exit()`
    

They are **not** called by:

- `quick_exit()`
    
- `_Exit()`
    
- `abort()`
    

---

# Quick Exit: quick_exit()

```c
#include <stdlib.h>

quick_exit(status);
```

Provides a faster form of normal termination.

Compared with `exit()`:

- Open streams might not be flushed.
    
- Temporary files might not be removed.
    
- `atexit()` handlers are **not** called.
    
- `at_quick_exit()` handlers **are** called.
    

---

# at_quick_exit()

Registers a function to be called when `quick_exit()` is used.

Example:

```c
#include <stdio.h>
#include <stdlib.h>

void on_quick_exit_1(void)
{
    printf("Quick exit handler 1 called!\n");
}

void on_quick_exit_2(void)
{
    printf("Quick exit handler 2 called!\n");
}

void on_exit(void)
{
    printf("Normal exit -- I won't be called!\n");
}

int main(void)
{
    at_quick_exit(on_quick_exit_1);
    at_quick_exit(on_quick_exit_2);

    atexit(on_exit);

    printf("About to quick exit...\n");

    quick_exit(0);
}
```

Output:

```text
About to quick exit...
Quick exit handler 2 called!
Quick exit handler 1 called!
```

Like `atexit()`, `at_quick_exit()` handlers execute in **reverse registration order**.

---

# Normal vs Quick Exit

|Function|Exit handlers|Stream flushing|Cleanup|
|---|---|---|---|
|`return` from `main()`|`atexit()`|Yes|Normal|
|`exit()`|`atexit()`|Yes|Normal|
|`quick_exit()`|`at_quick_exit()`|Might not|Limited|
|`_Exit()`|None|No|None|
|`abort()`|None|Not guaranteed|Abnormal|

---

# Immediate Exit: _Exit()

```c
#include <stdlib.h>

_Exit(status);
```

Terminates the process **immediately**.

It does **not**:

- Call `atexit()` handlers
    
- Call `at_quick_exit()` handlers
    
- Flush streams
    
- Perform normal cleanup
    

Use it when the process must terminate immediately without normal runtime cleanup.

---

# assert()

`assert()` is used to verify that a condition that **should always be true** is actually true.

Header:

```c
#include <assert.h>
```

Syntax:

```c
assert(condition);
```

Example:

```c
int goats = 50;

goats -= 100;

assert(goats >= 0);
```

If the condition is false, the program reports an assertion failure and normally terminates abnormally.

Example:

```text
Assertion `goats >= 0' failed.
Aborted
```

### Purpose

Assertions are mainly used by developers to detect **programming errors** or conditions that should never occur.

They are generally **not intended for normal user input validation**.

---

# Disabling assert()

Assertions can be disabled by defining:

```c
NDEBUG
```

before including `<assert.h>`:

```c
#define NDEBUG
#include <assert.h>
```

This makes:

```c
assert(condition);
```

effectively disappear during compilation.

This is why assertions are commonly used for **debug/development checks**, not essential program logic.

---

# Abnormal Exit: abort()

```c
#include <stdlib.h>

abort();
```

Terminates the program abnormally.

Unlike normal termination, `abort()` does **not** perform normal cleanup.

It:

- Does not call `atexit()` handlers.
    
- Does not call `at_quick_exit()` handlers.
    
- Does not guarantee that streams are flushed.
    
- Raises `SIGABRT`.
    

Conceptually:

```text
abort()
   ↓
SIGABRT
   ↓
Process terminates
```

On Unix-like systems, this may result in a **core dump**, depending on system configuration.

---

# Exit Functions — Mental Model

```text
                    Program
                       │
        ┌──────────────┼───────────────┐
        │              │               │
   return main()     exit()       quick_exit()
        │              │               │
        └──────┬───────┘               │
               ↓                       ↓
          atexit()              at_quick_exit()
          handlers                  handlers
               │                       │
               ↓                       ↓
        Normal cleanup           Limited cleanup
               │                       │
               └──────────┬────────────┘
                          ↓
                    Process exits


        _Exit() ─────────→ Immediate exit
        abort() ─────────→ SIGABRT → Abnormal exit
```

---

# Reverse Engineering Perspective

These functions are particularly useful to recognize during reverse engineering.

### `exit()`

A binary may contain a call such as:

```asm
call exit
```

This indicates an intentional process termination.

### `abort()`

You may see:

```asm
call abort
```

or code that ultimately raises:

```text
SIGABRT
```

This can indicate:

- Failed assertions
    
- Fatal internal errors
    
- Deliberate crash paths
    

### `assert()`

In a debug build, an assertion may ultimately lead to functions such as:

```text
__assert_fail()
      ↓
abort()
      ↓
SIGABRT
```

The exact implementation depends on the compiler and C runtime.

---

# Long-Term Takeaways

- `return 0` from `main()` performs a **normal exit**.
    
- `exit()` performs normal termination and calls `atexit()` handlers.
    
- `atexit()` handlers execute in **reverse registration order (LIFO)**.
    
- `quick_exit()` provides faster termination with less cleanup.
    
- `at_quick_exit()` registers handlers for `quick_exit()`.
    
- `_Exit()` terminates immediately with **no normal cleanup**.
    
- `assert()` checks conditions that should always be true.
    
- `NDEBUG` disables `assert()` checks.
    
- `abort()` performs an **abnormal termination** by raising `SIGABRT`.
    
- In reverse engineering, `exit()`, `abort()`, assertion routines, and termination paths are useful landmarks for understanding **program control flow and error-handling paths**.