# `goto` Statement

`goto` transfers execution directly to a **label** within the same function.

Syntax:

```c
goto label;

/* ... */

label:
    statement;
```

Example:

```c
#include <stdio.h>

int main(void)
{
    printf("One\n");
    printf("Two\n");

    goto skip;

    printf("Three\n");

skip:
    printf("Five!\n");
}
```

Output:

```text
One
Two
Five!
```

`goto` skips everything between the `goto` and the target label.

---

# Labels

A label identifies a location in a function:

```c
label:
    statement;
```

Labels have **function scope**.

Execution does not stop at a label; labels are simply targets for `goto`.

You can jump:

- Forward
    
- Backward
    

Example:

```c
infinite_loop:

printf("Hello, world!\n");

goto infinite_loop;
```

This creates an infinite loop.

---

# `goto` and Loops

`continue` and `break` only affect the **innermost enclosing loop**.

Example:

```c
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {

        printf("%d, %d\n", i, j);

        continue;
    }
}
```

`continue` only continues the `j` loop.

Similarly:

```c
break;
```

only exits the innermost loop.

---

# Simulating a Labeled `continue`

C does not support labeled `continue`.

`goto` can be used when you need to continue an **outer loop**:

```c
for (int i = 0; i < 3; i++) {

    for (int j = 0; j < 3; j++) {

        for (int k = 0; k < 3; k++) {

            printf("%d, %d, %d\n", i, j, k);

            goto continue_i;
        }
    }

continue_i:
    ;
}
```

The `goto` jumps directly to the point where the outer loop continues.

The `;` provides an empty statement for the label.

---

# Using `goto` for Error Handling

`goto` can provide a clean way to escape deeply nested code.

Example:

```c
for (...) {
    for (...) {
        while (...) {
            do {

                if (error)
                    goto bail;

            } while (...);
        }
    }
}

bail:
    /* Cleanup */
```

Without `goto`, you may need multiple flags and repeated checks to escape every level of nesting.

---

# Multi-Level Cleanup

`goto` is commonly useful when several resources must be initialized and cleaned up.

Example:

```c
if (init_system_1() == -1)
    goto shutdown;

if (init_system_2() == -1)
    goto shutdown_1;

if (init_system_3() == -1)
    goto shutdown_2;

if (init_system_4() == -1)
    goto shutdown_3;

do_main_thing();

shutdown_system4();

shutdown_3:
    shutdown_system3();

shutdown_2:
    shutdown_system2();

shutdown_1:
    shutdown_system1();

shutdown:
    printf("All subsystems shut down.\n");
```

### Important

Cleanup occurs in the **reverse order of initialization**.

```text
Initialize:
1 → 2 → 3 → 4

Cleanup:
4 → 3 → 2 → 1
```

This pattern is particularly useful in low-level/system programming.

---

# `goto` and Tail Call Optimization

A recursive function can sometimes be transformed into a loop-like structure using `goto`.

Example:

```c
int factorial(int n, int a)
{
    if (n == 0)
        return a;

    return factorial(n - 1, a * n);
}
```

The recursive call:

```c
return factorial(n - 1, a * n);
```

is a **tail call** because nothing needs to happen after the recursive call returns.

Conceptually, it can be transformed into:

```c
int factorial(int n, int a)
{
tco:
    if (n == 0)
        return a;

    int next_n = n - 1;
    int next_a = a * n;

    n = next_n;
    a = next_a;

    goto tco;
}
```

This removes the need for a new recursive stack frame each iteration.

---

# Why Temporary Variables Matter

This:

```c
a *= n;
n -= 1;
goto tco;
```

works because `a` uses the old value of `n`.

But carelessly reversing the operations:

```c
n -= 1;
a *= n;
```

changes the calculation.

Temporary variables make the intended next state explicit:

```c
int next_n = n - 1;
int next_a = a * n;

n = next_n;
a = next_a;
```

Compilers can often optimize such code automatically.

---

# `goto` and Interrupted System Calls

On Unix-like systems, a system call can sometimes be interrupted by a signal.

For example:

```c
byte_count = read(0, buf, sizeof(buf) - 1);
```

If interrupted, the call may fail with:

```text
errno == EINTR
```

The program may retry it:

```c
retry:

byte_count = read(0, buf, sizeof(buf) - 1);

if (byte_count == -1) {
    if (errno == EINTR)
        goto retry;
}
```

This is **Unix-specific** and outside the C standard.

Some Unix-like systems also provide `SA_RESTART` with `sigaction()` to request automatic restarting of certain interrupted system calls.

---

# `goto` and Thread Preemption

`goto` can also be used to retry acquiring multiple locks.

Conceptually:

```c
retry:

pthread_mutex_lock(L1);

if (pthread_mutex_trylock(L2) != 0) {

    pthread_mutex_unlock(L1);

    goto retry;
}

save_the_day();

pthread_mutex_unlock(L2);
pthread_mutex_unlock(L1);
```

