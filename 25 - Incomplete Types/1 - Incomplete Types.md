# Incomplete Types

An **incomplete type** is a type whose size or complete structure is not yet known at the point where it is being used.

The compiler knows that the type exists, but does not yet know enough about it to determine its complete layout.

Example:

```c
struct Person;
```

At this point, the compiler knows that `struct Person` is a structure type, but it does not know:

- What members it contains
    
- How large it is
    
- How those members are arranged
    

The structure can be completed later:

```c
struct Person
{
    int age;
    char name[50];
};
```

After this definition, `struct Person` is a **complete type**.

---

# What Can You Do With an Incomplete Type?

An incomplete type can be used in situations where the compiler does not need to know its size or complete layout.

For example, you can declare a pointer:

```c
struct Person;

struct Person *p;
```

This works because the compiler knows that `p` is a pointer to a `struct Person`.

The size of the structure itself is not required to determine the size of the pointer.

However, you cannot create an actual object of the incomplete type:

```c
struct Person p;   // Error
```

The compiler does not know how much memory `p` requires.

---

# Complete vs Incomplete Type

```text
Incomplete:

struct Person;

        ↓

Compiler knows:
"Person exists"

Compiler does NOT know:
"How large is Person?"
```

After the definition:

```c
struct Person
{
    int age;
    char name[50];
};
```

```text
Complete:

struct Person

        ↓

Compiler knows:
- members
- layout
- size
- alignment
```

---

# Use Case: Self-Referential Structures

One of the most important uses of incomplete types is creating **self-referential structures**.

A self-referential structure contains a pointer to another object of the same structure type.

Example:

```c
struct Node
{
    int value;
    struct Node *next;
};
```

Here:

```c
struct Node *next;
```

is a pointer to another `struct Node`.

This is commonly used to create:

- Linked lists
    
- Trees
    
- Graphs
    
- Other dynamic data structures
    

---

# Why Is `struct Node *` Allowed?

When the compiler encounters:

```c
struct Node
{
    int value;
    struct Node *next;
};
```

the structure is still being defined.

The compiler does not yet know the complete size of `struct Node`.

However, it **does know the size of a pointer**.

Therefore:

```c
struct Node *next;
```

can be used.

The pointer does not contain another complete `struct Node` directly. It only stores an address.

Conceptually:

```text
struct Node
+------------------+
| value            |
+------------------+
| next ------------|------+
+------------------+      |
                          |
                          v
                    struct Node
                    +----------+
                    | value    |
                    +----------+
                    | next ----|----> ...
                    +----------+
```

---

# What Would Not Work?

This would be invalid:

```c
struct Node
{
    int value;
    struct Node next;
};
```

The problem is that `next` would contain another complete `struct Node`.

That would require the structure to contain itself directly:

```text
Node
 └── Node
      └── Node
           └── Node
                └── ...
```

The size could never be determined.

Using a pointer solves this:

```c
struct Node
{
    int value;
    struct Node *next;
};
```

The structure contains only a pointer, whose size is known.

---

# Incomplete Type Error Messages

Using an incomplete type where a complete type is required causes a compiler error.

For example:

```c
struct Person;

int main(void)
{
    struct Person p;
}
```

The compiler cannot allocate `p` because the size of `struct Person` is unknown.

You may see an error similar to:

```text
error: storage size of 'p' isn't known
```

or:

```text
error: incomplete type 'struct Person'
```

The exact wording depends on the compiler.

---

# Common Operations That Require a Complete Type

The compiler generally needs a complete type when it must know the object's size or layout.

For example:

### Creating an object

```c
struct Person p;
```

### Accessing members

```c
p.age;
```

### Using `sizeof`

```c
sizeof(struct Person);
```

These operations require the compiler to know the structure's complete definition.

A pointer to the incomplete type is still allowed:

```c
struct Person *p;
```

---

# Other Incomplete Types

Structures are not the only incomplete types in C.

Several types can be incomplete.

## Incomplete Structure Types

```c
struct Data;
```

The structure has been declared, but its members have not yet been provided.

---

## Incomplete Union Types

The same concept applies to unions:

```c
union Value;
```

The compiler knows that `union Value` exists but does not yet know its members or size.

It can still be used through a pointer:

```c
union Value *p;
```

---

## Arrays of Unknown Size

An array can also be declared without specifying its size:

```c
extern int numbers[];
```

At this point, the compiler knows that `numbers` is an array of `int`, but its number of elements is not known from this declaration.

This is another form of incomplete type.

The size can be provided later:

```c
int numbers[10];
```

Now the array type is complete.

---

# Use Case: Arrays in Header Files

Incomplete array types are particularly useful in header files.

