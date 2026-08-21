# Atomic Classes

## Core Idea

Atomic classes provide thread-safe atomic operations on simple shared state without requiring a traditional lock for each operation.

A common problem is:

```java
volatile int count = 0;
count++;
```

`volatile` provides visibility, but `count++` is still a compound read-modify-write operation and is not atomic.

For a counter, we can instead use:

```java
AtomicInteger count = new AtomicInteger(0);
count.incrementAndGet();
```

The increment is atomic, so concurrent increments do not lose updates in the way an unsynchronized `count++` can.

## `AtomicInteger`

Example:

```java
class Counter {
    private final AtomicInteger count = new AtomicInteger(0);

    void increment() {
        count.incrementAndGet();
    }

    int getCount() {
        return count.get();
    }
}
```

Useful operations include:

```java
count.get();
count.set(20);
count.incrementAndGet();
count.getAndIncrement();
count.decrementAndGet();
count.getAndDecrement();
count.addAndGet(5);
count.getAndAdd(5);
```

The important distinction:

```text
incrementAndGet()
    10 → 11
    returns 11

getAndIncrement()
    10 → 11
    returns 10
```

## CAS — Compare-And-Swap

Atomic classes commonly rely on **Compare-And-Swap (CAS)** for atomic updates.

Conceptually, CAS says:

> "If the value is still what I expected, change it to the new value. Otherwise, don't overwrite it."

For example:

```text
Current value = 10
Expected value = 10
New value      = 11
```

If another thread has already changed the value:

```text
Expected = 10
Actual   = 12
```

the attempted update fails rather than blindly overwriting the newer value, and the operation can retry.

Mental model:

```text
        Is value still what I expected?
                    │
             ┌──────┴──────┐
            YES            NO
             │              │
             ↓              ↓
       update it         retry
```

For SDE2 preparation, the conceptual idea of CAS is sufficient; CPU-level and lock-free algorithm internals are not required here.

## `volatile` vs Atomic Classes

```text
volatile
   ↓
visibility
   ✗
compound-operation atomicity

AtomicInteger
   ↓
atomic operations
   +
visibility guarantees
```

Therefore:

```java
volatile int count;
count++;                 // not safe as an atomic increment
```

versus:

```java
AtomicInteger count;
count.incrementAndGet(); // atomic increment
```

## Atomic Classes vs `synchronized`

Both can solve the simple counter problem, but they are useful in different situations.

### `synchronized`

```java
synchronized void increment() {
    count++;
}
```

Uses a lock and is useful when multiple operations or a larger critical section must be protected together.

### `AtomicInteger`

```java
count.incrementAndGet();
```

Useful when the shared state is simple and the required operation is one of the atomic operations provided by the atomic class.

Mental model:

```text
Simple shared value
       │
       ├── Need a simple atomic update?
       │          ↓
       │     AtomicInteger
       │
       └── Need to protect a larger
           critical section?
                  ↓
             synchronized
```

## Other Common Atomic Classes

The same general idea exists for other types, such as:

```java
AtomicLong
AtomicReference<T>
```

For the current SDE2 preparation, understanding `AtomicInteger` and the basic CAS concept is more important than memorizing every atomic class and method.

## Common Use Cases

Atomic classes are particularly useful for simple shared state such as:

- Request counters
- Retry counters
- Sequence numbers
- Statistics
- Simple shared references/state updates

Example:

```java
private final AtomicInteger requestCount = new AtomicInteger();

void recordRequest() {
    requestCount.incrementAndGet();
}
```

## Interview-Level Takeaways

- Atomic classes provide thread-safe atomic operations for common shared-state use cases.
- `AtomicInteger` is a common choice for atomic counters.
- `incrementAndGet()` returns the new value; `getAndIncrement()` returns the old value.
- `volatile` does not make `count++` atomic.
- CAS means "update this value only if it is still what I expected"; failed attempts can retry.
- Use atomic classes for simple atomic updates; use `synchronized` when a larger critical section or multiple operations must be coordinated together.
