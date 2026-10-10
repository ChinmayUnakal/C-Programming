
## What is a Struct?

A **struct** is a user-defined type that groups multiple related values, possibly of different types, under one name.

Example:

```c
struct Person
{
    char name[50];
    int age;
    float height;
};
```

A `Person` contains:

```text
Person
 ├── name
 ├── age
 └── height
```

---

# Why Use Structs?

Structs are used to represent a single object containing related data.

Example:

```text
Person
 ├── Name
 ├── Age
 └── Height
```

Instead of using separate variables:

```c
char name[50];
int age;
float height;
```

---

# Declaring a Struct

Define the struct type:

```c
struct Person
{
    char name[50];
    int age;
};
```

Create a variable:

```c
struct Person p;
```

Access members with the **dot operator (`.`)**:

```c
p.age = 23;
```

---

# Struct Initializers

Initialize members in order:

```c
struct Person p = {"Alice", 25};
```

Or use designated initializers:

```c
struct Person p =
{
    .name = "Alice",
    .age = 25
};
```

Designated initializers make the code clearer.

---

# Passing Structs to Functions

## Passing by Value

The entire struct is copied.

```c
void print_person(struct Person p)
{
    printf("%d\n", p.age);
}
```

Changes to `p` do **not** affect the original struct.

---

## Passing by Pointer

Pass the address of the struct:

```c
void increment_age(struct Person *p)
{
    p->age++;
}
```

Call it with:

```c
increment_age(&person);
```

Changes affect the original struct.

---

# The Arrow Operator (`->`)

The arrow operator accesses a struct member through a **pointer**.

Given:

```c
struct Person *p;
```

Use:

```c
p->age
```

This is equivalent to:

```c
(*p).age
```

Remember:

```text
Struct variable  → .
Struct pointer   → ->
```

---

# Copying Structs

Structs can be copied directly.

```c
struct Person a = {"Alice", 25};
struct Person b;

b = a;
```

After copying:

```text
b contains a copy of a
```

The members are copied.

---

# Returning Structs

Functions can return structs.

```c
struct Person create_person(void)
{
    struct Person p = {"Alice", 25};

    return p;
}
```

Usage:

```c
struct Person person = create_person();
```

---

# Comparing Structs

Structs cannot generally be compared directly using:

```c
a == b    // Invalid for structs
```

Compare individual members instead:

```c
if (a.age == b.age)
{
    // same age
}
```

For strings:

```c
strcmp(a.name, b.name) == 0
```

Do **not** use `==` to compare string contents.

---

# Struct Memory Layout

Example:

```c
struct Person
{
    char grade;
    int age;
};
```

Conceptually:

```text
+-------+
| grade |  1 byte
+-------+
|       |
|padding|  Possible
|       |
+-------+
|  age  |  4 bytes
+-------+
```

The compiler may add **padding** between members for alignment.

Therefore:

```c
sizeof(struct Person)
```

may be larger than the sum of its member sizes.

---

# Reverse Engineering Perspective (x86-64 Assembly)

## Struct Access

C:

```c
person.age
```

becomes an access at a fixed **offset** from the struct's base address.

Conceptually:

```asm
mov eax, [rbx + 4]
```

Where:

```text
rbx       = address of struct
rbx + 4   = address of age
```

---

## Struct Offsets

Reverse engineers often see:

```asm
mov eax, [rcx]
mov edx, [rcx + 4]
mov r8,  [rcx + 8]
```

This may indicate a struct-like object:

```text
Offset 0x00 → Member 1
Offset 0x04 → Member 2
Offset 0x08 → Member 3
```

Reconstructing these offsets is a major part of reverse engineering.

---

## Struct Pointers

C:

```c
p->age
```

Conceptually becomes:

```asm
mov eax, [rcx + OFFSET]
```

The CPU does not know about structs.

It only sees:

```text
Base Address + Offset
```

---

## Passing Structs

Small structs may be passed in registers.

Larger structs may be passed through memory, depending on the calling convention and ABI.

Conceptually:

```text
Function
   │
   ├── Registers
   │
   └── Memory / Stack
```

---

# Mental Model

A struct is a block of related values:

```text
Base Address
     │
     ▼

+----------------+
| Member 1       |  Offset 0
+----------------+
| Member 2       |  Offset X
+----------------+
| Member 3       |  Offset Y
+----------------+
```

Accessing a member:

```text
Struct Base Address + Member Offset
```

---

# Long-Term Takeaways

- A struct groups related values under one type.
    
- Struct members can have different types.
    
- Use `.` for a struct variable.
    
- Use `->` for a pointer to a struct.
    
- `p->member` is equivalent to `(*p).member`.
    
- Structs can be copied, passed by value, and returned from functions.
    
- Structs generally cannot be compared directly with `==`.
    
- The compiler may add padding for alignment.
    
- In assembly, structs appear as **memory accesses using fixed offsets**.
    
- A key reverse engineering pattern is:
    

```text
Object Base Address + Offset = Struct Member
```