# OutOfMemoryError & StackOverflowError

Both are subclasses of `Error`, not normal application `Exception` types.

```text
Throwable
   └── Error
        ├── OutOfMemoryError
        └── StackOverflowError
```

## OutOfMemoryError

`OutOfMemoryError` occurs when the JVM cannot satisfy a memory allocation request.

The classic case is heap exhaustion:

```java
List<byte[]> list = new ArrayList<>();

while (true) {
    list.add(new byte[1024 * 1024]);
}
```

The list keeps references to all allocated arrays, so the objects remain reachable and the garbage collector cannot reclaim them. Eventually the JVM cannot satisfy another allocation and throws `OutOfMemoryError`.

### Why GC Cannot Save Us

Garbage collection only reclaims objects that are no longer reachable. If application code retains references to objects it no longer logically needs, those objects remain eligible for neither collection nor reclamation.

Conceptually:

```text
list
 ↓
byte[]   ← reachable
byte[]   ← reachable
byte[]   ← reachable
 ↓
 ...
```

This is why Java applications can still have memory leaks despite having a garbage collector.

### OOM Is Not Necessarily "Heap Is Literally Full"

The technically useful definition is:

> The JVM cannot satisfy a requested memory allocation.

Heap exhaustion is the common case, but OOM can also arise from other JVM/native resource allocation failures. Examples of messages include:

```text
OutOfMemoryError: Java heap space
OutOfMemoryError: Metaspace
OutOfMemoryError: unable to create native thread
```

For SDE2-level fundamentals, understand the allocation/resource distinction without going deep into JVM tuning.

### Can a Single Allocation Cause OOM?

Yes. A single very large allocation can fail even though the application has some free memory overall. What matters is whether the JVM can satisfy the requested allocation.

### Should We Catch It?

It is technically possible:

```java
try {
    // ...
} catch (OutOfMemoryError e) {
    // ...
}
```

But normal application code should not treat `OutOfMemoryError` as a recoverable business exception. It indicates severe memory/resource pressure and generally requires diagnosis rather than normal exception recovery.

## StackOverflowError

Every Java thread has a stack used to maintain method invocation state. Each method call requires a stack frame.

For example:

```text
Thread Stack
┌──────────────┐
│ c() frame    │
├──────────────┤
│ b() frame    │
├──────────────┤
│ a() frame    │
├──────────────┤
│ main() frame │
└──────────────┘
```

A `StackOverflowError` occurs when a thread's stack cannot accommodate additional stack frames.

### Classic Cause: Infinite Recursion

```java
static void recurse() {
    recurse();
}

public static void main(String[] args) {
    recurse();
}
```

The call chain becomes:

```text
main()
  ↓
recurse()
  ↓
recurse()
  ↓
recurse()
  ↓
...
```

Every invocation adds another stack frame. Eventually the thread's stack cannot grow enough and the JVM throws `StackOverflowError`.

### Recursion Itself Is Not the Problem

This is valid:

```java
int factorial(int n) {
    if (n == 0) {
        return 1;
    }

    return n * factorial(n - 1);
}
```

The problem is excessive stack depth, not recursion itself. A bounded recursive algorithm can complete normally.

### Does It Require Recursion?

No. Recursion is simply the most common cause. An excessively deep chain of ordinary method calls can also exhaust the stack.

For interview purposes:

> `StackOverflowError` → think excessive call depth, most commonly infinite/deep recursion.

## OOM vs StackOverflowError

| | `OutOfMemoryError` | `StackOverflowError` |
|---|---|---|
| Typical problem | Memory/resource allocation failure | Thread stack exhaustion |
| Common cause | Too many retained objects / huge allocation | Excessive recursion / call depth |
| Typical memory involved | Often heap | Thread stack |
| Classic symptom | Allocation fails | Method calls keep nesting |
| GC relevance | Can be involved in heap pressure, but cannot reclaim reachable objects | Not a garbage-collection problem |

The key mental model is:

```text
JVM
 │
 ├── Heap
 │     └── object allocation
 │             ↓
 │       OutOfMemoryError
 │
 └── Thread Stack
       └── stack frames
               ↓
         StackOverflowError
```

## Interview Answer

If asked for the difference:

> `OutOfMemoryError` occurs when the JVM cannot satisfy a memory allocation request, commonly because the heap or another JVM/native memory resource is exhausted. `StackOverflowError` occurs when a thread's stack cannot accommodate additional stack frames, most commonly because of excessive or infinite recursion.

## Key Takeaways

- Both are `Error`s, not normal `Exception` types.
- `OutOfMemoryError` is fundamentally an allocation/resource exhaustion problem.
- Heap exhaustion is the classic OOM scenario, but OOM is not limited to the heap.
- GC cannot reclaim objects that are still reachable.
- Java applications can therefore still suffer from memory leaks.
- `StackOverflowError` is fundamentally a thread-stack/call-depth problem.
- Infinite or excessively deep recursion is the classic cause.
- Recursion itself is not inherently wrong; bounded recursion can be perfectly valid.
- Neither error should normally be treated as a recoverable business exception.
