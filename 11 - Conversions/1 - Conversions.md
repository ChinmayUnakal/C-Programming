
# String Conversions

> **Core idea:** C provides several ways to convert between numeric values and strings. For reverse engineering, understanding these conversions is useful because they often appear around **input parsing, command-line arguments, configuration files, network protocols, and malware configuration parsing**.

---

# Numeric Value → String

To convert a numeric value into a string, C provides functions such as:

```c
sprintf()
snprintf()
```

The safer general choice is usually `snprintf()` because it allows you to specify the size of the destination buffer.

Example:

```c
#include <stdio.h>

int main(void)
{
    char s[10];
    float f = 3.14159;

    snprintf(s, 10, "%f", f);

    printf("String value: %s\n", s);
}
```

Output:

```text
String value: 3.141590
```

The arguments are:

```c
snprintf(destination, size, format, value);
```

So:

```c
snprintf(s, 10, "%f", f);
```

means:

> Convert `f` according to `%f`, and write the resulting string into `s`, using at most 10 bytes including the terminating `'\0'`.

### Why `snprintf()` matters

Consider:

```c
char s[5];

sprintf(s, "%f", 3.14);
```

The resulting string is much larger than the buffer can hold.

This can cause a **buffer overflow**.

With:

```c
snprintf(s, sizeof(s), "%f", 3.14);
```

the function knows the maximum amount of space available.

> **RE relevance:** When reversing a binary, calls to `sprintf`, `snprintf`, `printf`, `strcpy`, `strcat`, etc. can reveal **string construction and potentially dangerous buffer operations**.

---

# `sprintf()` vs `snprintf()`

### `sprintf()`

```c
sprintf(s, "%d", 1234);
```

Writes the formatted result into `s`.

It does **not** know the size of `s`.

### `snprintf()`

```c
snprintf(s, sizeof(s), "%d", 1234);
```

Takes the buffer size into account.

> **Security relevance:** Unbounded string formatting functions are historically associated with buffer-overflow vulnerabilities. During binary analysis, recognizing these functions is useful.

---

# String → Numeric Value

C provides several simple conversion functions in `<stdlib.h>`:

|Function|Converts to|
|---|---|
|`atoi()`|`int`|
|`atof()`|`double`|
|`atol()`|`long`|
|`atoll()`|`long long`|

Example:

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    char *pi = "3.14159";

    double d = atof(pi);

    printf("%f\n", d);
}
```

Output:

```text
3.141590
```

---

# Why `atoi()` Is Usually Not Preferred

You might see:

```c
int x = atoi("1234");
```

This produces:

```text
1234
```

But consider:

```c
int x = atoi("what");
```

The problem is that `atoi()` gives you **very limited error information**.

You cannot reliably distinguish:

```c
"0"
```

from an invalid conversion such as:

```c
"what"
```

using the return value alone.

For robust input parsing, prefer:

```c
strtol()
strtoul()
strtoll()
strtoull()
```

because these functions provide much better error handling.

> **RE relevance:** When analyzing malware, exploit PoCs, command-line parsers, or network-facing programs, recognizing `strtol`/`strtoul` can immediately tell you that a string is being interpreted as a number.

---

# `strto*()` Functions

The more powerful conversion functions include:

```c
strtol()
strtoll()
strtoul()
strtoull()

strtof()
strtod()
strtold()
```

They provide more control than `atoi()` and friends.

For example:

```c
char *s = "3490";

unsigned long x = strtoul(s, NULL, 10);
```

The `10` means:

> Interpret the string as a **base-10** number.

Output:

```text
3490
```

---

# The `base` Argument

The third argument specifies the numeric base.

```c
strtoul(s, NULL, base);
```

For example:

```c
strtoul("3490", NULL, 10);
```

means decimal.

```c
strtoul("101010", NULL, 2);
```

means binary.

```c
strtoul("2A", NULL, 16);
```

means hexadecimal.

Example:

```c
char *s = "101010";

unsigned long x = strtoul(s, NULL, 2);

printf("%lu\n", x);
```

Output:

```text
42
```

Because:

```text
101010₂ = 42₁₀
```

---

# Why Base Conversion Matters in RE

A program may receive:

```text
"0x401000"
```

or:

```text
"101010"
```

as text and convert it into a numeric value.

When reversing the program, a call such as:

```c
strtoul(input, NULL, 16);
```

suggests:

> The program expects the user/input to provide a hexadecimal number.

This is common in:

- Debuggers
    
- Exploit tools
    
- CTF programs
    
- Configuration parsers
    
- Malware
    
- Network tools
    
- Address/offset parsers
    

---

# Detecting Invalid Characters with `strtoul()`

One major advantage of `strtoul()` is that it can tell us **where conversion stopped**.

Consider:

```c
char *s = "34x90";
char *badchar;

