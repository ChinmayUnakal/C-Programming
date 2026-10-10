# Multithreading

A **thread** is an independent path of execution within a program.

Normally, a program may execute on one CPU core, but its work can be divided among multiple threads so several parts can run concurrently. The OS may schedule those threads across available CPU cores.

The `main()` function itself runs inside a thread.

---

# What Threads Can Do

A C thread can:

- Be created to execute a specified function.
    
- Run while the creating thread continues its own work.
    
- Be **joined** so another thread waits for it to finish.
    
- Be **detached** when no result or explicit waiting is needed.
    
- End by calling a thread-exit mechanism or by returning from its thread function.
    
- Sleep for a period while other threads execute.
    

---

# Data Races and the Standard Library

Not every standard-library function is automatically safe for concurrent use.

Some functions, such as:

```c
asctime()
strtok()
```

use or return shared static data, which can create thread-safety problems.

In general, the standard library tries to provide thread-safe behavior, but care is still required.

A warning sign is a function that:

- Keeps state between calls in storage you do not control.
    
- Returns a pointer to internal data instead of caller-provided storage.
    

---

# Creating Threads

Every C thread has an identifier of type:

```c
thrd_t
```

When creating a thread, you provide:

- A pointer to the function it should execute.
    
- A `void *` argument for that function, or `NULL`.
    

The new thread begins execution in that function.

---

# Waiting for a Thread

To wait until a particular thread finishes, use:

```c
thrd_join()
```

The thread ID tells C which thread should be joined.

Joining generally performs the necessary cleanup after a thread completes.

---

# `thrd_start_t`

The function supplied to `thrd_create()` must have the type:

```c
int (*)(void *)
```

In other words, it:

```text
takes → void *
returns → int
```

The function can have any name as long as its signature matches this requirement.

Example:

```c
int run(void *arg)
{
    return 12;
}
```

---

# Basic Thread Workflow

```text
1. Write the thread function
        ↓
2. Create the thread with thrd_create()
        ↓
3. Thread performs its work
        ↓
4. Parent thread can continue independently
        ↓
5. Use thrd_join() when you need the thread to finish
```

---

# `thrd_create()`

Basic form:

```c
thrd_create(&t, run, arg);
```

Where:

```text
&t  → receives the new thread ID
run → function executed by the thread
arg → argument passed to run()
```

Example:

```c
#include <stdio.h>
#include <threads.h>

int run(void *arg)
{
    int *a = arg;

    printf("THREAD: Running with arg %d\n", *a);

    return 12;
}

int main(void)
{
    thrd_t t;
    int arg = 3490;

    thrd_create(&t, run, &arg);

    int res;
    thrd_join(t, &res);

    printf("Thread returned %d\n", res);
}
```

The value returned by the thread is received through `thrd_join()`.

---

# Lifetime of Thread Arguments

The object passed through the `void *` argument must remain valid long enough for the new thread to access it.

Also, the parent thread must not modify that object before the new thread has safely copied or consumed its value.

This becomes important when creating many threads.

---

# Passing the Same Variable to Multiple Threads

Consider:

```c
for (i = 0; i < THREAD_COUNT; i++)
    thrd_create(t + i, run, &i);
```

Every thread receives the address of the **same `i`**.

The parent thread changes `i` rapidly while new threads may not yet have read it.

As a result, threads can observe unexpected values, duplicate values, or even the final value after the loop finishes. This is a **race condition**.

Conceptually:

```text
Main:
i = 0 → 1 → 2 → 3 → 4 → 5
       \
        \ same address
         \
Thread A ─┐
Thread B ─┼──> same i
Thread C ─┘
```

The important problem is not the pointer itself; it is that all threads refer to the same changing object.

---

# Giving Each Thread Its Own Argument

Each thread needs a separate object containing its argument.

One approach is dynamic allocation:

```c
int *arg = malloc(sizeof *arg);
*arg = i;

thrd_create(t + i, run, arg);
```

Then the thread can copy the value and release the allocated memory:

```c
int run(void *arg)
{
    int i = *(int *)arg;

    free(arg);

    printf("THREAD %d: running!\n", i);

    return i;
}
```

Each thread now receives a different memory location, so changes to the loop variable do not affect the argument already assigned to that thread.

---

# Detaching Threads

A thread can be detached with:

```c
thrd_detach(t);
```

This is useful when the parent does not need to wait for the thread or obtain its return value.

Example:

```c
thrd_create(&t, run, NULL);
thrd_detach(t);
```

After detaching, the thread can clean up its resources when it terminates without being joined by the parent.

---

# Thread IDs Are Opaque

`thrd_t` is intentionally an opaque type.

Do not assume it is:

```text
int
unsigned long
pointer
```

It could even be a structure.

Therefore, printing a `thrd_t` by guessing its underlying type is not portable.

A thread ID may also be reused after the corresponding thread terminates.

---

# Thread Local Data

Threads have separate call stacks, so ordinary automatic local variables are private to each thread.

However, variables with:

- `static` storage
    
- file scope
    

are shared.

Example:

```c
static int foo = 10;
```

Every thread accessing `foo` refers to the same object.

A local automatic variable such as:

```c
int x = foo;
```

is separate for each thread.

Therefore:

```text
Shared:
static int foo;

Per-thread:
int x;
```

Multiple threads can interfere with each other when modifying shared state.

---

# Race Condition

Suppose:

```c
static int foo = 10;

int x = foo;
```

Then another thread modifies:

```c
foo++;
```

The first thread's `x` remains unchanged, while `foo` changes.

So:

```text
x = 10
foo = 11
```

may occur even though the first thread did not modify `foo` itself.

This unpredictable interaction is an example of a **race condition**.

---

# `_Thread_local`

C provides:

```c
_Thread_local
```

to give each thread its **own instance** of a variable.

The `<threads.h>` header also provides:

```c
thread_local
```

as an easier spelling.

Example:

```c
thread_local static int foo = 10;
```

Now each thread has a separate `foo`.

```text
Thread 1 → foo = 10
Thread 2 → foo = 10
Thread 3 → foo = 10
```

Changes made by one thread do not modify another thread's copy.

For a block-scope `thread_local` variable, `static` is required unless the declaration is `extern`.

---

# Thread-Specific Storage (TSS)

Another way to maintain per-thread data is **thread-specific storage**.

C provides:

```c
tss_t
tss_create()
tss_set()
tss_get()
tss_delete()
```

A TSS object can also have a destructor.

The destructor type is:

```c
void (*)(void *)
```

A common choice is:

```c
free
```

so dynamically allocated per-thread data can be released automatically when the thread exits.

---

# TSS Workflow

```text
tss_create()
      ↓
tss_set()
      ↓
tss_get()
      ↓
Thread exits
      ↓
Destructor runs
      ↓
tss_delete()
```

Important distinction:

```text
Thread exits      → destructors are invoked
tss_delete()      → destroys the TSS object itself
```

`tss_delete()` does not run the thread destructors.

---

# Mutexes

A **mutex** is used to ensure that only one thread at a time enters a protected section of code.

That protected area is the **critical section**.

Main operations:

```c
mtx_init()
mtx_lock()
mtx_unlock()
mtx_destroy()
```

Workflow:

```text
mtx_init()
    ↓
mtx_lock()
    ↓
critical section
    ↓
mtx_unlock()
    ↓
mtx_destroy()
```

A thread that successfully locks the mutex can continue.

A thread that encounters a mutex already owned by another thread waits until it becomes available.

---

# Protecting Shared Data

Without synchronization:

```c
printf("%d\n", serial);
serial++;
```

different threads may execute these operations in an overlapping order.

For example, multiple threads might read the same value before any of them increments it.

That can produce duplicate results.

The solution is to protect the whole read/modify operation:

```c
mtx_lock(&serial_mtx);

printf("%d\n", serial);
serial++;

mtx_unlock(&serial_mtx);
```

The code between the lock and unlock forms the critical section.

---

