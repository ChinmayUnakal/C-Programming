
Dynamic memory allocation allows a program to **request and release memory at runtime**.

The main functions are:

```c
malloc()
calloc()
realloc()
free()
```

Dynamically allocated memory is commonly associated with the **heap**.

---

## `malloc()`

`malloc()` allocates a specified number of **bytes** and returns a pointer to the allocated memory.

```c
int *p = malloc(sizeof *p);

*p = 12;

printf("%d\n", *p);

free(p);
```

Conceptually:

```text
malloc()
   ↓
Heap allocation
   ↓
p ───────→ [ 12 ]
```

### Important

`malloc()` does **not initialize** the allocated memory.

```c
int *p = malloc(10 * sizeof *p);
```

allocates enough space for 10 `int`s.

---

## `sizeof *p`

Instead of:

```c
int *p = malloc(sizeof(int));
```

prefer:

```c
int *p = malloc(sizeof *p);
```

Because:

```text
*p        → int
sizeof *p → sizeof(int)
```

For an array:

```c
int *p = malloc(10 * sizeof *p);
```

---

## Checking `malloc()`

`malloc()` returns `NULL` if the allocation fails.

```c
int *p = malloc(10 * sizeof *p);

if (p == NULL) {
    printf("Allocation failed\n");
    return 1;
}
```

Always handle allocation failure when appropriate.

---

## Allocating an Array

Dynamic memory can be accessed using array notation:

```c
int *p = malloc(10 * sizeof *p);

if (p == NULL)
    return 1;

for (int i = 0; i < 10; i++)
    p[i] = i * 5;

free(p);
```

Conceptually:

```text
p
↓
[ 0 ][ 5 ][ 10 ][ 15 ][ ... ][ 45 ]
```

The memory is contiguous.

---

# `free()`

`free()` releases dynamically allocated memory.

```c
int *p = malloc(sizeof *p);

/* use p */

free(p);
```

After `free(p)`, the allocated object no longer exists.

---

## Use-After-Free

Accessing an object after it has been freed is **undefined behavior**.

```c
free(p);

*p = 3490;    // WRONG
```

A useful defensive pattern is:

```c
free(p);
p = NULL;
```

This prevents that pointer from continuing to contain the old address.

---

## Memory Leak

A memory leak occurs when allocated memory is no longer reachable and therefore cannot be freed.

```c
int *p = malloc(100 * sizeof *p);

/* p is lost */
```

```text
malloc()
   ↓
allocated memory
   ↓
pointer lost
   ↓
memory cannot be freed
```

---

# `calloc()`

`calloc()` allocates space for multiple elements.

```c
int *p = calloc(10, sizeof *p);
```

Syntax:

```c
calloc(number_of_elements, size_of_each_element)
```

The allocated bytes are initialized to zero.

Conceptually:

```text
calloc(4, sizeof(int))

[ 0 ][ 0 ][ 0 ][ 0 ]
```

---

## `malloc()` vs `calloc()`

|Function|Arguments|Initial state|
|---|---|---|
|`malloc()`|total bytes|Uninitialized|
|`calloc()`|element count + element size|Zero-initialized|

Example:

```c
malloc(10 * sizeof(int));
```

vs.

```c
calloc(10, sizeof(int));
```

---

# `realloc()`

`realloc()` changes the size of an existing allocation.

```c
int *p = malloc(20 * sizeof *p);

int *new_p = realloc(p, 40 * sizeof *p);
```

It can:

- Grow the allocation
    
- Shrink the allocation
    
- Move the allocation to another address
    

The existing contents are preserved up to the limits of the new allocation.

---

## `realloc()` Takes Bytes

The size argument is always a **number of bytes**.

Wrong:

```c
realloc(p, num_elements);
```

Correct:

```c
realloc(p, num_elements * sizeof *p);
```

---

## Safe `realloc()` Pattern

Prefer:

```c
int *new_p = realloc(p, new_size);

if (new_p != NULL)
    p = new_p;
```

Instead of:

```c
p = realloc(p, new_size);
```

### Why?

If `realloc()` fails:

```text
realloc()
   ↓
NULL
```

