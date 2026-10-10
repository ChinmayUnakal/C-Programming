# Initializers of Nested `struct`s and Arrays

A **nested struct** is a struct containing another struct, and structs can also contain arrays.

Example:

```c
struct Point
{
    int x;
    int y;
};

struct Rectangle
{
    struct Point top_left;
    struct Point bottom_right;
};
```

Initialization:

```c
struct Rectangle r = {
    {0, 10},
    {20, 0}
};
```

---

# Initializing Nested Members

You can initialize members in the order they appear.

```c
struct Person
{
    char name[20];
    int age;
};

struct Company
{
    struct Person CEO;
    int employees;
};

struct Company c = {
    {"Alice", 30},
    500
};
```

Mental model:

```text
Company
├── CEO
│   ├── name
│   └── age
│
└── employees
```

---

# Designated Initializers

You can explicitly name members:

```c
struct Company c = {
    .CEO = {
        .name = "Alice",
        .age = 30
    },
    .employees = 500
};
```

This improves readability and avoids depending entirely on member order.

---

# Arrays Inside Structs

Example:

```c
struct Student
{
    char name[20];
    int marks[3];
};
```

Initialization:

```c
struct Student s = {
    "Chinmay",
    {90, 85, 95}
};
```

Access:

```c
s.marks[0];
s.marks[1];
```

---

# Anonymous `struct`s

An **anonymous struct** is a struct without a tag name.

Example:

```c
struct
{
    int x;
    int y;
} point;
```

Here:

```text
struct type
    ↓
no tag name

point
    ↓
variable of that type
```

You can use `point`:

```c
point.x = 10;
point.y = 20;
```

But you cannot easily refer to that exact struct type later by a tag.

---

# Anonymous Nested Structs

Anonymous structs can also be members of another struct.

Example:

```c
struct Person
{
    char name[20];

    struct
    {
        int day;
        int month;
        int year;
    } birth;
};
```

Access:

```c
person.birth.day = 10;
```

---

# Self-Referential `struct`s

A **self-referential struct** contains a pointer to its own type.

Example:

```c
struct Node
{
    int value;
    struct Node *next;
};
```

This is essential for dynamic data structures such as:

- Linked lists
    
- Trees
    
- Graphs
    

---

# Why Use a Pointer?

This would not work:

```c
struct Node
{
    int value;
    struct Node next;
};
```

Because `next` would contain another complete `Node`, which contains another `Node`, forever.

A pointer has a known fixed size:

```text
Node
├── value
└── next ─────► another Node
```

---

# Flexible Array Members

A **flexible array member** is an array at the end of a struct with no specified size.

Example:

```c
struct Packet
{
    size_t length;
    unsigned char data[];
};
```

The flexible array must be the **last member**.

---

# Allocating a Flexible Array

Example:

```c
struct Packet *p =
    malloc(sizeof(struct Packet) + 100);
```

Now there is room for:

```text
Packet
├── length
└── data[100]
```

You can access:

```c
p->data[0];
```

---

# Why Flexible Arrays Are Useful

They allow one allocation for:

```text
Struct Metadata + Variable-Length Data
```

Common uses:

- Network packets
    
- File formats
    
- Custom memory structures
    
- Variable-sized objects
    

---

# Padding Bytes

The compiler may insert **padding bytes** between struct members to satisfy alignment requirements.

Example:

```c
struct Example
{
    char a;
    int b;
};
```

Possible memory layout:

```text
Offset

0       a
1-3     padding
4-7     b
```

The struct may occupy:

```text
8 bytes
```

instead of:

```text
1 + 4 = 5 bytes
```

---

# Why Padding Exists

CPUs generally access properly aligned data more efficiently.

For example, an `int` may need an address aligned to:

```text
4 bytes
```

The compiler adds padding to achieve this.

---

# Padding and Reverse Engineering

Padding is extremely important when reconstructing structures.

Suppose assembly accesses:

```asm
mov eax, [rcx + 4]
```

This may indicate:

```text
offset 0 → char member
offset 1-3 → padding
offset 4 → int member
```

Incorrectly assuming no padding can produce completely wrong struct definitions.

---

# `offsetof`

`offsetof` gives the byte offset of a struct member.

