
C has more character and string features than just:

```c
char c = 'A';
char *s = "Hello";
```

C also supports:

- Escape sequences
    
- Numeric escapes
    
- Unicode escapes
    
- Multibyte characters
    
- Wide characters
    
- Different character encodings
    

These become important when dealing with:

- Unicode
    
- UTF-8
    
- Localization
    
- Terminal output
    
- Binary data
    

---

# Escape Sequences

An **escape sequence** represents a special character inside a character constant or string literal.

Escape sequences begin with:

```c
\
```

Example:

```c
printf("Hello\nWorld\n");
```

Output:

```text
Hello
World
```

The characters:

```c
\n
```

represent one special character: a newline.

---

# Frequently-Used Escapes

|Escape|Description|
|---|---|
|`\n`|Newline|
|`\'`|Single quote|
|`\"`|Double quote|
|`\\`|Backslash|

Examples:

```c
printf("Use \\n for newline\n");
```

Output:

```text
Use \n for newline
```

---

# Single Quote Escape

A single quote can be escaped inside a character constant:

```c
char c = '\'';
```

Printing:

```c
printf("%c\n", c);
```

Output:

```text
'
```

---

# Double Quote Escape

A double quote inside a string must be escaped:

```c
printf("Say \"hello\"!\n");
```

Output:

```text
Say "hello"!
```

---

# Backslash Escape

To print a literal backslash:

```c
printf("C:\\Users\\Name\n");
```

Output:

```text
C:\Users\Name
```

The first backslash tells C:

> The next character is part of an escape sequence.

---

# Rarely-Used Escapes

|Escape|Description|
|---|---|
|`\a`|Alert|
|`\b`|Backspace|
|`\f`|Form feed|
|`\r`|Carriage return|
|`\t`|Horizontal tab|
|`\v`|Vertical tab|
|`\?`|Literal question mark|

---

# Alert: `\a`

```c
printf("Warning!\a\n");
```

`\a` requests an alert from the terminal.

Depending on the terminal, this might:

- Make a sound
    
- Flash
    
- Do nothing
    

Its behavior depends on the environment.

---

# Backspace: `\b`

```c
printf("ABC\bD\n");
```

Conceptually:

```text
ABC
  ↑
move cursor back
```

`\b` moves the cursor back one position.

Important:

> It does not necessarily delete a character.

Terminal behavior can vary.

---

# Form Feed: `\f`

```c
printf("Page 1\fPage 2\n");
```

Historically, `\f` moved to the next page.

It comes from printers and older terminals.

Modern terminals may handle it differently or similarly to another whitespace control character.

---

# Carriage Return: `\r`

`\r` moves the cursor back to the **beginning of the current line**.

Example:

```c
printf("Hello\rBye");
```

Conceptually:

```text
Hello
↑
cursor returns here
```

Then `Bye` overwrites the beginning of the line.

Possible display:

```text
Byelo
```

This is useful for updating output without creating new lines.

---

# Horizontal Tab: `\t`

```c
printf("Name\tAge\n");
```

Output might look like:

```text
Name    Age
```

`\t` moves the cursor to the next horizontal tab stop.

The exact spacing depends on the environment.

---

# Vertical Tab: `\v`

`\v` moves vertically to the next vertical tab position.

Example:

```c
printf("Hello\vWorld\n");
```

Its visible behavior can vary significantly between terminals.

---

# Question Mark Escape: `\?`

```c
printf("What\?\n");
```

This produces:

```text
What?
```

Usually, writing a normal `?` is fine.

The escape historically existed to avoid accidentally creating **trigraphs**.

---

# Single-Line Status Updates

`\r` and sometimes `\b` can be used to update the same terminal line repeatedly.

Example:

```c
#include <stdio.h>
#include <windows.h>

int main(void)

{

    for (int i = 10; i >= 0; i--) {
        printf("\rT minus %d second%s...   ", i, i != 1 ? "s" : "");
        fflush(stdout);
  
        Sleep(1000);  // Sleep for 1000 milliseconds = 1 second

    }
  
    printf("\rLiftoff!          \n");

    return 0;

}
```

---

# How It Works

```text
T minus 10 seconds...
        │
        ▼
\r moves cursor to line start
        │
        ▼
T minus 9 seconds...
```