# Multiple Mutexes

A program may use several mutexes.

When multiple locks are involved, use a consistent locking order.

A key rule from the reference is:

```text
Unlock mutexes in the reverse order in which they were locked.
```

This helps avoid lock-order problems such as deadlock.

---

# Mutex Types

`mtx_init()` supports several mutex configurations:

|Type|Purpose|
|---|---|
|`mtx_plain`|Normal mutex|
|`mtx_timed`|Supports timed locking|
|`mtx_plain \| mtx_recursive`|Recursive mutex|
|`mtx_timed \| mtx_recursive`|Recursive + timed|

A **recursive mutex** allows its owning thread to lock the same mutex multiple times.

It must unlock the mutex the same number of times before another thread can acquire it.

A **timed mutex** allows a thread to stop waiting after a specified deadline.

---

# Timed Mutex Lock

For a timed mutex:

```c
mtx_init(&serial_mtx, mtx_timed);
```

Use:

```c
mtx_timedlock()
```

instead of `mtx_lock()`.

The timeout is an **absolute UTC timestamp**, not a duration.

Example:

```c
struct timespec timeout;

timespec_get(&timeout, TIME_UTC);

timeout.tv_sec += 1;

int result = mtx_timedlock(&serial_mtx, &timeout);

if (result == thrd_timedout)
    printf("Mutex lock timed out!\n");
```

`timespec_get()` is useful for obtaining the current UTC time.

In `struct timespec`:

```text
tv_sec  → seconds since the epoch
tv_nsec → fractional nanoseconds
```

---

# Condition Variables

A **condition variable** allows a thread to sleep until some shared condition becomes true.

It is commonly combined with a mutex.

The condition variable itself does not hold the application's data. It provides a mechanism for threads to wait and be notified.

C uses:

```c
cnd_t
cnd_init()
cnd_wait()
cnd_signal()
cnd_broadcast()
cnd_destroy()
```

---

# Producer-Consumer Example

A common pattern is:

```text
Main thread
    ↓
Produces data
    ↓
Shared array
    ↓
Child thread
    ↓
Consumes/processes data
```

The shared data must be protected by a mutex.

The child thread can sleep instead of repeatedly polling the array.

Once the required amount of data is available, the main thread signals the child.

---

# `cnd_wait()`

A key property of:

```c
cnd_wait(&condition, &mutex);
```

is that the waiting thread:

1. Releases the mutex.
    
2. Goes to sleep.
    
3. Wakes when notified.
    
4. Reacquires the mutex before continuing.
    

This allows another thread to acquire the same mutex and update the shared state.

---

# Always Check the Condition in a `while` Loop

Use:

```c
while (condition_not_met)
    cnd_wait(&condition, &mutex);
```

rather than:

```c
if (condition_not_met)
    cnd_wait(&condition, &mutex);
```

The reason is **spurious wakeup**: a thread may wake without the desired condition actually being satisfied.

Therefore, after waking, the condition must be checked again.

---

# Condition Variable Workflow

### Worker thread

```text
1. Lock mutex
2. While condition is false:
       wait on condition variable
3. Perform the required work
4. Unlock mutex
```

### Producing thread

```text
1. Lock mutex
2. Add data to shared storage
3. If condition is satisfied:
       signal waiting thread
4. Unlock mutex
```

---

# `cnd_signal()`

```c
cnd_signal(&condition);
```

wakes **one** waiting thread.

It does not itself release the mutex.

The signaling thread still needs to unlock the mutex afterward.

---

# `cnd_broadcast()`

```c
cnd_broadcast(&condition);
```

wakes **all threads** currently waiting on that condition variable.

This is useful when several waiting threads may need to reconsider the shared condition. Only one can hold the mutex at a time, however.

---

# Timed Condition Wait

C also provides:

```c
cnd_timedwait()
```

It works like `cnd_wait()` but includes an absolute UTC deadline.

Example:

```c
struct timespec timeout;

timespec_get(&timeout, TIME_UTC);

timeout.tv_sec += 1;

int result =
    cnd_timedwait(&condition, &mutex, &timeout);

if (result == thrd_timedout)
    printf("Condition variable timed out!\n");
```

A timeout does not necessarily mean the thread runs immediately because it may still need to reacquire the mutex.

---

# Why Condition Variables Matter

Without a condition variable, a waiting thread might repeatedly check a condition:

```text
check
 ↓
not ready
 ↓
check
 ↓
not ready
 ↓
check...
```

This **polling** wastes CPU time.

A condition variable allows the thread to sleep until it has a reason to check again.

---

# Running a Function Only Once

C provides:

```c
call_once()
```

for initialization or other work that must happen only once.

A group of threads can all call it, but the target function executes only once.

You need an:

```c
once_flag
```

initialized with:

```c
once_flag of = ONCE_FLAG_INIT;
```

The function passed to `call_once()` must:

```text
take no arguments
return no value
```

Example:

```c
void run_once_function(void)
{
    printf("I'll only run once!\n");
}

call_once(&of, run_once_function);
```

Even if many threads reach the `call_once()` statement, `run_once_function()` executes only a single time.

---

# Mental Model

```text
                    Multithreading
                         │
        ┌────────────────┼────────────────┐
        │                │                │
     Threads          Shared data      Synchronization
        │                │                │
        │                │        ┌───────┼────────┐
        ▼                ▼        ▼       ▼        ▼
 thrd_create()       static data mutex  condition  once
 thrd_join()                         variable
 thrd_detach()                         │
 thrd_sleep()                   ┌──────┼───────┐
                                 ▼      ▼       ▼
                              wait   signal broadcast
```

---

# Reverse Engineering Perspective

When analyzing multithreaded binaries, look for:

```text
thread creation
thread termination
thread synchronization
shared memory
locking
waiting
thread-local storage
```

Important C APIs from this topic include:

```c
thrd_create()
thrd_join()
thrd_detach()
thrd_sleep()

mtx_init()
mtx_lock()
mtx_unlock()
mtx_destroy()
mtx_timedlock()

cnd_init()
cnd_wait()
cnd_signal()
cnd_broadcast()
cnd_timedwait()

call_once()

tss_create()
tss_get()
tss_set()
tss_delete()
```

Recognizing these APIs can help identify thread creation, synchronization, shared-state protection, waiting mechanisms, and per-thread data inside a binary.

---

# Long-Term Takeaways

- A thread is an independent execution path inside a process.
    
- `main()` itself executes within a thread.
    
- `thrd_t` identifies a thread.
    
- `thrd_create()` starts a new thread.
    
- `thrd_join()` waits for a thread and can retrieve its return value.
    
- `thrd_detach()` lets a thread finish independently without being joined.
    
- Thread arguments must remain valid until the new thread uses them.
    
- Passing the address of one changing variable to multiple threads can create a race condition.
    
- Give each thread its own argument storage when separate values are required.
    
- Automatic local variables belong to each thread's stack, while `static` and file-scope variables are shared.
    
- `_Thread_local` / `thread_local` creates a separate instance for each thread.
    
- TSS provides another mechanism for storing per-thread data and can attach destructors.
    
- A mutex protects a **critical section** so only one thread enters it at a time.
    
- `mtx_lock()` acquires a mutex and `mtx_unlock()` releases it.
    
- Recursive mutexes allow the owning thread to lock the same mutex repeatedly.
    
- Timed mutexes use an absolute UTC deadline.
    
- Condition variables let threads sleep until shared state may have changed.
    
- `cnd_wait()` releases the mutex while waiting and reacquires it before returning.
    
- Always test the condition again after `cnd_wait()` because of spurious wakeups.
    
- `cnd_signal()` wakes one waiter; `cnd_broadcast()` wakes all waiters.
    
- Condition variables avoid inefficient polling.
    
- `call_once()` ensures a function is executed only once.
    
- Thread IDs are opaque and may be reused after a thread terminates.
    
- Multithreading introduces concurrency issues that are important to recognize when analyzing low-level program behavior.