It is defined in:

```c
#include <stddef.h>
```

Example:

```c
struct Example
{
    char a;
    int b;
};

size_t offset = offsetof(struct Example, b);
```

The result might be:

```text
4
```

---

# Why `offsetof` Is Useful

It helps determine:

- Struct layout
    
- Member offsets
    
- Padding
    
- Binary format layouts
    

Example:

```c
printf("%zu\n", offsetof(struct Example, b));
```

---

# Reverse Engineering Perspective

When you see:

```asm
mov eax, [rcx + 0x10]
```

you can think:

```text
RCX
 │
 ▼
struct object
 │
 └── offset 0x10 → possible member
```

`offsetof()` is the C-level equivalent of determining those offsets.

---

# Fake OOP

C has no built-in classes or inheritance, but you can imitate some OOP concepts using:

- Structs
    
- Function pointers
    
- Composition
    

Example:

```c
struct Animal
{
    void (*speak)(void);
};
```

Different objects can use different functions:

```c
void dog_speak(void)
{
    printf("Woof\n");
}
```

```c
struct Animal dog = {
    .speak = dog_speak
};
```

Call:

```c
dog.speak();
```

---

# Fake OOP Mental Model

```text
Object
├── Data
└── Function Pointers
       │
       ▼
    Methods
```

This pattern is common in:

- Operating systems
    
- Device drivers
    
- GUI libraries
    
- Game engines
    
- C libraries
    

---

# Reverse Engineering Perspective

A struct containing multiple function pointers may look like:

```text
Object
├── data
├── function pointer
├── function pointer
└── function pointer
```

This can resemble a **vtable-like structure**.

When reversing C or C++ binaries, recognizing groups of function pointers is very useful.

---

# Bit-Fields

A **bit-field** allows a struct member to occupy a specific number of bits.

Example:

```c
struct Flags
{
    unsigned int read : 1;
    unsigned int write : 1;
    unsigned int execute : 1;
};
```

Each member uses one bit.

---

# Bit-Field Memory Model

Conceptually:

```text
Bit

0       read
1       write
2       execute
3-...   unused
```

Usage:

```c
flags.read = 1;
flags.write = 0;
```

---

# Why Use Bit-Fields?

Useful for compactly representing:

- Flags
    
- Hardware registers
    
- Protocol fields
    
- Permissions
    

However, exact layout can be implementation-dependent.

---

# Non-Adjacent Bit-Fields

Bit-fields do not always occupy one simple, predictable contiguous layout across all compilers.

Example:

```c
struct Example
{
    unsigned int a : 3;
    int x;
    unsigned int b : 3;
};
```

`a` and `b` are separated by:

```c
int x;
```

So they are **non-adjacent bit-fields**.

Their storage and exact layout should not be assumed purely from source without considering the compiler and ABI.

---

# Signed or Unsigned `int`s

Bit-fields can use integer types such as:

```c
signed int
unsigned int
int
```

Example:

```c
struct Example
{
    unsigned int x : 3;
};
```

An unsigned 3-bit field can generally represent:

```text
0 to 7
```

A signed field has a different range.

---

# Important: Plain `int`

Example:

```c
struct Example
{
    int x : 3;
};
```

Whether a plain `int` bit-field behaves as signed or unsigned can be implementation-defined.

For predictable behavior, explicitly use:

```c
signed int
```

or:

```c
unsigned int
```

---

# Unnamed Bit-Fields

An unnamed bit-field has no member name.

Example:

```c
struct Example
{
    unsigned int a : 3;
    unsigned int   : 2;
    unsigned int b : 3;
};
```

The unnamed field reserves:

```text
2 bits
```

Mental model:

```text
aaa__bbb
```

The two `_` bits are unused padding/reserved bits.

---

# Zero-Width Unnamed Bit-Fields

A zero-width unnamed bit-field forces the next bit-field to begin in a new allocation unit.

Example:

```c
struct Example
{
    unsigned int a : 3;
    unsigned int   : 0;
    unsigned int b : 3;
};
```

The `: 0` acts as a boundary.

Conceptually:

```text
Storage Unit 1
└── a

Storage Unit 2
└── b
```

