# The Outside Environment in C

Programs do not run in isolation. They interact with the **outside environment** through things like:

- Command-line arguments
    
- Exit status
    
- Environment variables
    

These are especially important in **Linux, scripting, automation, and reverse engineering**.

---

# Command-Line Arguments

Command-line arguments are values given to a program when it is started.

Example:

```bash
./foo i like turtles
```

C receives these through:

```c
int main(int argc, char *argv[])
```

---

# `argc`

`argc` means **argument count**.

It tells you how many command-line arguments were provided.

For:

```bash
./foo i like turtles
```

we have:

```text
argc = 4
```

Because there are four arguments:

```text
0 → ./foo
1 → i
2 → like
3 → turtles
```

---

# `argv`

`argv` means **argument vector**.

It is an array of strings containing the command-line arguments.

Example:

```c
#include <stdio.h>

int main(int argc, char *argv[])
{
    for (int i = 0; i < argc; i++) {
        printf("arg %d: %s\n", i, argv[i]);
    }
}
```

Running:

```bash
./foo i like turtles
```

Output:

```text
arg 0: ./foo
arg 1: i
arg 2: like
arg 3: turtles
```

### Important

`argv[0]` is normally the **program name/path**.

The actual user-supplied arguments begin at:

```text
argv[1]
```

---

# Mental Model of `argc` and `argv`

```text
./foo i like turtles
  │   │   │    │
  ▼   ▼   ▼    ▼

argv[0] argv[1] argv[2] argv[3]

argc = 4
```

Think:

```text
argc → How many?
argv → What are they?
```

---

# Command-Line Arguments and Pointers

This:

```c
int main(int argc, char *argv[])
```

is equivalent to:

```c
int main(int argc, char **argv)
```

Because function parameters treat array notation and pointer notation equivalently.

Similarly:

```c
void foo(char a[])
```

is equivalent to:

```c
void foo(char *a)
```

---

# `argv[i]` vs `*(argv + i)`

These are equivalent:

```c
argv[i]
```

and:

```c
*(argv + i)
```

For example:

```c
argv[2]
```

means:

```c
*(argv + 2)
```

This is important for understanding C pointers and reverse engineering.

---

# `argv` Memory Model

Conceptually:

```text
argv
 │
 ▼
┌───────────────┐
│ argv[0] ──────┼──► "./foo"
├───────────────┤
│ argv[1] ──────┼──► "123"
├───────────────┤
│ argv[2] ──────┼──► "456"
├───────────────┤
│ argv[3] ──────┼──► NULL
└───────────────┘
```

`argv` is therefore:

```text
char **
```

because:

```text
argv
 │
 ▼
array of char *
 │
 ▼
each char * points to a string
```

---

# The Last `argv` Is `NULL`

The C standard guarantees that:

```c
argv[argc] == NULL
```

So if:

```text
argc = 4
```

then:

```text
argv[0] → "./foo"
argv[1] → "i"
argv[2] → "like"
argv[3] → "turtles"
argv[4] → NULL
```

This allows you to iterate through arguments without explicitly using `argc`.

---

# Iterating Using `argv` Directly

Example:

```c
for (char **p = argv + 1; *p != NULL; p++) {
    printf("%s\n", *p);
}
```

Here:

```c
argv + 1
```

points to `argv[1]`.

And:

```c
*p
```

is the current string.

The loop continues until:

```c
*p == NULL
```

---

# Example: Adding Numbers from Arguments

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
    int total = 0;

    for (int i = 1; i < argc; i++) {
        int value = atoi(argv[i]);

        total += value;
    }

    printf("%d\n", total);
}
```

Run:

```bash
./sum 10 20 30
```

Result:

```text
60
```

The loop starts at `1` because:

```text
argv[0] = program name
argv[1] = first actual argument
```

### Reverse Engineering Connection

Command-line arguments are often used by real programs for:

- File paths
    
- Configuration options
    
- Flags
    
- Input values
    
- Modes of operation
    

When reversing a binary, understanding how `argc` and `argv` are used can reveal **how the program expects to be executed**.

---

# Exit Status

When you run a program, it is usually launched by another program, such as a **shell**.

For example:

```bash
./foo
```

The basic process is:

```text
Shell
  │
  │ launches
  ▼