unsigned long x = strtoul(s, &badchar, 10);
```

The string is:

```text
34x90
```

The valid decimal portion is:

```text
34
```

Conversion therefore produces:

```text
x = 34
```

But:

```text
x
```

is not a valid decimal digit.

`badchar` points to the first character that could not be converted.

Therefore:

```c
printf("%c\n", *badchar);
```

prints:

```text
x
```

### Important mental model

```text
"34x90"
  │
  ├── "34" → successfully converted
  │
  └── "x90" → conversion stops here
       ↑
    badchar
```

---

# Detecting Successful Conversion

If conversion succeeds completely:

```c
char *s = "3490";
char *badchar;

unsigned long x = strtoul(s, &badchar, 10);
```

After conversion:

```text
badchar → '\0'
```

because the conversion consumed the entire string.

So:

```c
if (*badchar == '\0') {
    printf("Success! %lu\n", x);
} else {
    printf("Partial conversion: %lu\n", x);
}
```

This gives us a simple way to determine whether the **entire string** represented a valid number.

---

# `strtol()` and `errno`

For serious error checking, `strtol()` can also be combined with `errno`.

```c
#include <errno.h>
#include <limits.h>
#include <stdlib.h>

char *end;
errno = 0;

long x = strtol("999999999999999999999", &end, 10);

if (errno == ERANGE) {
    // Value was outside the representable range
}
```

This is important because there are **two different problems** you may want to detect:

```text
Did parsing stop early?
        ↓
       endptr

Did the number exceed the representable range?
        ↓
       errno
```

> **RE relevance:** This gives you clues about how carefully a program validates attacker-controlled numeric input.

---

# Character Conversions

A character such as:

```c
'5'
```

is **not the same value** as:

```c
5
```

Example:

```c
printf("%d %d\n", 5, '5');
```

On an ASCII-based system:

```text
5 53
```

Why?

Because:

```text
'5' → character code 53
5   → integer value 5
```

---

# Character Codes

The C standard guarantees that the decimal digit characters occur in consecutive order:

```text
'0'
'1'
'2'
'3'
'4'
'5'
'6'
'7'
'8'
'9'
```

On ASCII systems:

```text
'0' = 48
'1' = 49
'2' = 50
'3' = 51
'4' = 52
'5' = 53
'6' = 54
'7' = 55
'8' = 56
'9' = 57
```

Therefore:

```c
char c = '6';

int x = c - '0';
```

produces:

```text
x = 6
```

---

# Character Digit → Integer

This is one of the most important small C patterns to recognize:

```c
char c = '7';

int x = c - '0';
```

Result:

```text
x = 7
```

Why?

```text
'7' = 55
'0' = 48

55 - 48 = 7
```

This works for any decimal digit:

```c
int digit = c - '0';
```

provided `c` is actually between:

```c
'0' and '9'
```

A common validation pattern is:

```c
if (c >= '0' && c <= '9') {
    int digit = c - '0';
}
```

> **RE relevance:** If you see assembly performing a subtraction involving `0x30` (`'0'`), the code may be converting an ASCII digit into its numeric value.

For example:

```asm
sub eax, 0x30
```

may correspond to:

```c
digit = c - '0';
```

---

# Integer → Character Digit

The reverse conversion also works:

```c
int x = 6;

char c = x + '0';
```

Now:

```text
c = '6'
```

Example:

```c
printf("%c\n", c);
```

Output:

```text
6
```

Again, this only makes sense when:

```text
0 <= x <= 9
```

---

# Character Conversion in Reverse Engineering

A classic pattern:

```c
if (c >= '0' && c <= '9')
    x = c - '0';
```

might compile into logic resembling:

```asm
cmp  al, '0'
jb   invalid

cmp  al, '9'
ja   invalid

sub  al, '0'
```

Recognizing this pattern lets you reconstruct:

```c
digit = c - '0';
```

instead of treating the assembly as arbitrary arithmetic.

---

# Numeric Conversions

C also performs conversions directly between numeric types.

For example:

```c
int x = 10;
double y = x;
```

The integer value:

```text
10
```

is converted to a floating-point value:

```text
10.0
```

---

# Boolean Conversion

When a value is converted to `_Bool`:

```text
0       → 0
non-zero → 1
```

Example:

```c
#include <stdio.h>
#include <stdbool.h>

