# Atomics

C11 introduced **atomic operations** for safely accessing shared data from multiple threads.

The related functionality is provided by:

```c
#include <stdatomic.h>
```

Atomics guarantee that an atomic access is performed as one indivisible operation. For example, another thread cannot observe only part of a multi-byte atomic write.

Atomics are optional in an implementation.

---

# Testing for Atomic Support

`<stdatomic.h>` support can be checked with:

```c
__STDC_NO_ATOMICS__
```

If atomics are unavailable, this macro has the value `1`.

A common compile-time test is:

```c
#if __STDC_VERSION__ < 201112L || __STDC_NO_ATOMICS__ == 1
#define HAS_ATOMICS 0
#else
#define HAS_ATOMICS 1
#endif
```

This checks both the C language version and whether the implementation disables atomics.

---

# Atomic Variables

An atomic variable can safely be accessed by multiple threads without another thread seeing a partially completed atomic access.

Example:

```c
#include <stdatomic.h>

atomic_int x;
```

`atomic_int` is one of the predefined atomic types.

For example, one thread can do:

```c
x = 3490;
```

while another repeatedly checks:

```c
while (x != 3490) {}
```

The access to `x` is atomic.

This kind of loop is called a **spin-wait** or **spin loop**.

Atomics can replace some mutex-based synchronization, although more advanced lock-free structures are harder to implement correctly.

---

# Synchronization

Atomicity and synchronization are related, but they are not the same thing.

A thread may write:

```c
x = 2;
y = 3;
```

while another thread waits for:

```c
while (y != 3) {}
```

Without proper synchronization, the second thread is not automatically guaranteed to observe `x == 2` at that point.

Why?

The compiler and CPU may reorder memory operations, or the architecture may delay visibility between threads.

Using an atomic variable for synchronization provides an ordering relationship.

Example:

```c
int x = 0;
atomic_int y = 0;

thread1:
    x = 2;
    y = 3;

thread2:
    while (y != 3) {}
    printf("%d", x);
```

After thread 2 successfully observes the atomic value written to `y`, the earlier write to `x` is visible to it.

`x` itself does **not** need to be atomic merely because `y` is being used as the synchronization point.

---

# Acquire and Release

An atomic read can act as an **acquire**.

An atomic write can act as a **release**.

```text
Read / Load  → Acquire
Write / Store → Release
```

If one thread releases an atomic variable and another later acquires that same atomic variable, earlier writes from the releasing thread become visible to the acquiring thread.

---

# Acquire

For an acquire operation on an atomic variable:

- Earlier writes from the thread that released that same atomic become visible.
    
- Operations that occur after the acquire cannot be moved before it by allowed compiler reordering.
    

Think of acquire as a **one-way ordering barrier**.

---

# Release

For a release operation:

- Earlier writes in the current thread become visible to a thread that later acquires the same atomic.
    
- Operations that occurred before the release cannot be moved to after it.
    

So:

```text
Thread 1
writes data
   ↓
release atomic
   ↓
Thread 2
acquire same atomic
   ↓
sees previous writes
```

---

# Important: Synchronization Is Per Atomic Variable

A release on one atomic object does not automatically synchronize with an acquire on a different atomic object.

For example:

```c
atomic_int a;
atomic_int b;
```

Synchronization through `a` is separate from synchronization through `b`.

---

# Sequential Consistency

**Sequential consistency** is C's default memory ordering for ordinary atomic operations.

It provides the strongest and easiest-to-reason-about ordering model among the standard memory orders discussed here.

It prevents the kinds of reorderings allowed by weaker memory-ordering modes.

For normal atomic operations:

```text
Load            → acquire
Store           → release
Read-modify-write → acquire + release
```

Sequential consistency also preserves a single total order for the relevant atomic operations.

Unless you have a specific reason and understand the consequences, sequential consistency is the safer default.

---

# Atomic Assignments and Operators

Not every expression involving an atomic variable is a single atomic operation.

For example:

```c
atomic_int x = 0;

x = x + 3;
```

contains:

```text
read x
calculate x + 3
write x
```

Another thread can act between the read and write, so the entire operation is **not atomic**.