the original allocation is still valid.

If you directly overwrite `p`, you lose the pointer to that allocation and may create a memory leak.

---

## `realloc()` Can Move Memory

`realloc()` may resize the allocation in place:

```text
Before:

p → [ allocation ]

After:

p → [ larger allocation ]
```

Or move it:

```text
Before:

p → [ OLD ]

After:

p ─────────→ [ NEW ]
```

Therefore:

> **Never assume that `realloc()` preserves the pointer address.**

Any other pointers pointing into the old allocation may become invalid if the allocation moves.

---

# Growing Dynamic Arrays

A common strategy is to increase capacity exponentially:

```text
4 → 8 → 16 → 32 → 64 → ...
```

Example:

```c
capacity *= 2;

int *new_p = realloc(p, capacity * sizeof *p);

if (new_p != NULL)
    p = new_p;
```

Doubling reduces the number of reallocations needed when repeatedly adding elements.

---

# Reading Arbitrarily Long Input

Dynamic memory is useful when the input size is unknown.

Basic strategy:

```text
Allocate initial buffer
        ↓
    Read data
        ↓
   Buffer full?
   ┌────┴────┐
  No        Yes
   ↓          ↓
Continue    realloc()
              ↓
         Larger buffer
              ↓
           Continue
```

For example:

```text
Initial capacity: 4

"abc"
 ↓
buffer still has space

"abcdef"
 ↓
buffer full
 ↓
realloc → 8 bytes

more input...
 ↓
realloc → 16 bytes
```

The buffer can optionally be shrunk after the input is complete.

---

# NUL Terminator

When dynamically constructing a C string, remember the terminating:

```c
'\0'
```

For:

```text
"Hello"
```

the memory contains:

```text
H  e  l  l  o  \0
```

Therefore, it requires:

```text
5 + 1 = 6 bytes
```

The extra byte is required for the NUL terminator.

---

# `realloc()` with `NULL`

These are equivalent:

```c
malloc(3490);
```

and:

```c
realloc(NULL, 3490);
```

This allows code to start with:

```c
int *p = NULL;
```

and later use:

```c
p = realloc(p, new_size);
```

However, the same `realloc()` failure precautions still apply.

---

# Ownership of Dynamic Memory

If a function returns dynamically allocated memory:

```c
char *create_buffer(void)
{
    char *p = malloc(100);

    return p;
}
```

the caller can become responsible for freeing it:

```c
char *p = create_buffer();

if (p != NULL) {
    /* use p */
    free(p);
}
```

### Important principle

> **Know who owns dynamically allocated memory and who is responsible for freeing it.**

---

# Aligned Allocation

Sometimes memory must start at a particular alignment boundary.

```c
char *p = aligned_alloc(64, 256);
```

This requests:

```text
256 bytes
aligned to a 64-byte boundary
```

Example:

```text
Address
   ↓
0x1200
   ↑
multiple of 64
```

`aligned_alloc()` is useful in some low-level, SIMD, hardware, and performance-sensitive code.

The allocated memory can still be released with:

```c
free(p);
```

> `aligned_alloc()` has additional requirements; the requested size must be a multiple of the alignment.

---

# Simplified Process Memory Layout

A simplified view:

```text
High Addresses
┌──────────────────┐
│      Stack       │
├──────────────────┤
│                  │
│                  │
├──────────────────┤
│       Heap       │ ← malloc/calloc/realloc
├──────────────────┤
│ Global / Static  │
├──────────────────┤
│       Code       │
└──────────────────┘
Low Addresses
```

This is a conceptual model. Actual process layouts depend on the OS, executable format, ABI, loader, ASLR, and other factors.

---

# How Does It Appear in Reverse Engineering? — x86-64 Assembly

Dynamic memory allocation is **very important in reverse engineering** because it helps identify:

- Heap objects
    
- Dynamic arrays
    
- Buffers
    
- Object creation/destruction
    
- Object lifetime
    
- Heap-related vulnerabilities
    

---

## `malloc()` in Assembly

C:

```c
int *p = malloc(sizeof *p);
```

You may see:

```asm
mov edi, 4
call malloc
```