int main(void)
{
    bool a = 0;
    bool b = 42;

    printf("%d\n", a);
    printf("%d\n", b);
}
```

Output:

```text
0
1
```

The important rule is:

```text
zero     → false
non-zero → true
```

> **RE relevance:** This often appears in conditional logic. A compiler may reduce something like `if (x)` to a test for whether a register is zero.

For example:

```asm
test eax, eax
je   zero
```

can correspond to:

```c
if (x == 0)
```

or:

```c
if (!x)
```

---

# Integer → Integer Conversion

When converting between integer types, the result depends on:

- Width
    
- Signedness
    
- Representable range
    

For unsigned integers, values are reduced modulo:

```text
2^N
```

where `N` is the number of bits in the destination type.

Example:

```c
unsigned char x = 300;
```

An 8-bit unsigned integer can represent:

```text
0–255
```

So the stored value wraps modulo 256:

```text
300 mod 256 = 44
```

Therefore:

```text
x = 44
```

> **RE relevance:** This kind of truncation is extremely important when reversing code involving bytes, packet fields, checksums, cryptography, and bit manipulation.

---

# Integer → Smaller Integer

Consider:

```c
int x = 0x12345678;

unsigned char c = x;
```

Only the low 8 bits fit into `unsigned char`.

```text
0x12345678
       ↓
      0x78
```

Therefore:

```text
c = 0x78
```

This is especially important in assembly because you may see:

```asm
mov eax, 0x12345678
mov BYTE PTR [...], al
```

The compiler is effectively storing only the **lowest byte**.

---

# Signed Integer Conversion

Converting an integer to a signed type when the value cannot be represented has an **implementation-defined result** or an implementation-defined signal, depending on the conversion context.

The key practical point is:

> Do not assume that an out-of-range signed conversion has one universal result across every C implementation.

For reverse engineering, determine the target platform/compiler/ABI rather than relying solely on abstract C assumptions.

---

# Integer → Floating Point

Example:

```c
int x = 42;

float f = x;
```

The value becomes approximately:

```text
42.0
```

Similarly:

```c
double d = 42;
```

produces:

```text
42.0
```

The destination floating-point type may not be able to represent every integer exactly.

For example, sufficiently large integers cannot all be represented exactly by a `float`.

> **RE relevance:** Integer-to-floating-point conversion may result in instructions that move data between **general-purpose registers and XMM registers**.

---

# Floating Point → Integer

Consider:

```c
double x = 3.99;

int y = x;
```

The fractional part is discarded:

```text
3.99 → 3
```

Likewise:

```text
-3.99 → -3
```

This is **truncation toward zero**, not ordinary rounding.

```c
int x = (int)3.99;
```

produces:

```text
3
```

---

# Floating Point Too Large for Integer

This is important:

```c
double x = 999999999999999999999.0;

int y = (int)x;
```

If the floating-point value cannot be represented in the destination integer type, the conversion has **undefined behavior**.

So don't assume it will simply "wrap around."

---

# Floating Point → Floating Point

Example:

```c
double x = 3.1415926535;

float y = x;
```

The `float` may not have enough precision to represent the exact `double`.

The result is therefore the closest representable value according to the implementation's floating-point behavior.

> **RE relevance:** This distinction matters when reversing scientific software, graphics programs, games, DSP code, and cryptographic/math-heavy programs.

---

# Implicit Conversions

An **implicit conversion** happens automatically when C needs to convert one type into another.

Example:

```c
int x = 10;
double y = x;
```

The compiler automatically converts:

```text
int → double
```

No cast is required.

Another example:

```c
double x = 3.14;
int y = x;
```

The compiler automatically performs:

```text
double → int
```

---

# Integer Promotions

Small integer types are frequently promoted before arithmetic.

Consider:

```c
char x = 10;
char y = 20;

int i = x + y;
```

Before the addition, `x` and `y` are generally promoted to `int`.

Conceptually:

```text
char x
   ↓
 int

char y
   ↓
 int

 int + int
   ↓
 int
```

This is why the result of:

```c
x + y
```

is normally an `int`, even though `x` and `y` are `char`.

---

# Integer Promotion in x86-64

Suppose:

```c
unsigned char x = 10;

int y = x;
```

The compiler may use:

```asm
movzx eax, BYTE PTR [...]
```

Meaning:

```text
8-bit value
    ↓