But:

```c
x += 3;
```

is an atomic read-modify-write operation.

The same applies to these operators:

```c
a++
a--
++a
--a

a += b
a -= b
a *= b
a /= b
a %= b

a &= b
a |= b
a ^= b
a >>= b
a <<= b
```

for an atomic `a`.

---

# Library Functions That Synchronize

Some standard functions establish synchronization between threads.

Important examples:

```text
call_once()
thrd_create()
thrd_join()

mtx_lock()
mtx_timedlock()
mtx_trylock()

malloc()
calloc()
realloc()
aligned_alloc()
```

Key relationships:

```text
thrd_create()
→ new thread sees earlier parent writes

thrd_join()
→ joining thread sees writes made by the completed thread

mtx_unlock()
→ later mtx_lock() on the same mutex can see protected writes

successful mtx_timedlock()/mtx_trylock()
→ similar synchronization to mtx_lock()
```

`call_once()` also synchronizes later calls using the same `once_flag`.

Dynamic allocation functions provide synchronization relationships for reuse of the same memory region.

---

# Atomic Type Specifier and Qualifier

C provides predefined atomic types such as:

```c
atomic_int
atomic_uint
atomic_long
atomic_llong
atomic_bool
```

and atomic versions of many standard integer types.

These correspond to forms such as:

```c
_Atomic int
_Atomic unsigned int
_Atomic long
```

The `<stdatomic.h>` aliases also cover types such as:

```text
char
short
long long
char16_t
char32_t
wchar_t
int_leastN_t
uint_leastN_t
int_fastN_t
uint_fastN_t
intptr_t
uintptr_t
size_t
ptrdiff_t
intmax_t
uintmax_t
```

with their atomic equivalents.

---

# `_Atomic` as a Type Specifier

`_Atomic` can be followed by a type in parentheses:

```c
_Atomic(double) f;
```

It can also be used with `typedef`:

```c
typedef _Atomic(double) atomic_double;
```

Then:

```c
atomic_double f;
```

The type being converted to atomic cannot be:

- An array
    
- A function
    
- Already atomic
    
- Otherwise qualified
    

---

# `_Atomic` as a Type Qualifier

`_Atomic` can also appear directly before a type:

```c
_Atomic int i;
```

This is another way to declare an atomic integer.

Other qualifiers can be combined with it:

```c
_Atomic volatile int k;
```

With the qualifier form, arrays and functions still cannot be made atomic.

---

# Lock-Free Atomics

Whether an atomic operation is implemented directly by hardware depends on the platform and data size.

If the hardware cannot perform the operation atomically, the implementation may internally use a lock.

So:

```text
Atomic type
    ↓
Hardware supports it?
   / \
 Yes  No
 ↓     ↓
direct  internal locking
```

Hardware lock-free operations are generally faster and can matter when atomics are used in signal handlers.

`atomic_flag` is the only atomic type that the standard guarantees to be lock-free.

---

# Detecting Lock-Free Types

Compile-time macros exist for several atomic types.

Examples:

```c
ATOMIC_BOOL_LOCK_FREE
ATOMIC_INT_LOCK_FREE
ATOMIC_LONG_LOCK_FREE
ATOMIC_LLONG_LOCK_FREE
ATOMIC_POINTER_LOCK_FREE
```

They can have three values:

|Value|Meaning|
|---|---|
|`0`|Never lock-free|
|`1`|May or may not be lock-free|
|`2`|Always lock-free|

The value `1` means the implementation cannot determine the final answer entirely at compile time; the result may depend on the runtime environment.

At runtime, use:

```c
atomic_is_lock_free(&x);
```

which reports whether the particular atomic object is lock-free.

---

# Signal Handlers and Lock-Free Atomics

Accessing shared objects from a signal handler is restricted.

For a shared variable with static storage duration or `_Thread_local` storage, the permitted portable approaches described in the reference are:

```text
volatile sig_atomic_t
```

or a **lock-free atomic type**.

This makes lock-free atomics particularly relevant when communicating information from a signal handler.

---

# Atomic Flags