If the second lock cannot be acquired:

```text
Lock L1
   ↓
Try L2
   ↓
L2 unavailable
   ↓
Unlock L1
   ↓
Retry
```

This can help avoid certain **deadlock situations** when acquiring multiple resources.

---

# `goto` and Variable Scope

Be careful when jumping across variable declarations and initialization.

Example:

```c
goto label;

{
    int x = 12345;

label:
    printf("%d\n", x);
}
```

The jump enters `x`'s scope but skips its initialization.

Therefore, `x` has an **indeterminate value** when accessed.

The safer approach is to ensure initialization happens after the label:

```c
goto label;

{
    int x;

label:
    x = 12345;

    printf("%d\n", x);
}
```

---

# Jumping Into a Variable's Scope

A `goto` can create situations where a variable is technically in scope but its initialization has been skipped.

Example:

```c
{
    int x = 10;

label:
    printf("%d\n", x);
}

goto label;
```

The first execution is fine.

After jumping back to `label`, execution enters `x`'s scope **after its initialization**, so the value is indeterminate.

The important rule is:

> Do not use `goto` to jump past a variable's initialization.

---

# `goto` and VLAs

VLAs have an additional restriction.

You **cannot jump from outside the scope of a VLA into its scope**.

Invalid:

```c
int x = 10;

goto label;

{
    int v[x];

label:
    printf("Hi!\n");
}
```

The jump enters the VLA's scope without executing its declaration.

This is rejected because the VLA has not been properly allocated.

---

# Safe VLA Jump

You can jump to a location **before** the VLA declaration:

```c
int x = 10;

goto label;

{
label:
    ;

    int v[x];

    printf("Hi!\n");
}
```

Now execution reaches the VLA declaration first:

```text
label
  ↓
VLA declaration
  ↓
VLA allocated
  ↓
use VLA
```

This preserves the VLA's proper lifetime.

---

# `goto` and VLA — RE Perspective

This is important when reversing or analyzing compiler-generated control flow.

A VLA requires runtime stack allocation:

```text
Calculate VLA size
       ↓
Allocate stack space
       ↓
Use VLA
       ↓
Leave scope
       ↓
Release stack space
```

A jump that bypasses the allocation could produce invalid program state, which is why C restricts such control flow.

---

# When Is `goto` Useful?

`goto` is generally best suited for **low-level control flow and cleanup**, not ordinary program logic.

Good use cases:

- Multi-level error handling
    
- Resource cleanup
    
- Breaking out of deeply nested structures
    
- Retrying operations
    
- Low-level state-machine/control-flow implementations
    

Avoid using it when a simple:

```c
if
for
while
break
continue
```

makes the control flow clearer.

---

# Reverse Engineering Perspective

`goto` is especially useful conceptually when reading assembly.

A C:

```c
goto label;
```

may become a direct jump:

```asm
jmp label
```

Conditional control flow:

```c
if (condition)
    goto label;
```

may become:

```asm
cmp ...
je label
```

or:

```asm
jne label
```

Therefore, understanding `goto` helps you map:

```text
C control flow
      ↓
Labels
      ↓
Conditional/unconditional jumps
      ↓
Assembly control-flow graph
```

This is particularly useful when analyzing:

- Compiler-generated loops
    
- Error paths
    
- Cleanup paths
    
- State machines
    
- Optimized code
    
- Malware/control-flow logic
    

---

# Mental Model

```text
                 goto
                  │
                  ▼
            ┌───────────┐
            │   label   │
            └─────┬─────┘
                  │
                  ▼
            Continue execution
```

For cleanup:

```text
Initialize 1
     ↓
Initialize 2
     ↓
Initialize 3
     ↓
Error
     │
     ▼
Cleanup 3
     ↓
Cleanup 2
     ↓
Cleanup 1
```

---

# Long-Term Takeaways

- `goto` transfers execution directly to a **label** in the same function.
    
- Labels have **function scope**.
    
- `goto` can jump forward or backward.
    
- Labels themselves do not affect normal execution.
    
- `break` and `continue` only affect the **innermost loop**.
    
- `goto` can simulate labeled `break`/`continue` behavior.
    
- `goto` can simplify **multi-level error handling and cleanup**.
    
- Resource cleanup should generally happen in the **reverse order of initialization**.
    
- Tail-recursive code can conceptually be transformed into a loop using parameter updates + `goto`.
    
- On Unix-like systems, `goto` is sometimes used to retry system calls interrupted with `EINTR`.
    
- `goto` can also be useful for retrying resource acquisition in multithreaded code.
    
- Jumping past variable initialization can result in **indeterminate values**.
    
- You cannot jump from outside into the scope of a **VLA**.
    
- Understanding `goto` maps directly to understanding assembly `jmp`/conditional jumps and **control-flow graphs**.