The output is repeatedly overwritten instead of scrolling.

`fflush(stdout)` forces the buffered output to appear immediately.

---

# Why the Extra Spaces?

Consider:

```c
printf("\rLiftoff!        \n");
```

The spaces can overwrite leftover characters from a longer previous message.

For example:

```text
Previous:
T minus 10 seconds...

Without clearing:
Liftoff!0 seconds...
```

Extra spaces help prevent this.

---

# Trigraphs

Historically, C supported **trigraphs**.

A trigraph was a sequence beginning with:

```text
??
```

that represented another character.

For example, historically:

```text
??!
```

could represent:

```text
|
```

This existed for old character sets that lacked certain symbols.

---

# The Question Mark Problem

Consider:

```c
printf("Doesn't it??!\n");
```

On implementations where trigraph processing applies, the sequence:

```text
??!
```

can be interpreted as:

```text
|
```

Conceptually:

```text
Doesn't it??!
           │
           ▼
           |
```

Result:

```text
Doesn't it|
```

---

# Avoiding the Trigraph Sequence

Historically, you could prevent accidental interpretation with:

```c
printf("Doesn't it?\?!\n");
```

The escaped question mark breaks the sequence.

Important:

> Trigraph handling is mainly historical and is not something you will commonly encounter in modern C code.

But you may encounter it when reading old source code.

---

# Numeric Escapes

C also allows characters to be represented using numeric values.

Important forms include:

```text
\123
\x4D
\u2620
\U0001243F
```

These represent characters or encoded values numerically.

---

# Octal Escapes

An octal escape uses:

```c
\123
```

The digits are interpreted as an octal value.

Example:

```c
printf("A\102C\n");
```

`102` in octal represents:

```text
66 decimal
```

which is:

```text
B
```

Output:

```text
ABC
```

---

# Important: Octal Escape Length

Octal escapes use:

```text
Up to 3 octal digits
```

Example:

```c
\101
```

represents:

```text
'A'
```

Be careful because following digits can become part of the escape.

---

# Hexadecimal Escapes

A hexadecimal escape begins with:

```c
\x
```

Example:

```c
\x41
```

represents:

```text
0x41
```

which is commonly:

```text
A
```

Example:

```c
printf("\x41\n");
```

Output:

```text
A
```

---

# Important: Hex Escape Length

Unlike octal escapes, `\x` is followed by **one or more hexadecimal digits**.

Example:

```c
"\x4142"
```

does not automatically mean:

```text
A B
```

The compiler can treat all following hexadecimal digits as part of the same escape.

When necessary, separate the string:

```c
"\x41" "42"
```

---

# Unicode Escape: `\u`

`\u` represents a Unicode code point using exactly:

```text
4 hexadecimal digits
```

Example:

```c
\u2022
```

The Unicode code point:

```text
U+2022
```

is:

```text
•
```

Usage:

```c
printf("\u2022 Bullet\n");
```

Output on a suitable terminal:

```text
• Bullet
```

---

# Unicode Escape: `\U`

`\U` represents a Unicode code point using exactly:

```text
8 hexadecimal digits
```

Example:

```c
\U00002022
```

This is also:

```text
U+2022
```

Usage:

```c
printf("\U00002022 Bullet\n");
```

Output:

```text
• Bullet
```

---

# `\u` vs `\U`

|Escape|Hex Digits|
|---|--:|
|`\u`|Exactly 4|
|`\U`|Exactly 8|

Example:

```c
\u2022
```

Equivalent code point:

```c
\U00002022
```

Leading zeros are required.

---

# Embedding UTF-8 Bytes Directly

You can manually embed byte values.

Example:

```c
printf("\xE2\x80\xA2 Bullet\n");
```

The bytes:

```text
E2 80 A2
```

represent the UTF-8 encoding of:

```text
•
```

On a UTF-8 terminal, the output is:

```text
• Bullet
```

---

# Why Manual UTF-8 Bytes Are Usually a Bad Idea

This:

```c
"\xE2\x80\xA2"
```

requires knowing the exact UTF-8 byte sequence.

Instead:

```c
"\u2022"
```

expresses your intent more clearly:

```text
Unicode code point
        │
        ▼
U+2022
        │
        ▼
Character encoding handles representation
```