zero-extend
    ↓
32-bit value
```

For a signed `char`:

```asm
movsx eax, BYTE PTR [...]
```

may be used instead.

### The key distinction

```text
movzx → zero extension
        unsigned interpretation

movsx → sign extension
        signed interpretation
```

This is one of the most useful clues for reconstructing C types.

---

# The Usual Arithmetic Conversions

When different arithmetic types are used together, C performs conversions so that the operands can participate in a common arithmetic operation.

Example:

```c
int x = 3;
double y = 1.2;

double z = x + y;
```

Conceptually:

```text
int 3
 ↓
double 3.0

3.0 + 1.2
 ↓
4.2
```

The result is:

```text
4.2
```

---

# Assignment Conversion

Consider:

```c
int x = 3.9;
```

The expression:

```c
3.9
```

is a `double`.

Before storing it in `x`, it is converted:

```text
double 3.9
     ↓
int 3
```

So:

```c
printf("%d\n", x);
```

prints:

```text
3
```

---

# `void *` Conversions

`void *` is the **generic object pointer type**.

A pointer to an object can be converted to `void *` and back.

Example:

```c
int x = 10;

void *p = &x;

int *q = p;
```

Conceptually:

```text
int *
 ↓
void *
 ↓
int *
```

The pointer still points to the same object.

### Important

`void *` does **not** tell the compiler what the pointed-to object is.

For example:

```c
void *p;
```

does not tell you whether `p` points to:

```text
int
char
struct
double
...
```

You need to interpret it correctly before dereferencing it.

---

# `void *` in Reverse Engineering

`void *` appears heavily in generic C APIs.

Examples include:

```c
malloc()
qsort()
memcpy()
memmove()
```

For example:

```c
void *p = malloc(100);
```

The compiler/library doesn't inherently know what those 100 bytes represent.

The programmer determines the interpretation.

> **RE relevance:** When reversing code involving `void *`, you often have to reconstruct the intended type from how the memory is accessed.

---

# Explicit Conversions

An explicit conversion is one where you tell the compiler which type you want.

The most common mechanism is a **cast**.

```c
int x = 10;

long y = (long)x;
```

The syntax is:

```c
(type) expression
```

Example:

```c
double x = 3.14;

int y = (int)x;
```

Now:

```text
y = 3
```

---

# Casting Pointers

You can explicitly convert pointer types:

```c
long x = 3490;
long *p = &x;

unsigned char *c = (unsigned char *)p;
```

Now `c` points at the same memory as `p`, but the compiler interprets accesses through `c` as `unsigned char`.

For example:

```c
printf("%u\n", *c);
```

reads one byte.

> **RE relevance:** This is extremely important for examining raw memory.

A pointer cast can effectively say:

> "Interpret these bytes as a different type."

---

# Pointer Casting and Memory

Suppose:

```c
unsigned int x = 0x12345678;

unsigned char *p = (unsigned char *)&x;
```

Now:

```c
p[0]
p[1]
p[2]
p[3]
```

access the individual bytes of `x`.

On a little-endian x86-64 machine, these bytes would appear as:

```text
78 56 34 12
```

This is a fundamental reverse-engineering concept.

> **RE relevance:** Pointer casts are commonly used to inspect **raw bytes, structures, binary formats, packet data, and memory layouts**.

---

# Casting `void *` in `qsort()`

`qsort()` uses generic pointers because it needs to work with arbitrary object types.

A comparison function might look like:

```c
int compar(const void *elem1, const void *elem2)
{
    const int *e1 = elem1;
    const int *e2 = elem2;

    return *e2 - *e1;
}
```

The important part is:

```c
const int *e1 = elem1;
```

The `void *` is being interpreted as:

```c
const int *
```

Now:

```c
*e1
```

means:

> Read an `int` from that memory.

---

# Cast Directly vs Assignment

You could instead write:

```c
int compar(const void *elem1, const void *elem2)
{
    if (*((const int *)elem2) > *((const int *)elem1))
        return 1;

    if (*((const int *)elem2) < *((const int *)elem1))
        return -1;

    return 0;
}
```

Here:

```c
(const int *)elem2
```

explicitly casts the pointer.

But this is often easier to read:

```c
const int *e2 = elem2;
```

The assignment performs the permitted conversion from `void *`.

> **General rule:** Don't add casts merely because you can. In normal C code, unnecessary casts often make code harder to read.

---

# `%p` and Pointer Printing

The `%p` format specifier is used to print a pointer.

Example:

```c
int x = 3490;
int *p = &x;

