Unicode becomes important when a program needs to work with characters beyond basic ASCII.

C has several character types and functions for handling **multibyte characters, wide characters, locales, and Unicode encodings**.

---

# What is Unicode?

**Unicode** is a universal character set that assigns a unique number to characters from different languages and symbol systems.

Examples:

```text
A    → U+0041
€    → U+20AC
•    → U+2022
```

Unicode defines **which character a number represents**, not how that character is stored as bytes.

---

# Code Points

A **code point** is the numeric value assigned to a Unicode character.

Example:

```text
A → U+0041
```

Here:

```text
U+0041
  │
  └── Unicode code point
```

A code point is **not necessarily one byte**.

---

# Encoding

An **encoding** specifies how a Unicode code point is represented in memory or a file.

Common Unicode encodings:

```text
UTF-8
UTF-16
UTF-32
```

Conceptually:

```text
Character
    │
    ▼
Code Point
    │
    ▼
Encoding
    │
    ▼
Bytes / Code Units
```

For example:

```text
€ → U+20AC → UTF-8 → E2 82 AC
```

---

# Source and Execution Character Sets

C has two important character-set concepts.

### Source Character Set

The characters that can appear in C source code.

### Execution Character Set

The character representation used by the compiled program for character constants and string literals.

Conceptually:

```text
C Source
   │
   ▼
Compiler
   │
   ▼
Execution Character Set
   │
   ▼
Program
```

---

# Unicode in C

C provides several character-related types:

```c
char
wchar_t
char16_t
char32_t
```

Important:

> None of these automatically means "one Unicode character".

Their meaning depends on the type, encoding, implementation, and locale.

---

# A Quick Note on UTF-8

UTF-8 is a **variable-length Unicode encoding**.

A code point uses:

```text
1–4 bytes
```

Examples:

```text
A    → 41
€    → E2 82 AC
```

ASCII characters use exactly **one byte** in UTF-8.

---

# Different Character Types

|Type|Purpose|
|---|---|
|`char`|Byte-oriented character data|
|`wchar_t`|Wide character|
|`char16_t`|UTF-16 code unit|
|`char32_t`|UTF-32 code unit|

`char` is commonly used for UTF-8 data, but C does **not require `char` strings to use UTF-8**.

---

# Multibyte Characters

A **multibyte character** uses multiple `char` bytes to represent one character.

For example, in UTF-8:

```text
€ → E2 82 AC
```

Therefore:

```text
1 character
    │
    ▼
3 bytes
```

This means:

```c
strlen("€");
```

may return:

```text
3
```

not `1`, because `strlen()` counts bytes.

---

# Wide Characters

C provides:

```c
wchar_t
```

for **wide characters**.

Wide-character literals use the `L` prefix:

```c
wchar_t c = L'A';

wchar_t *s = L"Hello";
```

Wide-character functionality is mainly provided by:

```c
#include <wchar.h>
```

---

# Using Wide Characters and `wchar_t`

Example:

```c
#include <wchar.h>

int main(void)
{
    wchar_t c = L'A';

    wprintf(L"%lc\n", c);
}
```

Wide strings are terminated by a wide null character.

```text
L"Hello"
     │
     ▼
L'H' L'e' L'l' L'l' L'o' L'\0'
```

---

# `wchar_t` Is Implementation-Defined

The size and representation of `wchar_t` depend on the implementation.

Commonly:

```text
Linux/macOS → 32-bit
Windows     → 16-bit
```

Therefore:

> Do not assume `wchar_t` means UTF-32.

---

# Multibyte to `wchar_t` Conversions

C provides functions for converting multibyte characters to wide characters.

Examples:

```c
mbtowc()
mbrtowc()
```

Conceptually:

```text
Multibyte Character
        │
        ▼
    Conversion
        │
        ▼
     wchar_t
```

The interpretation depends on the current locale.

---

# Wide Character Functionality

Wide-character versions of many C library functions exist.

Examples:

```c
wprintf()
wcslen()
wcscmp()
wcschr()
```

Include:

```c
#include <wchar.h>
```

---

# `wint_t`

`wint_t` is an integer type capable of representing:

- Wide-character values
    
- `WEOF`
    

Example:

```c
wint_t c;

c = getwchar();
```

`wint_t` is useful for wide-character I/O because `wchar_t` is not necessarily able to represent `WEOF`.

---

# I/O Stream Orientation

A C stream initially has no orientation.

Using byte I/O makes it:

```text
Byte-oriented
```

Using wide-character I/O makes it:

```text
Wide-oriented
```

Conceptually:

```text
             ┌──► Byte-oriented
Unoriented ──┤
             └──► Wide-oriented
```

Once oriented, you should use the corresponding type of I/O.

---

# I/O Functions

### Byte-oriented

```c
printf()
fprintf()
fgetc()
fputc()
```

### Wide-oriented

```c
wprintf()
fwprintf()
fgetwc()
fputwc()
```

Avoid mixing byte and wide I/O on the same stream after it has become oriented.

---

# Type Conversion Functions

Important conversion functions include:

```c
mbtowc()
mbrtowc()
wctomb()
wcrtomb()
```

They convert between:

```text
Multibyte representation
        ↕
Wide-character representation
```

---

# String and Memory Copying Functions

Wide-character copying functions include:

```c
wcscpy()
wcsncpy()
wmemcpy()
wmemmove()
```

They operate on wide-character data rather than ordinary `char` strings.

---

# String and Memory Comparing Functions

Examples:

```c
wcscmp()
wcsncmp()
wmemcmp()
```

Locale-aware wide-string comparison functions include:

```c
wcscoll()
wcsxfrm()
```

---

# String Searching Functions

Common wide-string searching functions include:

```c
wcschr()
wcsrchr()
wcsstr()
wcspbrk()
wcsspn()
wcscspn()
```

They provide functionality similar to many `string.h` functions.

---

# Length / Miscellaneous Functions

A common function is:

```c
wcslen()
```

Example:

```c
wcslen(L"Hello");
```

returns the number of `wchar_t` elements before the terminating null character.

Important:

> It does not necessarily represent the number of Unicode characters perceived by the user.

---

# Character Classification Functions

Include:

```c
#include <wctype.h>
```

Examples:

```c
iswalpha()
iswdigit()
iswalnum()
iswspace()
iswupper()
iswlower()
```

They classify wide characters according to the current locale.

---

# Character Conversion Functions

Examples:

```c
towlower()
towupper()
```

They convert wide characters according to locale rules.

Example:

```text
Wide Character
      │
      ▼
Locale Rules
      │
      ▼
Converted Character
```

---

# Parse State, Restartable Functions

Some multibyte characters require multiple bytes to be processed.

C therefore provides **restartable conversion functions** such as:

```c
mbrtowc()
wcrtomb()
```

These can maintain conversion state between calls.

---

# `mbstate_t`

`mbstate_t` stores the state of a multibyte character conversion.

Example concept:

```text
Byte 1
  │
  ▼
Conversion State
  │
  ▼
Byte 2
  │
  ▼
Complete Character
```

This is useful when a multibyte sequence is processed across multiple function calls.

---

# Unicode Encodings and C

Unicode and encoding are different concepts.

```text
Unicode
   │
   ▼
Code Point
   │
   ├──► UTF-8
   ├──► UTF-16
   └──► UTF-32
```

C provides different character types and literal prefixes for working with these representations.

---

# UTF-8

UTF-8 represents Unicode code points using **1–4 bytes**.

Examples:

```text
A    → 41
€    → E2 82 AC
😀   → F0 9F 98 80
```

A UTF-8 string is commonly stored as:

```c
char *s;
```

Important:

> UTF-8 is byte-oriented, so one Unicode character may occupy multiple bytes.

---

# UTF-16, UTF-32, `char16_t`, and `char32_t`

### UTF-16

Uses **16-bit code units**.

Most code points use one code unit, while code points outside the Basic Multilingual Plane use **two code units** called a surrogate pair.

C provides:

```c
char16_t
```

### UTF-32

Uses **32-bit code units**.

A Unicode code point fits into one UTF-32 code unit.

C provides:

```c
char32_t
```

---

# `char16_t` and `char32_t` Literals

Include:

```c
#include <uchar.h>
```

Examples:

```c
char16_t c = u'B';
char16_t *s = u"Hello";

char32_t d = U'B';
char32_t *t = U"Hello";
```

Prefixes:

```text
u'...' → char16_t
U'...' → char32_t

u"..." → char16_t string
U"..." → char32_t string
```

---

# `wchar_t` vs `char16_t` vs `char32_t`

These types are not interchangeable.

```text
wchar_t
   │
   └── Implementation-defined wide character

char16_t
   │
   └── 16-bit code unit

char32_t
   │
   └── 32-bit code unit
```

In particular:

> `wchar_t` is not guaranteed to be UTF-16 or UTF-32.

---

# Multibyte Conversions

Important conversion functions include:

```c
mbrtowc()
wcrtomb()
mbsrtowcs()
wcsrtombs()
```

They convert between multibyte and wide-character sequences.

Conceptually:

```text
Multibyte Data
      │
      ▼
Conversion
      │
      ▼
Wide Data
```

The active locale determines how multibyte sequences are interpreted.

---

# Third-Party Libraries

The C standard library provides basic character, locale, and multibyte functionality.

For advanced Unicode processing, third-party libraries are often used.

Examples:

- ICU
    
- libunistring
    
- utf8proc
    

They can provide features such as:

- Unicode normalization
    
- Case folding
    
- Unicode properties
    
- Grapheme-cluster handling
    
- Advanced text processing
    

---

# Character vs Code Point vs Byte

These are **not the same thing**.

```text
Character
    │
    ▼
Unicode Code Point
    │
    ▼
Code Units
    │
    ▼
Bytes
```

For example:

```text
€
│
└── U+20AC
      │
      ▼
   UTF-8
      │
      ▼
 E2 82 AC
```

Therefore:

```text
1 code point
3 UTF-8 bytes
```

---

# Reverse Engineering Perspective

When reversing a program, determine **what character representation is actually stored**.

```text
char *
 │
 └── Byte/multibyte data, possibly UTF-8

wchar_t *
 │
 └── Implementation-dependent wide data

char16_t *
 │
 └── UTF-16 code units

char32_t *
 │
 └── UTF-32 code units
```

This matters when analyzing:

- Strings in memory
    
- Windows APIs
    
- File formats
    
- Network protocols
    
- Malware
    
- Exploit payloads
    

---

# Memory View

For:

```c
char *s = "€";
```

with UTF-8 execution encoding, memory may contain:

```text
E2 82 AC 00
│  │  │  │
│  │  │  └── '\0'
│  │  └───── UTF-8 byte
│  └──────── UTF-8 byte
└─────────── UTF-8 byte
```

So:

```c
strlen(s);
```

returns:

```text
3
```

because it counts bytes.

---

# Mental Model

```text
Unicode
   │
   ▼
Code Point
   │
   ├───────────┬───────────┐
   ▼           ▼           ▼
 UTF-8       UTF-16      UTF-32
   │           │           │
1–4 bytes   1–2 units    1 unit
   │           │           │
   └───────────┴───────────┘
               │
               ▼
             Memory
```

In C:

```text
char
 │
 └── Byte / multibyte data

wchar_t
 │
 └── Implementation-defined wide character

char16_t
 │
 └── UTF-16 code unit

char32_t
 │
 └── UTF-32 code unit
```

---

# Long-Term Takeaways

- **Unicode** assigns unique code points to characters.
    
- A **code point** identifies a Unicode character.
    
- An **encoding** determines how code points are represented.
    
- UTF-8, UTF-16, and UTF-32 are Unicode encodings.
    
- UTF-8 uses **1–4 bytes** per code point.
    
- UTF-16 uses **1 or 2 16-bit code units** per code point.
    
- UTF-32 uses **one 32-bit code unit** per code point.
    
- `char` commonly stores UTF-8/multibyte data, but `char` does **not** imply UTF-8.
    
- `wchar_t` is implementation-defined.
    
- Never assume `wchar_t` means UTF-32.
    
- `char16_t` represents UTF-16 code units.
    
- `char32_t` represents UTF-32 code units.
    
- `u"..."` creates `char16_t` string literals.
    
- `U"..."` creates `char32_t` string literals.
    
- Multibyte characters can occupy multiple `char` bytes.
    
- `strlen()` counts **bytes**, not Unicode characters.
    
- Wide-character functionality is mainly provided by `<wchar.h>`.
    
- Wide-character classification uses `<wctype.h>`.
    
- `wint_t` can represent wide characters and `WEOF`.
    
- C streams become **byte-oriented or wide-oriented**.
    
- Do not arbitrarily mix byte and wide I/O on an oriented stream.
    
- `mbrtowc()` and `wcrtomb()` perform restartable conversions.
    
- `mbstate_t` stores multibyte conversion state.
    
- Locale affects multibyte and character-processing behavior.
    
- **Byte, code unit, code point, and user-perceived character are different concepts.**
    
- For advanced Unicode processing, libraries such as ICU are often preferable to the basic C facilities.