On the common System V AMD64 ABI:

```text
RDI → first integer/pointer argument
RAX → return value
```

So after:

```asm
call malloc
```

the returned heap address is typically in:

```asm
RAX
```

Conceptually:

```text
malloc(4)
   ↓
RAX = heap address
```

---

## Storing Through the Returned Pointer

C:

```c
*p = 12;
```

might become:

```asm
mov DWORD PTR [rax], 12
```

Meaning:

```text
RAX
 ↓
[ 12 ]
```

This tells you that the value returned from `malloc()` is being treated as a pointer to a 4-byte object.

---

## Loading Through the Pointer

C:

```c
int x = *p;
```

might become:

```asm
mov eax, DWORD PTR [rax]
```

Conceptually:

```text
RAX → [ int ]
       ↑
       read
```

---

# `free()` in Assembly

C:

```c
free(p);
```

may appear as:

```asm
mov rdi, rax
call free
```

On System V AMD64:

```text
RDI = argument to free()
```

So:

```text
RDI → heap address
        ↓
      free()
```

The exact instructions vary with compiler and optimization level.

---

# Recognizing Heap Allocation

A common pattern is:

```asm
mov edi, 0x20
call malloc
```

Think:

```text
malloc(0x20)
       ↓
32-byte heap allocation
       ↓
RAX = pointer
```

If the returned pointer is then used repeatedly:

```asm
mov [rax], ...
mov [rax+8], ...
mov [rax+0x10], ...
```

you may be looking at a **heap-allocated structure/object**.

---

# Structure Reconstruction

Suppose C contains:

```c
struct user {
    int id;
    int age;
    char *name;
};
```

A compiler might access the members approximately as:

```asm
mov DWORD PTR [rax], ...
mov DWORD PTR [rax+4], ...
mov rcx, QWORD PTR [rax+8]
```

You can infer something like:

```text
rax + 0x00 → id
rax + 0x04 → age
rax + 0x08 → name pointer
```

This is one of the most important techniques in reverse engineering:

> **Memory offsets can reveal the layout of structures and objects.**

---

# Dynamic Arrays in Assembly

C:

```c
int *p = malloc(10 * sizeof *p);

p[i] = 123;
```

If `int` is 4 bytes, the compiler may generate something resembling:

```asm
mov DWORD PTR [rax + rcx*4], 123
```

Where:

```text
RAX = base address
RCX = index
4   = sizeof(int)
```

Therefore:

```text
Address = Base + Index × Element Size
```

This is the same pointer arithmetic you learned earlier.

---

# Very Important RE Pattern

When you see:

```asm
[base + index*4]
```

think:

```c
base[index]
```

for a 4-byte element.

Common scale factors on x86-64:

```text
*1 → 1 byte
*2 → 2 bytes
*4 → 4 bytes
*8 → 8 bytes
```

For example:

```asm
mov eax, [rbx + rcx*4]
```

strongly suggests something like:

```c
int value = array[index];
```

---

# Heap Object Lifetime in RE

You can often trace:

```text
malloc
  ↓
initialization
  ↓
object used
  ↓
object modified
  ↓
free
```

Example:

```asm
call malloc
        ↓
mov [rax], ...
        ↓
mov [rax+8], ...
        ↓
...
        ↓
mov rdi, rax
call free
```

This can reveal the lifetime of a heap object.

---

# `calloc()` in Assembly

C:

```c
int *p = calloc(10, sizeof *p);
```

You may see:

```asm
mov esi, 4
mov edi, 10
call calloc
```

Conceptually:

```text
EDI → number of elements
ESI → size of each element
RAX → returned pointer
```

Again, exact instructions depend on the ABI and compiler.

---

# `realloc()` in Assembly

C:

```c
p = realloc(p, new_size);
```

On System V AMD64, conceptually:

```text
RDI → old pointer
RSI → new size
RAX ← new pointer
```

You may see:

```asm
mov rdi, rbx
mov rsi, rdx
call realloc
```

After the call:

```text
RAX = new allocation address
```

The returned address may differ from the original.

---

# `memcpy()` + Heap Allocation

A common pattern is:

```c
p = malloc(size);
memcpy(p, source, size);
```