printf("%p\n", (void *)p);
```

The conventional form is:

```c
printf("%p\n", (void *)p);
```

because `%p` expects a `void *` argument.

The output might look like:

```text
0x7ffde1234560
```

The exact address changes between executions because of things such as **ASLR**.

> **RE relevance:** Pointer values are normally displayed in hexadecimal because they represent memory addresses.

---

# Pointer Casting in Reverse Engineering

Consider:

```c
long x = 3490;
unsigned char *p = (unsigned char *)&x;
```

The underlying memory hasn't changed.

Only the **type used to access it** has changed.

Think of it as:

```text
Same bytes
   │
   ├── long *          → interpret as long
   │
   └── unsigned char * → interpret one byte at a time
```

This mental model is extremely useful when reversing.

---

# Character Conversion Functions and `unsigned char`

Functions from:

```c
#include <ctype.h>
```

such as:

```c
isdigit()
isalpha()
isspace()
tolower()
toupper()
```

have an important requirement:

> Their argument must either be `EOF` or representable as `unsigned char`.

A common safe pattern is:

```c
int c = getchar();

if (isdigit((unsigned char)c)) {
    ...
}
```

This avoids problems when `char` is signed and contains a negative value.

> **RE relevance:** This is less about identifying a particular machine instruction and more about understanding why casts appear in real-world C source code.

---

# Conversion Cheat Sheet

|Conversion|Example|Important behavior|
|---|---|---|
|Number → string|`snprintf()`|Formats numeric value as text|
|String → `int`|`atoi()`|Simple but poor error handling|
|String → `long`|`strtol()`|Better validation|
|String → unsigned|`strtoul()`|Supports arbitrary base|
|String → `double`|`strtod()`|Parses floating-point text|
|Digit char → number|`c - '0'`|`'7' → 7`|
|Number → digit char|`x + '0'`|`7 → '7'`|
|Integer → unsigned|`unsigned x`|Modular reduction if needed|
|Floating → integer|`(int)x`|Fraction discarded|
|Integer → floating|`(double)x`|Converted to floating representation|
|`void *` → object pointer|`int *p = vp`|Generic pointer conversion|
|Pointer → `void *`|`void *p = &x`|Generic object pointer|
|Explicit conversion|`(int)x`|Cast|

---

# RE Conversion Patterns to Memorize

These are worth recognizing immediately when looking at assembly.

### ASCII digit → integer

```c
digit = c - '0';
```

Likely involves:

```asm
sub ..., 0x30
```

because:

```text
'0' = 0x30
```

---

### Unsigned byte → larger integer

```c
int x = (unsigned char)c;
```

Likely:

```asm
movzx eax, BYTE PTR [...]
```

---

### Signed byte → larger integer

```c
int x = (signed char)c;
```

Likely:

```asm
movsx eax, BYTE PTR [...]
```

---

### Integer → smaller integer

```c
unsigned char c = x;
```

Likely involves only the low byte:

```asm
mov BYTE PTR [...], al
```

---

### Signed comparison

C:

```c
if (x > y)
```

when `x` and `y` are signed integers.

Possible branch:

```asm
jg
```

---

### Unsigned comparison

C:

```c
if (x > y)
```

when `x` and `y` are unsigned integers.

Possible branch:

```asm
ja
```

---

### Integer → floating point

Possible instructions involving:

```text
XMM registers
```

and integer-to-floating-point conversion instructions.

---

# The Big Reverse-Engineering Picture

When C source is compiled:

```text
C type
   ↓
Compiler applies conversions
   ↓
Machine instructions
   ↓
Most type information disappears
```

When reversing:

```text
Machine instructions
       ↓
Observe:
 ├── operand size
 ├── sign extension
 ├── zero extension
 ├── comparison instructions
 ├── integer vs XMM registers
 ├── memory access width
 ├── library calls
 └── pointer usage
       ↓
Infer likely C types
```

For example:

```asm
movzx eax, BYTE PTR [rbp-0x5]
cmp   eax, 0x39
ja    invalid
sub   eax, 0x30
```

A strong reconstruction is:

```c
if (c >= '0' && c <= '9')
    digit = c - '0';
```

because:

```text
0x30 = '0'
0x39 = '9'
```


> **Key RE principle:** **Conversions are not just C syntax. They leave fingerprints in the generated assembly. Learn to recognize those fingerprints.**