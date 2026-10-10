# Using `setjmp()` and `longjmp()`

The `<setjmp.h>` header provides two functions used to save and restore an execution state:

```c
#include <setjmp.h>
```

They are commonly used when execution needs to jump back to an earlier point in the call chain without returning normally through each function.

---

# `jmp_buf`

Before using `setjmp()` and `longjmp()`, a buffer is required to store the information needed to restore the previous execution state.

```c
jmp_buf env;
```

`jmp_buf` is a type defined by `<setjmp.h>`.

You can think of `env` as storing enough state for `longjmp()` to later resume execution from the corresponding `setjmp()`.

---

# `setjmp()`

`setjmp()` saves the current execution environment into a `jmp_buf`.

Syntax:

```c
setjmp(env);
```

The important property of `setjmp()` is that it can appear to return **twice**:

- The first time, execution reaches `setjmp()` normally → returns `0`.
    
- Later, if `longjmp()` jumps back to it → it returns the value supplied by `longjmp()`.
    

Example:

```c
int value = setjmp(env);
```

---

# `longjmp()`

`longjmp()` restores an execution environment previously saved by `setjmp()`.

Syntax:

```c
longjmp(env, value);
```

Instead of continuing normally through the current chain of function calls, execution resumes at the corresponding `setjmp()`.

The `value` becomes the return value observed by `setjmp()`.

---

# Example

```c
#include <stdio.h>
#include <setjmp.h>

jmp_buf env;

void depth2(void)
{
    printf("Entering depth 2\n");

    longjmp(env, 3490);

    printf("Leaving depth 2\n");   // Never executed
}

void depth1(void)
{
    printf("Entering depth 1\n");

    depth2();

    printf("Leaving depth 1\n");   // Never executed
}

int main(void)
{
    switch (setjmp(env)) {

        case 0:
            printf("Calling into functions, setjmp() returned 0\n");

            depth1();

            printf("Returned from functions\n");  // Never executed
            break;

        case 3490:
            printf("Jumped back to main, setjmp() returned 3490\n");
            break;
    }
}
```

Output:

```text
Calling into functions, setjmp() returned 0
Entering depth 1
Entering depth 2
Jumped back to main, setjmp() returned 3490
```

### Execution flow

Initially:

```text
main()
  |
  | setjmp() → 0
  v
depth1()
  |
  v
depth2()
  |
  | longjmp(env, 3490)
  |
  └──────────────► main()
                   |
                   | setjmp() → 3490
                   v
                case 3490
```

Notice that execution does **not** return normally from `depth2()` or `depth1()`.

The jump goes directly back to the point where `setjmp()` saved the environment.

---

# How `setjmp()` and `longjmp()` Work Conceptually

At a low level, a function's execution depends on machine state such as:

- Stack pointer
    
- Instruction pointer
    
- Register state
    
- Other implementation-dependent execution information
    

Conceptually, `setjmp()` records the state necessary to resume execution later.

Then `longjmp()` restores that state and causes execution to continue from the saved `setjmp()` location.

```text
setjmp()
   |
   | Save execution state
   v
jmp_buf
   |
   | ... execution continues ...
   |
longjmp()
   |
   | Restore saved state
   v
setjmp() appears to return again
```

For reverse engineering, this is useful to understand because the behavior involves **changing control flow and restoring execution state**, rather than performing an ordinary function return.

---

# Important: Local Variables

There is a special rule for automatic local variables.

If an automatic local variable is modified after `setjmp()` and before `longjmp()`, its value after the jump is **indeterminate** unless the variable is declared `volatile`.

Example:

```c
int x = 20;

if (setjmp(env) == 0) {
    x = 30;
}
```

If execution later jumps back using `longjmp()`, the value of `x` is not guaranteed to be `30`.

To preserve the value:

```c
volatile int x = 20;

if (setjmp(env) == 0) {
    x = 30;
}
```

Now the value of `x` after the jump is guaranteed to reflect the modification.

### Important detail

The `volatile` requirement matters for automatic local variables whose values are changed between `setjmp()` and `longjmp()`.

It does **not** apply in the same way to:

- `static` variables
    
- `extern` variables
    

---

# What State Does `longjmp()` Restore?

`longjmp()` does not restore the entire state of the C program.

Its purpose is specifically to restore the execution environment saved by the corresponding `setjmp()`.

Things such as the following are **not generally restored**:

- Floating-point status flags
    
- State of open files
    
- Other parts of the C abstract machine's state
    

Therefore, `longjmp()` should not be viewed as creating a complete snapshot of the program.

Think of it more like:

```text
Save enough execution state
          ↓
       setjmp()
          ↓
   Program continues
          ↓
       longjmp()
          ↓
Restore saved execution state
          ↓
Continue from setjmp()
```

---

# Restrictions on `setjmp()`

`setjmp()` has strict rules about where it can appear.

It cannot simply be placed inside an arbitrary complex expression.

For example, this is not valid:

```c
if (x == 12 && setjmp(env) == 0) {
    ...
}
```