Program
  │
  │ runs
  ▼
Program terminates
  │
  │ returns exit status
  ▼
Shell
```

The shell can inspect the program's exit status.

---

# Exit Status Meaning

The most important values are:

|Exit Status|Meaning|
|---|---|
|`0`|Success|
|Non-zero|Failure/error|

C provides:

```c
EXIT_SUCCESS
EXIT_FAILURE
```

from:

```c
#include <stdlib.h>
```

Typically:

```c
EXIT_SUCCESS
```

is equivalent to:

```c
0
```

---

# Returning an Exit Status

Example:

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
    if (argc != 3) {
        printf("usage: mult x y\n");
        return EXIT_FAILURE;
    }

    printf("%d\n", atoi(argv[1]) * atoi(argv[2]));

    return EXIT_SUCCESS;
}
```

Run incorrectly:

```bash
./mult
```

Output:

```text
usage: mult x y
```

The program returns:

```text
EXIT_FAILURE
```

Run correctly:

```bash
./mult 3 4
```

Output:

```text
12
```

The program returns:

```text
EXIT_SUCCESS
```

---

# Checking Exit Status in Bash

After running a program, Bash stores its exit status in:

```bash
$?
```

Example:

```bash
./mult 3 4
echo $?
```

Output:

```text
12
0
```

`0` means success.

Another example:

```bash
./mult
echo $?
```

Output:

```text
usage: mult x y
1
```

The `1` indicates failure.

---

# Why Exit Status Matters

Exit statuses allow **scripts and other programs to make decisions**.

Example:

```bash
./backup

if [ $? -eq 0 ]; then
    echo "Backup succeeded"
else
    echo "Backup failed"
fi
```

So:

```text
Program
   │
   ▼
Exit status
   │
   ▼
Shell/script decides what to do next
```

---

# Other Exit Status Values

On Unix-like systems:

```text
0       → success
1–255   → generally failure/error codes
```

Different numbers can represent different errors.

For example:

```text
1 → General error
2 → Invalid usage
3 → Configuration error
4 → File error
```

The exact meanings are application-specific unless a convention is defined.

On Linux, process exit status is effectively represented as an **8-bit value**, so values outside `0–255` do not give you additional distinct exit codes.

---

# Environment Variables

Environment variables are **key-value pairs provided to a process by its environment**.

Examples:

```text
PATH=/usr/bin:/bin
HOME=/home/user
USER=chinmay
```

Programs can read these values.

---

# Reading an Environment Variable

C provides:

```c
getenv()
```

Example:

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    char *val = getenv("FROTZ");

    if (val == NULL) {
        printf("Cannot find the FROTZ environment variable\n");
        return EXIT_FAILURE;
    }

    printf("Value: %s\n", val);
}
```

If `FROTZ` doesn't exist:

```text
Cannot find the FROTZ environment variable
```

---

# Setting an Environment Variable

In Bash:

```bash
export FROTZ="C is awesome!"
```

Then:

```bash
./foo
```

Output:

```text
Value: C is awesome!
```

In Windows CMD:

```cmd
set FROTZ=value
```

In PowerShell:

```powershell
$Env:FROTZ="value"
```

---

# `getenv()` Mental Model

```text
Environment
┌──────────────────────┐
│ PATH=/usr/bin        │
│ HOME=/home/chinmay   │
│ FROTZ=C is awesome!  │
└──────────┬───────────┘
           │
           │ getenv("FROTZ")
           ▼
        "C is awesome!"
```

If the variable doesn't exist:

```c
getenv("FROTZ")
```

returns:

```c
NULL
```

---

# Setting Environment Variables from C

On Unix-like systems, functions include:

```c
putenv()
setenv()
unsetenv()
```

These are **not standard C**, so they are less portable.

On Windows, you may encounter:

```c
_putenv()
```

For simply reading variables, prefer:

```c
getenv()
```

because it is portable.

---

# Accessing All Environment Variables

On Unix-like systems, you can access the environment through:

```c
extern char **environ;
```

Example:

```c
#include <stdio.h>

extern char **environ;

