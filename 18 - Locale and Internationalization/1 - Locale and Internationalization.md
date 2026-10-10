A **locale** tells a C program how to handle language- and region-specific behavior.

This includes:

- Character handling
    
- Number formatting
    
- Currency formatting
    
- Date and time formatting
    
- String collation
    

C locale support is provided through:

```c
#include <locale.h>
```

---

# Setting the Locale

Use:

```c
setlocale()
```

Syntax:

```c
setlocale(category, locale);
```

Example:

```c
setlocale(LC_ALL, "");
```

This tells C to use the locale from the environment for all locale categories.

---

# Common `setlocale()` Calls

Use the environment's locale:

```c
setlocale(LC_ALL, "");
```

Use the default C locale:

```c
setlocale(LC_ALL, "C");
```

A specific locale might look like:

```c
setlocale(LC_ALL, "en_US.UTF-8");
```

### Important

Specific locale names are **not fully portable**.

A locale available on one system might not exist or use the same name on another.

---

# Locale Categories

C divides localization into categories.

|Macro|Controls|
|---|---|
|`LC_ALL`|All locale categories|
|`LC_COLLATE`|String comparison and collation|
|`LC_CTYPE`|Character classification and conversion|
|`LC_MONETARY`|Monetary formatting|
|`LC_NUMERIC`|Numeric formatting and decimal point|
|`LC_TIME`|Date and time formatting|

---

# `LC_ALL`

```c
setlocale(LC_ALL, "");
```

Changes all locale categories at once.

Conceptually:

```text
LC_ALL
  │
  ├── LC_COLLATE
  ├── LC_CTYPE
  ├── LC_MONETARY
  ├── LC_NUMERIC
  └── LC_TIME
```

---

# `LC_COLLATE`

Controls locale-aware string ordering.

It affects functions such as:

```c
strcoll()
strxfrm()
```

Different languages can have different alphabetical sorting rules.

---

# `LC_CTYPE`

Controls character classification and conversion.

It affects functions involving:

```c
isalpha()
isdigit()
tolower()
toupper()
```

It is especially important when dealing with:

- Multibyte characters
    
- Wide characters
    
- Locale-specific character sets
    

---

# `LC_MONETARY`

Controls monetary formatting information.

You can retrieve this information with:

```c
localeconv();
```

---

# `LC_NUMERIC`

Controls numeric formatting.

For example, the decimal separator can differ between locales.

Conceptually:

```text
Some locales:

3.14
```

Other locales may use:

```text
3,14
```

This affects numeric formatting functions such as the `printf()` family.

---

# `LC_TIME`

Controls locale-specific date and time formatting.

It affects:

```c
strftime()
wcsftime()
```

For example, date names and formats can differ by locale.

---

# Getting Locale Information

Use:

```c
struct lconv *x = localeconv();
```

`localeconv()` returns a pointer to a `struct lconv` containing locale-specific numeric and monetary information.

Example:

```c
struct lconv *x = localeconv();
```

You can then access fields such as:

```c
x->currency_symbol
x->mon_decimal_point
x->frac_digits
```

---

# Important `struct lconv` Fields

Common monetary fields include:

|Field|Meaning|
|---|---|
|`mon_decimal_point`|Monetary decimal separator|
|`mon_thousands_sep`|Monetary thousands separator|
|`mon_grouping`|Digit grouping rules|
|`positive_sign`|Positive sign|
|`negative_sign`|Negative sign|
|`currency_symbol`|Local currency symbol|
|`frac_digits`|Digits after the decimal point|

Example conceptually:

```text
$1,234.56

currency_symbol = "$"
mon_thousands_sep = ","
mon_decimal_point = "."
frac_digits = 2
```

---

# Currency Symbol Position

These fields control whether the currency symbol appears before or after the amount:

```c
p_cs_precedes
n_cs_precedes
```

Where:

```text
p = positive
n = negative
```

Value:

```text
1 = Currency symbol before value
0 = Currency symbol after value
```

Examples:

```text
$100
```

or:

```text
100$
```

---

# Currency Spacing

These fields control spacing:

```c
p_sep_by_space
n_sep_by_space
```

Possible values:

|Value|Meaning|
|---|---|
|`0`|No space|
|`1`|Separate currency/sign and value with a space|
|`2`|Locale-specific separation involving the sign|

Conceptually:

```text
$100
```

versus:

```text
$ 100
```

---

# Sign Position

These fields control where the positive or negative sign appears:

```c
p_sign_posn
n_sign_posn
```