This can be useful when matching specific layouts, but exact details remain implementation-dependent.

---

# Unions

A **union** allows multiple members to share the same memory location.

Example:

```c
union Data
{
    int i;
    float f;
    char bytes[4];
};
```

Unlike a struct:

```text
Struct
├── i
├── f
└── bytes
```

a union is:

```text
Union
┌──────────────┐
│ same memory  │
├──────────────┤
│ i            │
│ f            │
│ bytes        │
└──────────────┘
```

---

# Union Size

A union must be large enough to store its largest member, subject to alignment requirements.

Example:

```c
union Data
{
    char c;
    int i;
};
```

The union will generally be large enough for an `int`.

---

# Using a Union

Example:

```c
union Data d;

d.i = 42;
```

Now the union's memory contains the representation of:

```text
42 as an int
```

If you then write:

```c
d.f = 3.14f;
```

the same memory is overwritten with the representation of a `float`.

---

# Unions and Type Punning

**Type punning** means interpreting the same bits as another type.

Example:

```c
union Data
{
    float f;
    unsigned int i;
};
```

You might write:

```c
d.f = 1.0f;
```

and inspect:

```c
d.i
```

to examine the floating-point representation.

---

# Important Warning About Type Punning

Union-based type punning has subtle language and implementation rules.

For portable low-level code, `memcpy()` is often a safer way to copy an object's representation into another object of the same size.

Example:

```c
float f = 1.0f;
unsigned int i;

memcpy(&i, &f, sizeof i);
```

This preserves the raw bytes without relying on questionable aliasing assumptions.

---

# Reverse Engineering Perspective

Type punning is extremely common conceptually in reverse engineering.

You may see the same bytes interpreted as:

```text
Bytes
 │
 ├── integer
 ├── float
 ├── pointer
 └── flags
```

The raw memory does not contain type information.

The type depends on how instructions interpret those bytes.

---

# Pointers to Unions

A pointer can point to a union.

Example:

```c
union Data
{
    int i;
    float f;
};

union Data d;
union Data *p = &d;
```

Access members using:

```c
p->i
p->f
```

Equivalent to:

```c
(*p).i
(*p).f
```

---

# Memory View

```text
p
│
▼
union Data
┌──────────────┐
│ shared bytes │
└──────────────┘
```

The pointer points to the union's shared storage.

---

# Common Initial Sequences in Unions

Two struct types can have matching initial members.

Example:

```c
struct A
{
    int type;
    int value;
};

struct B
{
    int type;
    int value;
    float extra;
};
```

Their common initial sequence is:

```text
type
value
```

---

# Union Example

```c
union Object
{
    struct A a;
    struct B b;
};
```

The common initial members can be used to inspect shared initial data under the rules for common initial sequences.

This is useful for implementing tagged data structures.

---

# Tagged Union Pattern

Example:

```c
struct IntValue
{
    int type;
    int value;
};

struct FloatValue
{
    int type;
    float value;
};
```

A tag tells the program how to interpret the data.

Mental model:

```text
type
 │
 ├── INT   → interpret as integer
 │
 └── FLOAT → interpret as float
```

---

# Unions and Unnamed Structs

Anonymous/unnamed structs can be used inside unions.

Example:

```c
union Value
{
    int i;
    float f;

    struct
    {
        unsigned char a;
        unsigned char b;
        unsigned char c;
        unsigned char d;
    } bytes;
};
```

Now the same memory can be accessed as:

```c
value.i
value.f
value.bytes.a
```

---

# Why This Is Useful

It provides multiple views of the same memory:

```text
Same Bytes
    │
    ├── Integer View
    ├── Float View
    └── Byte View
```

This is useful for:

- Binary parsing
    
- Hardware interfaces
    
- Protocols
    
- Low-level programming
    

---

# Passing `struct`s

Structs can be passed to functions by value.

Example:

```c
struct Point
{
    int x;
    int y;
};

void print_point(struct Point p)
{
    printf("%d %d\n", p.x, p.y);
}
```

Calling:

```c
struct Point p = {10, 20};

print_point(p);
```

The function receives a copy of the struct.

---

# Passing a Pointer to a `struct`

For large structs, passing a pointer can avoid copying the entire object.