Suppose you have an array defined in a `.c` file:

```c
int numbers[5] = {1, 2, 3, 4, 5};
```

You want other source files to access it.

Instead of placing the actual definition in the header, you can put an `extern` declaration there.

### Header file

```c
/* numbers.h */

extern int numbers[];
```

This tells other source files:

> An array named `numbers` exists somewhere, but its size is defined elsewhere.

Then the actual definition can be placed in the source file:

```c
/* numbers.c */

#include "numbers.h"

int numbers[5] = {1, 2, 3, 4, 5};
```

The compiler can use the declaration from the header without needing the array's size at that point.

---

# Why Use an Incomplete Array in a Header?

This separates the **declaration** from the **definition**.

```text
numbers.h
    |
    | declaration
    v
extern int numbers[];

    |
    v

numbers.c
    |
    | definition
    v
int numbers[5] = {...};
```

This is useful when multiple source files need access to the same global array.

The header tells them that the array exists, while the `.c` file provides its actual storage and size.

---

# `extern` Does Not Create the Array

This is important.

```c
extern int numbers[];
```

does **not** allocate storage for the array.

It is only a declaration.

The actual definition must exist somewhere:

```c
int numbers[5];
```

The definition is what creates the object and provides its storage.

---

# Completing Incomplete Types

An incomplete type becomes complete when enough information is provided for the compiler to determine its size and layout.

For a structure:

```c
struct Person;
```

The type is incomplete.

Then:

```c
struct Person
{
    int age;
    char name[50];
};
```

Now it is complete.

---

# Completing an Incomplete Array Type

An array declared without a size:

```c
extern int numbers[];
```

can be completed by providing its definition:

```c
int numbers[5] = {1, 2, 3, 4, 5};
```

The compiler now knows:

```text
Element type → int
Number of elements → 5
Total size → 5 × sizeof(int)
```

---

# Incomplete Types and Pointers

Pointers are especially important when working with incomplete types.

Example:

```c
struct Secret;

struct Secret *p;
```

This is valid.

You can pass such a pointer between functions even though the structure's definition is hidden.

For example:

```c
void process(struct Secret *p);
```

The function does not necessarily need to know the internal structure of `struct Secret`.

This idea is heavily used in larger C programs to **hide implementation details**.

---

# Reverse Engineering Perspective

Incomplete types are useful to understand when reading real-world C code and headers.

You may encounter declarations such as:

```c
struct SomeInternalObject;
```

followed by:

```c
struct SomeInternalObject *obj;
```

without seeing the structure's members.

This means the implementation is intentionally hiding the object's internal layout.

In RE, this concept appears frequently when:

- Reading C headers
    
- Analyzing APIs
    
- Working with opaque handles
    
- Reconstructing structures from binaries
    
- Understanding data structures whose definitions are hidden
    
- Analyzing libraries where internal structures are not exposed
    

For example:

```c
struct Connection;

struct Connection *create_connection(void);
void destroy_connection(struct Connection *c);
```

The user of the API can work with a `struct Connection *` without knowing what is inside the structure.

The implementation can keep the actual definition private:

```c
struct Connection
{
    int socket;
    int state;
    void *data;
};
```

This is commonly called an **opaque type** or **opaque structure**.

---

# Mental Model

```text
Incomplete Type
      │
      │ Compiler knows the type exists
      │ but not its complete layout
      ▼
Can use pointers
      │
      │
      ├── struct Node *
      └── union Value *
      
Cannot generally:
      │
      ├── Create an object
      ├── Access its members
      └── Determine its size with sizeof
       
      │
      ▼
Complete the type
      │
      ▼
Compiler knows size + layout
```

---

# Long-Term Takeaways

- An **incomplete type** is a type whose complete size or layout is not yet known.
    
- A structure can be forward-declared:
    

```c
struct Node;
```

- Pointers to incomplete types are allowed:
    

```c
struct Node *p;
```

- You cannot normally create an object of an incomplete type because its size is unknown.
    
- Self-referential structures use pointers to their own type:
    

```c
struct Node
{
    int value;
    struct Node *next;
};
```

- A structure cannot contain an instance of itself directly.
    
- `union` types can also be incomplete.
    
- Arrays declared without a known size are incomplete:
    

```c
extern int numbers[];
```

- Incomplete arrays are useful in header files when the actual array definition is kept in a `.c` file.
    
- `extern` declares an object but does not allocate its storage.
    
- A structure becomes complete when its member definition is provided.
    
- Incomplete types are useful for hiding implementation details through **opaque structures**.
    
- In reverse engineering, incomplete/opaque types are important when structure definitions are hidden and must potentially be reconstructed from program behavior and binary data.