The problem is that `setjmp()` can later resume execution after a `longjmp()`, so the language places restrictions on the surrounding expression.

---

# Valid Forms of `setjmp()`

It can be used directly as the controlling expression of a conditional:

```c
if (setjmp(env)) {
    ...
}
```

It can also be used as the controlling expression of a `switch`:

```c
switch (setjmp(env)) {
    ...
}
```

It can be compared with an integer constant when the complete expression controls the conditional:

```c
if (setjmp(env) == 0) {
    ...
}
```

It can also be used with the logical NOT operator:

```c
if (!setjmp(env)) {
    ...
}
```

Or used as a standalone expression:

```c
setjmp(env);
```

and:

```c
(void)setjmp(env);
```

### Key idea

Do not treat `setjmp()` like an ordinary function that can safely be embedded anywhere in a complicated expression.

The C standard places restrictions on its use because `longjmp()` can later resume execution at that point.

---

# When `longjmp()` Is Invalid

Calling `longjmp()` can result in **undefined behavior** in several situations.

### 1. No corresponding `setjmp()`

There must be a valid execution environment previously saved by `setjmp()`.

---

### 2. The `setjmp()` was executed in another thread

The execution environment saved by `setjmp()` must be used in the appropriate thread.

You cannot save an environment in one thread and use `longjmp()` from another thread.

---

### 3. A VLA's scope has already been exited

If `setjmp()` executes while a variable-length array is in scope, you cannot later use `longjmp()` to jump back after execution has already left that VLA's scope.

---

### 4. The function containing `setjmp()` has already returned

The function that executed `setjmp()` must still be active when `longjmp()` occurs.

For example:

```text
main()
  |
  v
function()
  |
  | setjmp()
  |
  v
function() returns
  |
  v
main()
  |
  | longjmp() back to function()
  X
```

This is invalid because the stack frame belonging to `function()` no longer exists.

A similar problem occurs if another `longjmp()` has already moved execution to an earlier point in the call chain, causing the function containing the original `setjmp()` to no longer be active.

---

# `longjmp()` Cannot Return `0`

The second argument to `longjmp()` should not be `0`.

For example:

```c
longjmp(env, 0);
```

If `0` is supplied, `longjmp()` makes `setjmp()` return `1` instead.

This is necessary because:

```c
setjmp(env) == 0
```

specifically identifies the **initial return** from `setjmp()`.

Therefore:

```text
setjmp() initially returns → 0

longjmp(env, 3490)
        ↓
setjmp() returns → 3490
```

But:

```text
longjmp(env, 0)
        ↓
setjmp() returns → 1
```

---

# `longjmp()` and Variable-Length Arrays

Variable-length arrays require additional care.

Example:

```c
void function(void)
{
    int n = 10;
    int array[n];

    ...
}
```

The memory associated with a VLA has a lifetime tied to its scope.

Using `longjmp()` to jump across scopes containing VLAs can cause problems because normal scope cleanup does not happen in the same way as ordinary control flow.

A jump can therefore leave VLA-related storage behind.

This can result in **memory leaks** or other undefined behavior depending on how the jump is performed.

---

# Reverse Engineering Perspective

`setjmp()` and `longjmp()` are particularly interesting from an RE perspective because they provide **non-local control flow**.

Normal function execution:

```text
main()
  ↓
function1()
  ↓
function2()
  ↓
return
  ↓
function1()
  ↓
return
  ↓
main()
```

With `longjmp()`:

```text
main()
  ↓
function1()
  ↓
function2()
  ↓
longjmp()
  │
  └──────────────► main()
```

Instead of unwinding the call chain through normal `ret` instructions, execution is transferred back to a previously saved execution environment.

This is why understanding `setjmp()`/`longjmp()` is useful when analyzing unusual control-flow patterns in compiled binaries.

---

# Mental Model

```text
setjmp()
   │
   │ Save execution environment
   ▼
jmp_buf
   │
   │ Program continues normally
   ▼
longjmp()
   │
   │ Restore saved environment
   ▼
setjmp() executes again
   │
   └── returns supplied value
```

---

# Long-Term Takeaways

- `setjmp()` saves an execution environment in a `jmp_buf`.
    
- `longjmp()` restores that environment and transfers execution back to the corresponding `setjmp()`.
    
- The first return from `setjmp()` is `0`.
    
- After `longjmp()`, `setjmp()` returns the value supplied to `longjmp()`.
    
- `longjmp()` with a value of `0` causes `setjmp()` to return `1`.
    
- `longjmp()` does **not** restore the entire state of the program.
    
- Modified automatic local variables may have indeterminate values after `longjmp()` unless they are `volatile`.
    
- The function containing `setjmp()` must still be active when `longjmp()` is performed.
    
- `setjmp()` cannot be placed arbitrarily inside complex expressions.
    
- Jumping across VLA scopes has special restrictions and can lead to undefined behavior.
    
- From an RE perspective, `setjmp()`/`longjmp()` provide a mechanism for **non-local control flow**, allowing execution to resume at an earlier point in the call chain.