int main(void)
{
    for (char **p = environ; *p != NULL; p++) {
        printf("%s\n", *p);
    }
}
```

This prints entries such as:

```text
PATH=/usr/bin:/bin
HOME=/home/chinmay
USER=chinmay
...
```

---

# `environ` Memory Model

Conceptually:

```text
environ
   │
   ▼
┌──────────────────────┐
│ char * ──────────────┼──► "PATH=/usr/bin"
├──────────────────────┤
│ char * ──────────────┼──► "HOME=/home/chinmay"
├──────────────────────┤
│ char * ──────────────┼──► "USER=chinmay"
├──────────────────────┤
│ char * ──────────────┼──► NULL
└──────────────────────┘
```

Again, this is:

```text
char **
```

because it points to an array of `char *`.

---

# `environ` vs `getenv()`

Use:

```c
getenv("PATH");
```

when you want **one specific variable**.

Use:

```c
environ
```

when you need to **iterate through the entire environment** on systems that provide it.

### Portability

```text
getenv()
   ↓
Standard C → Portable

environ
   ↓
Unix-like → Non-standard
```

---

# Environment as a Parameter to `main()`

Some Unix-like systems also allow:

```c
int main(int argc, char **argv, char **env)
```

Here:

```text
argc → argument count
argv → command-line arguments
env  → environment variables
```

Example:

```c
#include <stdio.h>

int main(int argc, char **argv, char **env)
{
    (void)argc;
    (void)argv;

    for (char **p = env; *p != NULL; p++) {
        printf("%s\n", *p);
    }
}
```

However, this form is **not standard C** and is less portable than using `getenv()`.

---

# Reverse Engineering Perspective

The outside environment is extremely important when reversing programs.

A program can receive input from several places:

```text
                Program
                   ▲
        ┌──────────┼──────────┐
        │          │          │
     argv      Environment   Files
        │          │
        ▼          ▼
   Arguments   getenv()
```

When analyzing a binary, look for:

### Command-line arguments

```c
argv[1]
argv[2]
```

These may control:

- Input files
    
- Commands
    
- Options
    
- Modes
    
- Configuration
    

### Environment variables

```c
getenv("PATH")
getenv("HOME")
getenv("USER")
```

These may influence program behavior.

### Exit status

```c
return 0;
return 1;
```

This tells the shell or calling process whether an operation succeeded.

---

# Important Pointer Connections

You will see the same pointer pattern repeatedly:

```c
char **argv
```

means:

```text
char **
  │
  ▼
pointer to char *
  │
  ▼
pointer to characters
  │
  ▼
string
```

And:

```c
argv[i]
```

is equivalent to:

```c
*(argv + i)
```

Likewise:

```c
*p
```

gives the current `char *`.

This is why code such as:

```c
for (char **p = argv + 1; *p != NULL; p++)
```

works.

---

# Mental Model

```text
                Outside Environment
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
 Command Line      Environment      Files
 Arguments         Variables
        │              │
        ▼              ▼
      argv          getenv()
        │
        ▼
      Program
        │
        ▼
   Exit Status
        │
        ▼
      Shell
```

---

# Long-Term Takeaways

- `argc` = **number of command-line arguments**.
    
- `argv` = **array of strings containing command-line arguments**.
    
- `argv[0]` is normally the program name.
    
- Actual user arguments normally start at `argv[1]`.
    
- `argv[argc]` is guaranteed to be `NULL`.
    
- `char *argv[]` and `char **argv` are equivalent in a function parameter.
    
- `argv[i]` is equivalent to `*(argv + i)`.
    
- `0` normally means **successful termination**.
    
- Non-zero exit statuses generally indicate **failure**.
    
- Bash exposes the previous exit status through `$?`.
    
- `getenv()` retrieves an environment variable.
    
- `getenv()` returns `NULL` when the variable doesn't exist.
    
- `environ` can be used on Unix-like systems to iterate over all environment variables.
    
- `environ` and `char **env` in `main()` are **not standard C**.
    
- Environment variables and command-line arguments are important sources of external input.
    
- In reverse engineering, understanding `argc`, `argv`, environment variables, and exit codes helps you understand **how a binary interacts with its operating environment**.