C provides date and time functionality through the standard `<time.h>` header.

```c
#include <time.h>
```

It provides facilities for:

- Obtaining the current calendar time
    
- Representing time in different forms
    
- Converting between time representations
    
- Formatting dates and times
    
- Measuring elapsed time
    
- Working with higher-resolution timestamps
    

---

# Quick Terminology and Information

There are several different concepts involved in C's time functionality.

### Calendar Time

**Calendar time** represents a point in time, such as:

```text
September 4, 2026, 12:30:00
```

C commonly represents calendar time using:

```c
time_t
```

---

### Broken-Down Time

A calendar time can be separated into individual components such as:

```text
Year
Month
Day
Hour
Minute
Second
```

C represents this form using:

```c
struct tm
```

---

### Time Interval

Sometimes we are interested in the amount of time between two points rather than the actual date.

For example:

```text
Start: 10:00:00
End:   10:05:30

Difference: 330 seconds
```

C provides facilities for calculating such differences.

---

### Resolution

**Resolution** refers to the smallest time unit that a particular representation can distinguish.

For example:

```text
seconds
milliseconds
nanoseconds
```

Different C time functions provide different levels of resolution.

---

# Date Types

The main types used by `<time.h>` are:

```c
time_t
struct tm
struct timespec
```

---

## `time_t`

`time_t` is used to represent calendar time.

Example:

```c
time_t current;
```

The exact underlying representation of `time_t` is implementation-defined.

Therefore, you should not assume that it is always:

```text
64-bit integer
```

or:

```text
32-bit integer
```

Instead, treat it as the implementation's time representation.

---

# `struct tm`

`struct tm` stores a broken-down representation of a date and time.

Its members include:

```c
struct tm
{
    int tm_sec;    // seconds
    int tm_min;    // minutes
    int tm_hour;   // hours
    int tm_mday;   // day of month
    int tm_mon;    // month
    int tm_year;   // years since 1900
    int tm_wday;   // day of week
    int tm_yday;   // day of year
    int tm_isdst;  // daylight saving time information
};
```

Some fields use values that may initially look unusual.

For example:

```text
tm_mon
```

ranges from:

```text
0 → January
1 → February
...
11 → December
```

And:

```text
tm_year
```

contains the number of years since 1900.

Therefore:

```text
tm_year = 126
```

represents:

```text
2026
```

---

# `struct timespec`

`struct timespec` provides a representation with finer resolution than `time_t`.

It contains:

```c
struct timespec
{
    time_t tv_sec;
    long   tv_nsec;
};
```

where:

```text
tv_sec  → whole seconds
tv_nsec → additional nanoseconds
```

Conceptually:

```text
Time
 │
 ├── tv_sec
 │      seconds
 │
 └── tv_nsec
        fractional second
```

---

# Getting the Current Time

The `time()` function can obtain the current calendar time.

Syntax:

```c
time_t time(time_t *timer);
```

Example:

```c
time_t current;

time(&current);
```

It can also be used as:

```c
time_t current = time(NULL);
```

Passing `NULL` means that only the return value is needed.

---

# Initialization and Conversion Between Types

The two major representations you'll work with are:

```text
time_t
struct tm
```

They represent related information in different forms.

```text
                 time_t
                   │
          ┌────────┴────────┐
          │                 │
          ▼                 ▼
     localtime()         gmtime()
          │                 │
          ▼                 ▼
      struct tm         struct tm
```

You can then convert a `struct tm` back into `time_t`.

---

# Converting `time_t` to `struct tm`

There are two important functions:

```c
localtime()
gmtime()
```

---

## `localtime()`

`localtime()` converts a `time_t` value into a broken-down time representing the **local time**.

Example:

```c
time_t current = time(NULL);

struct tm *local = localtime(&current);
```

The returned pointer refers to a `struct tm` containing fields such as:

```text
Year
Month
Day
Hour
Minute
Second
```

---

## `gmtime()`

`gmtime()` also converts `time_t` into `struct tm`, but the result represents **Coordinated Universal Time (UTC)** rather than local time.

Example:

```c
time_t current = time(NULL);

struct tm *utc = gmtime(&current);
```

Conceptually:

```text
time_t
  │
  ├── localtime() → local time
  │
  └── gmtime()    → UTC
```

---

# Important: Returned `struct tm` Storage

`localtime()` and `gmtime()` return pointers to `struct tm`.

You should not assume that the returned structure is newly allocated every time.

The implementation may use internal storage.

Therefore, if you need to preserve the result while performing another time conversion, copy the structure:

```c
struct tm copy = *local;
```

rather than relying on the returned pointer remaining unchanged.

---

# Converting `struct tm` to `time_t`

The reverse conversion can be performed using:

```c
mktime()
```

Example:

```c
struct tm date = {0};

date.tm_year = 2026 - 1900;
date.tm_mon  = 8;
date.tm_mday = 4;

time_t t = mktime(&date);
```

`mktime()` interprets the `struct tm` as a **local calendar time** and converts it into `time_t`.

Conceptually:

```text
struct tm
    │
    │ mktime()
    ▼
  time_t
```

---

# `mktime()` Can Normalize Values

`mktime()` can normalize fields of the supplied `struct tm`.

For example, if a field contains a value outside its usual range, the function can adjust the other fields accordingly.

This means `mktime()` is useful not only for conversion but also for normalizing a broken-down local date/time.

After the call, the `struct tm` may contain adjusted values.

---

# Formatted Date Output

A `struct tm` contains separate fields for the components of a date.

Instead of manually printing every field, C provides:

```c
strftime()
```

Syntax:

```c
strftime(buffer, size, format, timeptr);
```

Example:

```c
char buffer[100];

strftime(
    buffer,
    sizeof(buffer),
    "%Y-%m-%d %H:%M:%S",
    local
);

printf("%s\n", buffer);
```

A possible result is:

```text
2026-09-04 12:30:00
```

---

# Common `strftime()` Format Specifiers

Some useful conversion specifiers are:

|Specifier|Meaning|
|---|---|
|`%Y`|Four-digit year|
|`%m`|Month as `01`–`12`|
|`%d`|Day of month|
|`%H`|Hour in 24-hour format|
|`%M`|Minute|
|`%S`|Second|
|`%A`|Full weekday name|
|`%a`|Abbreviated weekday|
|`%B`|Full month name|
|`%b`|Abbreviated month|
|`%j`|Day of year|
|`%z`|UTC offset|
|`%Z`|Time-zone name|

Example:

```c
strftime(
    buffer,
    sizeof(buffer),
    "%A, %B %d, %Y",
    local
);
```

could produce:

```text
Friday, September 04, 2026
```

---

# More Resolution with `timespec_get()`

`time()` works with `time_t`, but sometimes a program needs a timestamp with additional resolution.

C provides:

```c
timespec_get()
```

which can fill a `struct timespec`.

Example:

```c
struct timespec ts;

timespec_get(&ts, TIME_UTC);
```

Afterward:

```text
ts.tv_sec
    ↓
whole seconds

ts.tv_nsec
    ↓
nanoseconds within that second
```

For example:

```text
tv_sec  = 1788525000
tv_nsec = 123456789
```

represents a time containing:

```text
1788525000 seconds
+ 0.123456789 seconds
```

The actual resolution and accuracy available are implementation-dependent. Having a `tv_nsec` field does not necessarily mean the underlying clock can measure time accurately to a nanosecond.

---

# `struct timespec` vs `time_t`

```text
time_t
└── calendar time representation

struct timespec
├── tv_sec
└── tv_nsec
```

Use `time_t` when you mainly need to represent calendar time.

Use `struct timespec` when you need a representation that includes fractional seconds.

---

# Differences Between Times

To determine how much time has passed between two `time_t` values, C provides:

```c
difftime()
```

Syntax:

```c
double difftime(time_t end, time_t start);
```

Example:

```c
time_t start = time(NULL);

/* some operation */

time_t end = time(NULL);

double elapsed = difftime(end, start);
```

The result represents:

```text
end - start
```

in seconds.

---

# Why Use `difftime()`?

You might be tempted to write:

```c
double elapsed = end - start;
```

But `time_t` is an implementation-defined type.

`difftime()` provides the standard way to calculate the difference between two `time_t` values.

Example:

```c
double elapsed = difftime(end, start);
```

If:

```text
start = 1000
end   = 1350
```

then:

```text
elapsed = 350 seconds
```

---

# Date/Time Conversion Flow

A useful way to remember the main functions is:

```text
                 time()
                   │
                   ▼
                time_t
                /     \
               /       \
              ▼         ▼
       localtime()     gmtime()
              │         │
              ▼         ▼
          struct tm  struct tm
              │
              │
              ▼
           strftime()
              │
              ▼
       Formatted string
```

And the reverse direction:

```text
struct tm
    │
    │ mktime()
    ▼
  time_t
```

For higher-resolution timestamps:

```text
timespec_get()
      │
      ▼
struct timespec
 ├── tv_sec
 └── tv_nsec
```

For differences:

```text
time_t ─────┐
            ├── difftime() → seconds
time_t ─────┘
```

---

# Reverse Engineering Perspective

Date and time handling appears frequently in real-world binaries.

You may encounter:

```c
time_t
struct tm
struct timespec
```

when analyzing programs that use:

- File timestamps
    
- Log timestamps
    
- Timeouts
    
- Expiration dates
    
- Event scheduling
    
- Network protocols
    
- Authentication systems
    
- File metadata
    
- Performance measurements
    

Recognizing functions such as:

```text
time()
localtime()
gmtime()
mktime()
strftime()
timespec_get()
difftime()
```

can therefore provide useful clues about what a particular part of a binary is doing.

For example, a sequence that obtains a timestamp, converts it into `struct tm`, and formats it with `strftime()` is likely constructing a human-readable date/time string.

---

# Long-Term Takeaways

- `<time.h>` provides C's standard date and time functionality.
    
- `time_t` is used to represent calendar time.
    
- `struct tm` stores a broken-down date/time representation.
    
- `struct timespec` provides seconds plus a fractional-second component.
    
- `time(NULL)` obtains the current calendar time.
    
- `localtime()` converts `time_t` into local broken-down time.
    
- `gmtime()` converts `time_t` into UTC broken-down time.
    
- `mktime()` converts a local `struct tm` back into `time_t`.
    
- `strftime()` converts a `struct tm` into a formatted character string.
    
- `timespec_get()` can obtain a timestamp with a seconds/nanoseconds representation.
    
- Nanosecond representation does **not** guarantee nanosecond clock accuracy.
    
- `difftime()` calculates the difference between two `time_t` values in seconds.
    
- `time_t`'s exact underlying representation is implementation-defined.
    
- `tm_mon` is zero-based, while `tm_year` counts years from 1900.
    
- `localtime()` and `gmtime()` return pointers to implementation-managed `struct tm` storage, so copy the structure if you need to preserve it across subsequent conversions.
    
- In reverse engineering, time-related functions can reveal code dealing with **timestamps, expiration logic, logging, scheduling, timeouts, and file metadata**.