`atomic_flag` is a special atomic type designed for **test-and-set** behavior.

It is guaranteed to be lock-free.

Initialization:

```c
atomic_flag f = ATOMIC_FLAG_INIT;
```

Set-and-test:

```c
atomic_flag_test_and_set(&f);
```

This:

- Sets the flag.
    
- Returns its previous state as `_Bool`.
    

Clear it with:

```c
atomic_flag_clear(&f);
```

Example sequence:

```text
Initial → clear

test_and_set() → returns 0, becomes set
test_and_set() → returns 1, remains set
clear()        → becomes clear
test_and_set() → returns 0, becomes set
```

---

# Atomic Structures and Unions

Using `_Atomic`, a structure or union can also be made atomic.

Example:

```c
struct point
{
    float x, y;
};

_Atomic(struct point) p;
```

You can test whether the object is lock-free:

```c
atomic_is_lock_free(&p);
```

Small structures may be lock-free on some implementations.

---

# Accessing Atomic Structures

You cannot directly access a member through an atomic structure:

```c
p.x;       // Error
```

Instead, atomically copy the complete object to a normal structure:

```c
struct point t;

t = p;
```

Then access:

```c
printf("%f\n", t.x);
```

The reverse direction also works:

```c
p = (struct point){1, 2};
```

which atomically stores the complete structure.

Individual structure members can also be declared atomic.

Whether atomic types can be used on bit-fields is implementation-defined.

---

# Atomic Pointers

There are three different cases.

### Pointer to an atomic object

```c
_Atomic int x;
_Atomic int *p;

p = &x;
```

Here:

```text
p itself      → ordinary pointer
*p            → atomic int
```

### Atomic pointer to a normal object

```c
int x;
int * _Atomic p;

p = &x;
```

Here:

```text
p             → atomic pointer
*p            → normal int
```

### Atomic pointer to an atomic object

```c
_Atomic int x;
_Atomic int * _Atomic p;

p = &x;
```

Here both are atomic:

```text
p             → atomic pointer
*p            → atomic int
```

---

# Memory Orders

C defines several memory-order options:

```c
memory_order_seq_cst
memory_order_acq_rel
memory_order_release
memory_order_acquire
memory_order_consume
memory_order_relaxed
```

They control how atomic operations interact with memory visibility and instruction reordering.

---

# Explicit Memory Ordering

An atomic operation can use the default sequentially consistent behavior:

```c
atomic_int x = 0;

x += 5;
```

Or an atomic library function:

```c
atomic_fetch_add(&x, 5);
```

For explicit ordering:

```c
atomic_fetch_add_explicit(
    &x,
    5,
    memory_order_seq_cst
);
```

A weaker ordering can also be selected:

```c
atomic_fetch_add_explicit(
    &x,
    5,
    memory_order_acq_rel
);
```

Weaker memory orders can improve performance, but they require a much better understanding of the memory model.

---

# Memory Order: Acquire

An acquire applies to an atomic load/read.

Properties:

- It can observe writes made before another thread's release of the same atomic.
    
- Operations following the acquire cannot be reordered before it.
    

---

# Memory Order: Release

A release applies to an atomic store/write.

Properties:

- Earlier writes in the current thread become visible to another thread that later acquires the same atomic.
    
- Earlier operations cannot be moved across the release in the wrong direction.
    

---

# Memory Order: Consume

`memory_order_consume` is weaker than acquire.

It primarily affects operations that are **data-dependent** on the loaded atomic value.

Example concept:

```text
load atomic pointer
        ↓
use that pointer
        ↓
dependent memory operations
```

Unlike acquire, it does not impose the same ordering on unrelated operations.

The reference notes that consume is a specialized and unusual ordering mode.

---

# Memory Order: Acquire/Release

`memory_order_acq_rel` applies to **read-modify-write** operations.

The operation combines:

```text
read side  → acquire
write side → release
```

So one operation performs both synchronization roles.

---

# Memory Order: Relaxed

`memory_order_relaxed` imposes the fewest ordering constraints.

The atomic access is still indivisible, but there is no synchronization guarantee comparable to acquire/release.

Operations may be reordered much more freely.