Unicode escapes are generally easier to read and maintain.

---

# Character Encoding

A character is not necessarily stored as one byte.

For example:

```text
ASCII
```

uses simple single-byte character values.

But UTF-8 can use:

```text
1 to 4 bytes
```

for a Unicode code point.

Conceptually:

```text
Character
   │
   ▼
Unicode Code Point
   │
   ▼
Encoding
   │
   ├── UTF-8
   ├── UTF-16
   └── UTF-32
```

---

# Multibyte Characters

A **multibyte character** can use multiple bytes to represent one character.

Example in UTF-8:

```text
A
```

usually uses:

```text
1 byte
```

But:

```text
•
```

uses:

```text
3 bytes in UTF-8
```

So:

```c
char s[] = "•";
```

contains multiple bytes, even though it displays as one character.

---

# `char` and Unicode

A `char` represents one byte.

Example:

```c
char c = 'A';
```

For ordinary ASCII characters, this is simple.

But one Unicode character may require multiple `char` values when encoded as UTF-8.

Example conceptually:

```text
"•"

char[0] = 0xE2
char[1] = 0x80
char[2] = 0xA2
```

plus the null terminator.

---

# Strings and Bytes

Remember:

```c
char s[] = "Hello";
```

is fundamentally an array of bytes:

```text
H  e  l  l  o  \0
```

With UTF-8:

```text
character count
        ≠
byte count
```

This is extremely important when working with:

- `strlen()`
    
- Buffer sizes
    
- Binary parsing
    
- Unicode text
    
- Reverse engineering
    

---

# Reverse Engineering Perspective

Suppose you find these bytes in a binary:

```text
E2 80 A2
```

They might represent:

```text
•
```

when interpreted as UTF-8.

But the CPU only sees:

```text
0xE2 0x80 0xA2
```

The meaning depends on:

```text
Raw Bytes
    │
    ▼
How the program interprets them
    │
    ├── Text encoding
    ├── Integer
    ├── Machine data
    └── Other binary data
```

---

# Escape Sequences in Binaries

Source:

```c
printf("Hello\n");
```

The string stored in the binary typically contains an actual newline byte:

```text
48 65 6C 6C 6F 0A 00
 H  e  l  l  o \n \0
```

The binary does not normally store:

```text
\ + n
```

Instead, the compiler converts the escape sequence into the appropriate character value.

---

# Mental Model

```text
C Source

"Hello\n"
    │
    ▼
Escape Sequence Processing
    │
    ▼
Characters / Encoded Bytes

H e l l o 0A
          │
          ▼
       Newline
```

For Unicode:

```text
"\u2022"
    │
    ▼
Unicode Code Point

U+2022
    │
    ▼
Execution Character Encoding
    │
    ▼
Encoded Bytes
```

---

# Long-Term Takeaways

- Escape sequences represent special characters inside strings and character constants.
    
- Common escapes include `\n`, `\'`, `\"`, and `\\`.
    
- `\r` moves the cursor to the beginning of the current line.
    
- `\b` moves the cursor backward, but does not inherently delete a character.
    
- `\t` creates a horizontal tab.
    
- `\a`, `\f`, and `\v` are less commonly used and may depend on the environment.
    
- `\r` is useful for single-line terminal status updates.
    
- `fflush(stdout)` can force buffered output to appear immediately.
    
- `\?` historically helped avoid accidental trigraph sequences.
    
- Trigraphs are mainly a historical C feature you may encounter in old code.
    
- Octal escapes use up to 3 octal digits.
    
- Hexadecimal escapes use `\x` followed by hexadecimal digits.
    
- `\u` uses exactly 4 hexadecimal digits for a Unicode code point.
    
- `\U` uses exactly 8 hexadecimal digits for a Unicode code point.
    
- Unicode code points and their encoded byte representation are different things.
    
- UTF-8 uses a variable number of bytes per character.
    
- A `char` represents one byte, not necessarily one complete Unicode character.
    
- `strlen()` counts bytes before `\0`, not Unicode characters.
    
- In a binary, escape sequences are usually stored as their resulting byte values.
    
- In reverse engineering, always distinguish between raw bytes and the encoding or type used to interpret them.