Conceptually:

```text
malloc
  ↓
heap buffer
  ↓
memcpy
  ↓
data copied into buffer
```

In RE, this can indicate:

- Buffer creation
    
- Object initialization
    
- Data duplication
    
- Parsing
    
- Serialization/deserialization
    

---

# Heap Bugs in Reverse Engineering

Dynamic allocation is especially important when analyzing vulnerabilities.

### Use-After-Free

```text
malloc
  ↓
use
  ↓
free
  ↓
use again  ← BUG
```

### Double-Free

```text
malloc
  ↓
free
  ↓
free again ← BUG
```

### Heap Overflow

```text
[ Buffer ][ Adjacent Object ]
     ↑
  too much data
     ↓
[ Buffer ][ CORRUPTED DATA ]
```

### Memory Leak

```text
malloc
  ↓
pointer lost
  ↓
memory remains allocated
```

These are fundamental concepts in exploit development and binary vulnerability research.

---

# Reverse Engineering Mental Model

When you encounter:

```asm
call malloc
```

ask:

```text
1. How many bytes are allocated?
2. Where is the returned pointer stored?
3. What offsets are accessed from that pointer?
4. What types might those offsets represent?
5. Is the memory treated as an array or structure?
6. Is the allocation resized with realloc()?
7. Where is it eventually freed?
```

This gives you a way to reconstruct the object's **size, layout, usage, and lifetime**.

---

# C → x86-64 Mental Mapping

|C|Common x86-64 idea|
|---|---|
|`malloc(size)`|`call malloc`|
|Returned pointer|`RAX`|
|`free(p)`|pointer passed to `free`|
|`p[i]`|`[base + index × element_size]`|
|`*p`|`[address]`|
|`struct->member`|`[base + member_offset]`|
|`calloc(n, size)`|`call calloc`|
|`realloc(p, size)`|`call realloc`|
|Heap object|Dynamically allocated memory|

> **The exact registers depend on the calling convention.** The table above primarily describes the System V AMD64 ABI commonly encountered on Linux.

---

# Long-Term Takeaways

- `malloc()` → allocate raw memory.
    
- `calloc()` → allocate and zero-initialize.
    
- `realloc()` → resize an existing allocation.
    
- `free()` → release an allocation.
    
- Allocation sizes are specified in **bytes**.
    
- `sizeof *p` is a useful allocation idiom.
    
- Always consider allocation failure.
    
- Never access memory after `free()`.
    
- Never free the same allocation twice.
    
- Avoid memory leaks.
    
- `realloc()` may move the allocation.
    
- Use a temporary pointer when handling `realloc()` failure.
    
- Dynamic arrays commonly grow using `realloc()`.
    
- C strings require an extra byte for `'\0'`.
    
- `realloc(NULL, size)` behaves like `malloc(size)`.
    
- `aligned_alloc()` provides explicitly aligned memory.
    
- Heap allocations commonly appear around calls to `malloc`, `calloc`, `realloc`, and `free`.
    
- In x86-64 RE, **heap addresses + offsets reveal object layouts**.
    
- `[base + index × scale]` is a major clue for array access.
    
- `[base + constant]` is often a clue for a structure/object member.
    
- Tracking `malloc → use → realloc → free` helps reconstruct **object lifetime**.
    
- Heap bugs are fundamental to binary exploitation and vulnerability research.
    

---

# Core Mental Model

```text
                    DYNAMIC MEMORY

                       malloc()
                          │
                          ▼
                    ┌───────────┐
                    │   HEAP    │
                    │           │
                    │  object   │
                    └───────────┘
                          ▲
                          │
                       pointer
                          │
                     program uses
                          │
             ┌────────────┴────────────┐
             │                         │
          realloc()                 free()
             │                         │
       resize / move              released
             │
             ▼
       new heap address
```

### The most important RE connection

```text
C:

p[i]

        ↓ compiler

Address = p + i × sizeof(element)

        ↓ x86-64

[base + index × scale]
```

For example:

```asm
mov eax, [rbx + rcx*4]
```

Think:

```c
value = ((int *)rbx)[rcx];
```