Relaxed operations can be combined with fences when synchronization is required.

---

# Fences

A **fence** provides acquire/release-style ordering without using an atomic variable as the synchronization object.

Example:

```c
atomic_thread_fence(memory_order_release);
```

A release fence in one thread can be paired with an acquire fence in another when the synchronization requirements are satisfied.

`memory_order_consume` is treated as acquire for fences.

---

# `atomic_signal_fence()`

C also provides:

```c
atomic_signal_fence()
```

It is intended for synchronization between normal execution and a signal handler in the **same thread**.

Unlike `atomic_thread_fence()`:

- It does not synchronize with another thread.
    
- It does not emit a hardware memory-fence instruction.
    

It mainly prevents compiler reordering of relevant operations so that changes are visible within the same thread's signal-handling context.

---

# Mental Model

```text
                     ATOMICS
                        │
          ┌─────────────┼─────────────┐
          │             │             │
       Atomicity    Synchronization  Ordering
          │             │             │
          ▼             ▼             ▼
     indivisible     acquire       seq_cst
       access        release       acq_rel
                                    acquire
                                    release
                                    consume
                                    relaxed
```

```text
Thread 1                           Thread 2

write shared data
       ↓
release atomic
       ↓
                               acquire same atomic
                                      ↓
                              sees earlier writes
```

---

# Reverse Engineering Perspective

Atomics are important when reversing multithreaded programs because they can reveal:

- Shared counters
    
- Synchronization flags
    
- Lock-free data structures
    
- Thread communication
    
- Reference counters
    
- State transitions
    
- Memory-ordering barriers
    

When examining assembly, an atomic operation may appear as a normal load/store or as a specialized read-modify-write instruction depending on the compiler, architecture, and chosen memory order.

A C statement such as:

```c
x += 1;
```

on an atomic object may compile into an instruction sequence that guarantees an indivisible update rather than an ordinary load/add/store sequence.

This distinction is important when reconstructing synchronization logic from a binary.

---

# Long-Term Takeaways

- Atomics provide indivisible accesses to shared objects.
    
- Atomic support is optional and can be checked with `__STDC_NO_ATOMICS__`.
    
- `<stdatomic.h>` provides predefined atomic types such as `atomic_int`.
    
- Atomicity does not automatically mean that every other memory access is synchronized.
    
- Atomic operations can establish **happens-before-style visibility** between threads through acquire/release relationships.
    
- An acquire reads an atomic; a release writes an atomic.
    
- Synchronization through one atomic object does not automatically apply to another atomic object.
    
- Sequential consistency is the default and easiest memory-ordering model.
    
- `x = x + 3` is not one atomic read-modify-write operation merely because `x` is atomic.
    
- Operators such as `x += 3`, `x++`, `x--`, and the listed bitwise/shift assignments are atomic read-modify-write operations.
    
- `_Atomic` can be used as either a type specifier or type qualifier.
    
- Arrays and functions cannot be made atomic.
    
- Hardware may implement an atomic operation using an internal lock when direct lock-free support is unavailable.
    
- `atomic_flag` is guaranteed to be lock-free.
    
- `atomic_is_lock_free()` checks lock-free status at runtime.
    
- Atomic structures and unions can be used, but their members cannot be accessed directly through the atomic object.
    
- Pointers and pointed-to objects can independently be atomic or non-atomic.
    
- Available memory orders are:  
    `seq_cst`, `acq_rel`, `release`, `acquire`, `consume`, and `relaxed`.
    
- Acquire prevents later operations from moving before it.
    
- Release prevents earlier operations from moving after it.
    
- Consume mainly orders operations that depend on the loaded value.
    
- `acq_rel` combines acquire and release on a read-modify-write operation.
    
- Relaxed operations preserve atomicity but provide minimal ordering and synchronization.
    
- `atomic_thread_fence()` provides ordering without requiring an atomic object as the fence itself.
    
- `atomic_signal_fence()` is limited to compiler ordering within the same thread and does not create inter-thread synchronization.
    
- In RE, atomics are useful clues for identifying **shared state, synchronization, lock-free algorithms, and thread communication**.