Possible meanings include:

|Value|Meaning|
|---|---|
|`0`|Surround amount and currency with parentheses|
|`1`|Sign before currency and value|
|`2`|Sign after currency and value|
|`3`|Sign directly before currency symbol|
|`4`|Sign directly after currency symbol|

Examples:

```text
-$100
$100-
($100)
```

---

# International Monetary Fields

`struct lconv` also contains international versions of monetary settings.

Examples:

```c
int_curr_symbol
int_frac_digits
int_p_cs_precedes
int_n_cs_precedes
```

These provide formatting information for international currency conventions.

Example:

```text
currency_symbol     = "$"
int_curr_symbol     = "USD "
```

---

# Unavailable Locale Values

Many `char` fields in `struct lconv` use:

```c
CHAR_MAX
```

from:

```c
#include <limits.h>
```

to indicate:

> This value is not available for the current locale.

Always account for this when interpreting locale information.

---

# Monetary Digit Grouping

The:

```c
mon_grouping
```

field describes how digits are grouped to the left of the decimal point.

For example:

```text
1,000,000,000
```

has groups of:

```text
3, 3, 3
```

---

# Repeating Groups

A grouping sequence can use:

```text
0
```

to mean:

> Repeat the previous group size.

Example:

```text
3, 0
```

means:

```text
3 digits
3 digits
3 digits
3 digits
...
```

Conceptually:

```text
1,000,000,000,000
  └─3─┘└─3─┘└─3─┘
```

---

# Stopping Digit Grouping

`CHAR_MAX` in a grouping sequence means:

> No more grouping should occur.

Example conceptually:

```text
3, 2, CHAR_MAX
```

After the specified groups, remaining digits are not grouped.

If the first value is:

```text
CHAR_MAX
```

then no digit grouping occurs.

---

# Locale Mental Model

```text
Program
   │
   ▼
setlocale()
   │
   ▼
Selected Locale
   │
   ├── Character Rules
   ├── Number Rules
   ├── Currency Rules
   ├── Date/Time Rules
   └── String Sorting Rules
```

The locale changes how standard library functions interpret and format data.

---

# Why `LC_CTYPE` Is Important

`LC_CTYPE` is especially important for internationalized programs.

It affects how C understands characters.

Conceptually:

```text
Character
    │
    ▼
LC_CTYPE Rules
    │
    ▼
Classification / Conversion
```

This becomes particularly important with:

- UTF-8
    
- Multibyte characters
    
- Wide characters
    
- Unicode-related processing
    

---

# Reverse Engineering Perspective

Locale settings can change program behavior without changing the program's code.

For example:

```text
Same program
     │
     ├── Locale A → 3.14
     │
     └── Locale B → 3,14
```

When reversing or debugging software, environment-dependent behavior may therefore come from:

- Locale settings
    
- Character encoding
    
- Date/time formatting
    
- String comparison rules
    

Functions such as:

```c
setlocale()
localeconv()
strcoll()
```

can indicate locale-dependent behavior in a binary.

---

# Mental Model

```text
Environment Locale
        │
        ▼
setlocale()
        │
        ▼
C Runtime Locale
        │
        ├── LC_CTYPE
        ├── LC_NUMERIC
        ├── LC_MONETARY
        ├── LC_TIME
        └── LC_COLLATE
        │
        ▼
Program Output and Behavior
```

---

# Long-Term Takeaways

- A **locale** controls language- and region-specific program behavior.
    
- Include `<locale.h>` to use locale facilities.
    
- Use `setlocale()` to select a locale.
    
- `setlocale(LC_ALL, "")` uses the environment's locale.
    
- `"C"` is the standard default C locale.
    
- Specific locale names such as `"en_US.UTF-8"` are not fully portable.
    
- `LC_ALL` changes all locale categories.
    
- `LC_COLLATE` controls locale-aware string ordering.
    
- `LC_CTYPE` controls character classification and conversion.
    
- `LC_MONETARY` controls monetary locale information.
    
- `LC_NUMERIC` controls numeric formatting, including decimal separators.
    
- `LC_TIME` controls date and time formatting.
    
- `localeconv()` returns a `struct lconv` containing numeric and monetary settings.
    
- `mon_grouping` describes locale-specific digit grouping.
    
- `0` in a grouping sequence repeats the previous group size.
    
- `CHAR_MAX` can indicate unavailable information or the end of grouping.
    
- International fields in `struct lconv` provide international monetary formatting information.
    
- Locale settings can significantly affect program behavior, especially with multibyte and wide characters.