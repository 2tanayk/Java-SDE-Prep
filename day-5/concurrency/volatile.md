# `volatile`

## Core Idea

`volatile` is primarily about **visibility** between threads.

Suppose one thread changes a shared flag:

```java
class Worker {
    private boolean running = true;

    void run() {
        while (running) {
            // do work
        }
    }

    void stop() {
        running = false;
    }
}
```

Without a suitable synchronization mechanism, we do not have the required cross-thread visibility guarantee for Thread A to observe Thread B's update to `running`.

Declaring it volatile:

```java
private volatile boolean running = true;
```

provides the visibility semantics needed for another thread accessing that variable to observe the update.

Mental model:

```text
Thread B                    Thread A

running = false
      │
      ↓
 update becomes visible
      │
      └────────────────────→ reads false
                              ↓
                           exits loop
```

## What `volatile` Gives Us

The key interview-level idea is:

> **`volatile` provides visibility guarantees for accesses to the variable.**

It is useful when multiple threads access a shared variable and the main problem is ensuring that updates made by one thread are visible to other threads.

A classic example is a shared stop/shutdown flag:

```java
private volatile boolean shutdownRequested;
```

One thread can do:

```java
shutdownRequested = true;
```

while another checks:

```java
if (shutdownRequested) {
    shutdown();
}
```

## `volatile` Does NOT Make Compound Operations Atomic

This is the most important interview trap.

Consider:

```java
class Counter {
    volatile int count = 0;

    void increment() {
        count++;
    }
}
```

`volatile` does **not** make `count++` thread-safe.

`count++` is still conceptually:

```text
read count
    ↓
add 1
    ↓
write count
```

Two threads can still interleave:

```text
Thread A          Thread B

read 0
                  read 0

add 1
                  add 1

write 1
                  write 1
```

Final result can still be `1` instead of `2`.

So:

```text
volatile
   ↓
visibility
   ✗
mutual exclusion / atomicity for compound operations
```

For such cases, depending on the problem, mechanisms such as `synchronized` or atomic classes are appropriate.

## `volatile` vs `synchronized`

| | `volatile` | `synchronized` |
|---|---|---|
| Visibility | Yes | Yes |
| Mutual exclusion | No | Yes |
| Makes `count++` atomic? | No | Yes, when the operation is protected by the same lock |
| Uses a lock? | No | Yes |
| Good for simple shared flags | Yes | Can be used, but may be unnecessary |

Useful mental model:

```text
volatile
    ↓
"I need threads to see updates to this variable."

synchronized
    ↓
"I need threads to coordinate access to shared state."
```

## Good Use Case: Stop Flag

```java
class Worker implements Runnable {

    private volatile boolean running = true;

    @Override
    public void run() {
        while (running) {
            doWork();
        }
    }

    public void stop() {
        running = false;
    }
}
```

Here the shared state is a simple flag. We are not performing a compound read-modify-write operation on it. The main requirement is that the worker thread observes the state change made by another thread.

## When `volatile` Is a Poor Choice

Do not use:

```java
volatile int count;
count++;
```

as a replacement for synchronization.

The problem is that `count++` requires multiple steps and those steps can still interleave between threads.

## Mental Model

```text
                 Shared variable
                       │
              ┌────────┴────────┐
              ↓                 ↓
          Need only          Need to coordinate
          visibility?        concurrent access?
              │                 │
              ↓                 ↓
           volatile        synchronized
                                │
                                ↓
                       mutual exclusion
                              +
                           visibility
```

## Interview-Level Takeaways

- `volatile` is primarily about visibility between threads.
- A volatile write is visible to subsequent reads of that variable by other threads, with the Java Memory Model providing the relevant visibility and ordering guarantees.
- A common use case is a shared stop/shutdown flag.
- `volatile` does not provide mutual exclusion.
- `volatile` does not make compound operations such as `count++` atomic.
- Use `synchronized` or an appropriate atomic/concurrent mechanism when multiple steps must be coordinated safely.
- Do not confuse visibility with atomicity: they solve different concurrency problems.
