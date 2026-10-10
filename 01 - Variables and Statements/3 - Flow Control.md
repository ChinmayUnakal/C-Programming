
## What is Flow Control?

Flow control determines **the order in which statements are executed**.

Normally, a program executes from **top to bottom**.

Flow control allows a program to:

- Make decisions
    
- Repeat code
    
- Choose between multiple paths
    

---

# if Statement

Executes a block of code **only if a condition is true**.

Syntax:

```c
if (condition)
{
    // code
}
```

Example:

```c
if (age >= 18)
{
    printf("Adult");
}
```

---

# if-else Statement

Chooses between **two execution paths**.

Syntax:

```c
if (condition)
{
    // executes if true
}
else
{
    // executes if false
}
```

Example:

```c
if (x % 2 == 0)
{
    printf("Even");
}
else
{
    printf("Odd");
}
```

---

# else-if Ladder

Checks **multiple conditions** in order.

The first true condition is executed.

Syntax:

```c
if (condition1)
{
}
else if (condition2)
{
}
else
{
}
```

Example:

```c
if (marks >= 90)
    printf("A");
else if (marks >= 75)
    printf("B");
else
    printf("C");
```

---

# while Loop

Repeats a block **while a condition is true**.

The condition is checked **before** each iteration.

Syntax:

```c
while (condition)
{
    // code
}
```

Example:

```c
int i = 1;

while (i <= 5)
{
    printf("%d\n", i);
    i++;
}
```

May execute **zero times**.

---

# do-while Loop

Repeats a block **while a condition is true**.

The condition is checked **after** each iteration.

Syntax:

```c
do
{
    // code
}
while (condition);
```

Example:

```c
int i = 1;

do
{
    printf("%d\n", i);
    i++;
}
while (i <= 5);
```

Always executes **at least once**.

---

# for Loop

Used when the number of iterations is known.

Syntax:

```c
for (initialization; condition; update)
{
    // code
}
```

Example:

```c
for (int i = 0; i < 5; i++)
{
    printf("%d\n", i);
}
```

Components:

- Initialization → Runs once
    
- Condition → Checked before each iteration
    
- Update → Runs after each iteration
    

---

# switch Statement

Selects one execution path from multiple cases.

Syntax:

```c
switch (value)
{
    case 1:
        // code
        break;

    case 2:
        // code
        break;

    default:
        // code
}
```

Example:

```c
switch (grade)
{
    case 'A':
        printf("Excellent");
        break;

    case 'B':
        printf("Good");
        break;

    default:
        printf("Invalid");
}
```

`break` prevents execution from continuing into the next case.

---

# Reverse Engineering Perspective (x86-64 Assembly)

## if Statement

C:

```c
if (a > b)
```

Assembly:

```asm
cmp eax, ebx
jle skip

; if body

skip:
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

## if-else Statement

Implemented using **conditional** and **unconditional** jumps.

```asm
cmp eax, ebx
jle else

; if block

jmp end

else:
; else block

end:
```

---

## Loops

All loops eventually become:

```text
Compare
      │
      ▼
Conditional Jump
      │
      ▼
Loop Body
      │
      ▼
Jump Back
```

Example:

```asm
loop_start:

cmp eax, 5
jge end

; body

inc eax
jmp loop_start

end:
```

The CPU has **no dedicated loop construct**—only comparisons and jumps.

---

## switch Statement

Small switches may compile into several comparisons.

Large switches often become a **jump table**, allowing direct jumps to the correct case.

Example:

```text
Index
 │
 ▼
Jump Table
 │
 ▼
Case Block
```

Jump tables are very common in optimized binaries.

---

# Mental Model

```text
Program Starts
      │
      ▼
Decision?
      │
 ┌────┴────┐
 │         │
Yes        No
 │         │
 ▼         ▼
Execute   Skip
```

Loops:

```text
Initialize
     │
     ▼
Condition
     │
 ┌───┴───┐
 │       │
True    False
 │       │
 ▼       ▼
Body    Exit
 │
 ▼
Update
 │
 └──────────► Condition
```

---

# Long-Term Takeaways

- Flow control changes the normal top-to-bottom execution of a program.
    
- `if` chooses one path.
    
- `if-else` chooses between two paths.
    
- `else-if` checks multiple conditions.
    
- `while` checks the condition first.
    
- `do-while` executes at least once.
    
- `for` combines initialization, condition, and update.
    
- `switch` selects code based on a value.
    
- In assembly, all flow control is implemented using **comparisons (`cmp`, `test`) and jumps (`je`, `jne`, `jmp`, etc.)**.