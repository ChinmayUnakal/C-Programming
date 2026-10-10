# Signal Handling

Signals are a mechanism used by the operating system or program to notify a process that an **event has occurred**.

Header:

```
#include <signal.h>
```

---

# What Are Signals?

A signal is an **asynchronous notification** sent to a process.

Common C-standard signals:

|Signal|Description|
|---|---|
|`SIGABRT`|Abnormal termination, usually from `abort()`|
|`SIGFPE`|Arithmetic/floating-point exception|
|`SIGILL`|Illegal instruction|
|`SIGINT`|Interrupt, commonly caused by `Ctrl+C`|
|`SIGSEGV`|Invalid memory access|
|`SIGTERM`|Termination request|

The exact signals available can depend on the operating system.

---

# Signal Actions

A program can configure how it responds to a signal using `signal()`.

Possible actions:

```
Handler function → Run custom code
SIG_IGN         → Ignore the signal
SIG_DFL         → Use default behavior
```

---

# signal()

Syntax:

```
signal(signal, handler);
```

Example:

```
#include <stdio.h>
#include <signal.h>

int main(void)
{
    char s[1024];

    signal(SIGINT, SIG_IGN);

    printf("Try hitting ^C... (hit RETURN to exit)\n");

    fgets(s, sizeof s, stdin);
}
```

Here:

```
signal(SIGINT, SIG_IGN);
```

tells the program to **ignore `SIGINT`**.

Therefore, pressing:

```
Ctrl+C
```

will not terminate the program.

---

# Writing a Signal Handler

Instead of ignoring a signal, you can provide your own handler.

Handler format:

```
void handler(int signum)
{
    // Handle signal
}
```

Example:

```
void sigint_handler(int signum)
{
    // Handle SIGINT
}
```

Register it:

```
signal(SIGINT, sigint_handler);
```

When `SIGINT` occurs:

```
SIGINT
   ↓
sigint_handler()
```

---

# signal() Function Type

The full type of `signal()` is complicated:

```
void (*signal(int sig, void (*func)(int)))(int);
```

Conceptually:

```
signal()
   │
   ├── takes a signal number
   ├── takes a pointer to a handler function
   │
   └── returns a pointer to the previous handler
```

A handler has this type:

```
void (*handler)(int);
```

Meaning:

> Pointer to a function that takes an `int` and returns `void`.

---

# Previous Signal Handler

`signal()` returns the previous handler.

Example:

```
void (*old_handler)(int);

old_handler = signal(SIGINT, sigint_handler);
```

This allows the program to save the previous signal behavior.

---

# SIG_IGN

`SIG_IGN` tells the program to **ignore** a signal.

Example:

```
signal(SIGINT, SIG_IGN);
```

Effect:

```
Ctrl+C
   ↓
SIGINT
   ↓
Ignored
```

---

# SIG_DFL

`SIG_DFL` restores the signal's **default behavior**.

Example:

```
signal(SIGINT, SIG_DFL);
```

For `SIGINT`, the default behavior normally terminates the process.

---

# Signal Handler Resetting

With the C `signal()` interface, the signal disposition may be reset to `SIG_DFL` before the handler executes.

Therefore, portable code may reset the handler inside itself:

```
void sigint_handler(int signum)
{
    signal(SIGINT, sigint_handler);
}
```

This allows the handler to continue handling future `SIGINT` signals.

### Important

This behavior is one reason why `signal()` is considered limited and difficult to use portably.

---

# What Can a Signal Handler Safely Do?

Signal handlers have **very strict restrictions**.

Portable C code should keep signal handlers extremely simple.

### Avoid:

```
printf();
malloc();
free();
fopen();
exit();
```

and other standard-library functions unless specifically permitted by the relevant platform/API.

The C standard does not generally guarantee that arbitrary library functions are safe to call from a signal handler.

---

# Static and Thread-Local Data

Signal handlers also have restrictions on accessing objects with:

- Static storage duration
- Thread storage duration

A commonly used mechanism for communicating between a signal handler and normal program code is:

```
volatile sig_atomic_t
```

---

# sig_atomic_t

`sig_atomic_t` is an integer type that can be accessed atomically with respect to signal handling.

Example:

```
volatile sig_atomic_t flag = 0;
```

A handler can set it:

```
void handler(int signum)
{
    flag = 1;
}
```

Then normal program code can check it:

```
while (!flag)
{
    /* continue */
}
```

The important idea is:

```
Signal handler
      │
      │ set flag
      ▼
volatile sig_atomic_t
      │
      ▼
Main program checks flag
```

The implementation provides limits for the type through:

```
SIG_ATOMIC_MIN
SIG_ATOMIC_MAX
```

---

# Simple Signal Flag Pattern

A minimal pattern is:

```
#include <signal.h>

volatile sig_atomic_t interrupted = 0;

void handler(int signum)
{
    (void)signum;

    interrupted = 1;
}

int main(void)
{
    signal(SIGINT, handler);

    while (!interrupted)
    {
        /* Do work */
    }
}
```

The handler does very little:

```
SIGINT
  ↓
handler()
  ↓
set flag
  ↓
return
  ↓
main() notices flag
```

This is much safer than performing complex operations inside the handler.

---

# Signal Handler vs Normal Code

A useful mental model:

```
Normal Program
      │
      │ running
      ▼
   Signal arrives
      │
      ▼
Signal Handler
      │
      │ do minimal work
      ▼
   Return
      │
      ▼
Normal Program continues
```

Signals can arrive **asynchronously**, meaning they may interrupt normal program execution at unexpected times.

---

# Why Complex Signal Handlers Are Dangerous

Consider:

```
void handler(int signum)
{
    printf("Signal received!\n");
    count++;
    exit(0);
}
```

This looks reasonable but is problematic for portable C because the handler is performing operations that are not generally guaranteed to be safe in a signal-handler context.

### Better approach

```
volatile sig_atomic_t received = 0;

void handler(int signum)
{
    received = 1;
}
```

Then let normal program execution perform the actual work.

---

# Example: Handling Ctrl+C

```
#include <stdio.h>
#include <signal.h>

volatile sig_atomic_t interrupted = 0;

void sigint_handler(int signum)
{
    (void)signum;

    interrupted = 1;
}

int main(void)
{
    signal(SIGINT, sigint_handler);

    printf("Press Ctrl+C to stop.\n");

    while (!interrupted)
    {
        /* Program work */
    }

    printf("Stopping normally.\n");
}
```

The important design is:

```
Handler:
    signal arrives
        ↓
    set flag

Main:
    sees flag
        ↓
    performs cleanup
        ↓
    exits normally
```

---

# SIGINT and Ctrl+C

On many systems:

```
Ctrl+C
   ↓
SIGINT
   ↓
Process receives signal
```

Default behavior:

```
SIGINT → terminate process
```

But you can change it:

```
signal(SIGINT, SIG_IGN);
```

or:

```
signal(SIGINT, sigint_handler);
```

---

# One-Time Handling

You can also handle a signal once and then restore its default behavior:

```
void sigint_handler(int signum)
{
    (void)signum;

    signal(SIGINT, SIG_DFL);
}
```

Initially:

```
SIGINT
  ↓
custom handler
```

Handler changes it to:

```
SIG_DFL
```

So the next `SIGINT` uses the default behavior.

Conceptually:

```
First Ctrl+C
     ↓
Custom handler
     ↓
Restore SIG_DFL

Second Ctrl+C
     ↓
Default SIGINT behavior
     ↓
Process terminates
```

---

# C signal() vs OS Signal APIs

The C `signal()` function is **portable but limited**.

Operating systems generally provide more powerful signal APIs.

For example, Unix-like systems commonly use:

```
sigaction()
```

instead of `signal()` for serious signal handling.

These OS-specific APIs generally provide much finer control over:

- Signal behavior
- Signal masks
- Handler configuration
- Restart behavior
- Signal information

---

# Reverse Engineering Perspective

Signals are important when analyzing crashes and process behavior.

Common signals to recognize:

```
SIGSEGV → Invalid memory access
SIGILL  → Illegal instruction
SIGFPE  → Arithmetic exception
SIGABRT → Program deliberately aborted
SIGINT  → User interrupt
SIGTERM → Termination request
```

For example:

```
Invalid memory access
        ↓
     SIGSEGV
        ↓
Signal handler / OS
        ↓
Process termination
```

In a debugger, signals can therefore be important clues when analyzing:

- Crashes
- Memory corruption
- Invalid instructions
- Assertions
- Deliberate termination
- Anti-debugging behavior

---

# Important RE Connection

When reversing a native binary, you may encounter code that eventually performs:

```
raise()
kill()
abort()
signal()
sigaction()
```

These can indicate **signal generation, handling, or termination paths**.

A `SIGSEGV`, for example, can be the result of an actual memory bug—but a program can also deliberately manipulate signal behavior, so you should examine the surrounding control flow rather than assuming every crash is accidental.

---

# Portable Signal-Handling Rule

For portable C:

> **Keep signal handlers extremely simple.**

Prefer:

```
volatile sig_atomic_t flag;
```

and let the main program perform the actual work.

Avoid turning the handler into a mini-program.

---

# Mental Model

```
                 Signal
                    │
                    ▼
              Process receives
                    │
          ┌─────────┼─────────┐
          │         │         │
       SIG_IGN   Handler    SIG_DFL
          │         │         │
       Ignore    Custom     Default
                  action     action
                    │
                    ▼
             Return/terminate
```

---

# Long-Term Takeaways

- A **signal** is an asynchronous notification delivered to a process.
- `<signal.h>` provides the standard C signal interface.
- Common signals include `SIGINT`, `SIGSEGV`, `SIGILL`, `SIGFPE`, `SIGABRT`, and `SIGTERM`.
- `signal()` configures how a program responds to a signal.
- `SIG_IGN` means **ignore** the signal.
- `SIG_DFL` means **use the default behavior**.
- A custom handler has the form:

```
void handler(int signum);
```

- `signal()` returns a pointer to the **previous handler**.
- `signal()` may reset a handler to `SIG_DFL`, so resetting the handler may be necessary when using this interface.
- Signal handlers have **strict restrictions** on what they can safely do.
- Avoid calling arbitrary standard-library functions such as `printf()` from a portable signal handler.
- `volatile sig_atomic_t` is commonly used to communicate simple state from a signal handler to normal code.
- For serious Unix/Linux signal handling, prefer **`sigaction()`** over `signal()`.
- In reverse engineering, signals are important for understanding **crashes, exception-like behavior, termination paths, and process control**.