Example:

```c
void move(struct Point *p)
{
    p->x++;
}
```

Calling:

```c
move(&p);
```

Now the original struct can be modified.

---

# Returning `struct`s

Functions can also return structs.

Example:

```c
struct Point create_point(void)
{
    struct Point p = {10, 20};
    return p;
}
```

Usage:

```c
struct Point p = create_point();
```

Returning a struct by value is valid C.

---

# Passing and Returning Unions

Unions can also be passed and returned by value.

Example:

```c
union Data create_data(void)
{
    union Data d;

    d.i = 42;

    return d;
}
```

Usage:

```c
union Data x = create_data();
```

The union object is passed or returned according to the platform's ABI.

---

# Reverse Engineering Perspective (x86-64)

Structs and unions do not always correspond to one simple register pattern.

Depending on:

- Size
    
- Members
    
- ABI
    
- Compiler
    
- Optimization
    

they may be:

```text
Passed in registers
Passed on the stack
Passed indirectly through pointers
Returned in registers
Returned through hidden memory pointers
```

---

# Small Struct Example

C:

```c
struct Point
{
    int x;
    int y;
};
```

A small struct might be passed using registers.

Conceptually:

```text
Caller
   │
   ├── x
   └── y
       │
       ▼
Registers / ABI-defined locations
       │
       ▼
Function
```

The exact registers depend on the platform ABI.

---

# Large Struct Example

A larger struct may effectively be passed indirectly.

Conceptually:

```text
Caller
   │
   ▼
Address of Struct
   │
   ▼
Function
```

This can look similar to passing a pointer when analyzing assembly, although the exact ABI mechanism matters.

---

# Struct Member Access in Assembly

C:

```c
p.x
```

may become conceptually:

```asm
mov eax, [rcx]
```

C:

```c
p.y
```

might use:

```asm
mov eax, [rcx + 4]
```

The offsets depend on:

```text
Member sizes
+ Alignment
+ Padding
```

---

# Union Access in Assembly

The CPU does not know:

> This is a union.

It only sees memory.

For example, the same address may be loaded and interpreted differently:

```text
Memory Address
     │
     ├── integer instruction
     │
     └── floating-point instruction
```

A union is mainly a C-level description of multiple interpretations of the same storage.

---

# Mental Model

```text
STRUCT
Members have separate storage

┌───────┬───────┬───────┐
│   A   │   B   │   C   │
└───────┴───────┴───────┘


UNION
Members share storage

┌───────────────────────┐
│ A or B or C            │
│ Same memory            │
└───────────────────────┘
```

---

# Long-Term Takeaways

- Nested structs and arrays can be initialized using nested braces or designated initializers.
    
- Anonymous structs have no tag name.
    
- Self-referential structs use pointers to their own type.
    
- Flexible array members provide variable-length data at the end of a struct.
    
- Flexible arrays require extra memory allocation.
    
- Struct padding is inserted to satisfy alignment requirements.
    
- `offsetof()` reveals the byte offset of a struct member.
    
- Padding and offsets are extremely important in reverse engineering.
    
- C can imitate OOP using structs, composition, and function pointers.
    
- Bit-fields store values using a specified number of bits.
    
- Bit-field layout can be implementation-dependent.
    
- Explicit `signed int` or `unsigned int` is safer than relying on plain `int` bit-field signedness.
    
- Unnamed bit-fields reserve unused bits.
    
- A zero-width unnamed bit-field forces a new allocation boundary.
    
- Union members share the same memory.
    
- A union is generally large enough for its largest member, considering alignment.
    
- Type punning interprets the same bits using different types and requires care for portability.
    
- `memcpy()` is often a safer technique for examining an object's raw representation.
    
- Pointers to unions use `->` to access union members.
    
- Common initial sequences allow certain shared initial struct members to be accessed through union members.
    
- Unions can provide multiple views of the same bytes.
    
- Structs and unions can both be passed and returned by value.
    
- The exact assembly representation depends heavily on the ABI, compiler, size, and optimization.
    
- When reverse engineering, member offsets often reveal the structure of a `struct`.
    
- The CPU does not know about C structs or unions—it only operates on bytes, registers, and memory addresses.  
