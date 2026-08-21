# `synchronized`

## Core Idea

`synchronized` is Java's built-in mechanism for coordinating access to shared state using a lock.

The key mental model is:

> **`synchronized` is about acquiring a lock, not about locking a method itself.**

When a thread enters synchronized code, it must acquire the relevant lock. If another thread already holds that same lock, the second thread waits until the lock is released.

This provides two important benefits:

```text
synchronized
      │
      ├── Mutual exclusion
      │      → only one thread at a time holds the lock
      │
      └── Visibility
             → changes become visible across the same lock
```

## Intrinsic Locks / Monitors

Every Java object can be associated with an intrinsic lock (monitor).

For example:

```java
class Counter {
    private int count = 0;

    synchronized void increment() {
        count++;
    }
}
```

Conceptually, the object has an associated lock that Java manages automatically.

## `synchronized` Instance Method

```java
synchronized void increment() {
    count++;
}
```

A synchronized instance method synchronizes on the current object (`this`).

Conceptually, the locking behavior is equivalent to:

```java
void increment() {
    synchronized (this) {
        count++;
    }
}
```

So for:

```java
Counter counter = new Counter();
```

calling `counter.increment()` requires the thread to acquire `counter`'s intrinsic lock.

## Multiple Synchronized Instance Methods

Consider:

```java
class Counter {
    private int count;

    synchronized void setCount(int value) {
        count = value;
    }

    synchronized int getCount() {
        return count;
    }
}
```

Both methods use the **same object lock** when called on the same `Counter` instance.

Therefore, if Thread A is executing `setCount()` on a particular `Counter` object and Thread B tries to execute `getCount()` on that **same object**, B cannot acquire the lock until A releases it.

This is why writing `synchronized` on multiple instance methods is not saying that each method has a separate lock. It means each method requires the object's lock while it executes.

## Different Objects Have Different Locks

Consider:

```java
Counter c1 = new Counter();
Counter c2 = new Counter();
```

Conceptually:

```text
c1 → 🔒 Lock 1
c2 → 🔒 Lock 2
```

A thread using `c1` does not block a thread using `c2` merely because both methods are synchronized.

The key rule is:

> **Same object → same instance lock → threads contend for that lock.**
>
> **Different objects → different instance locks → they do not block each other because of instance synchronization.**

## `synchronized` Block

You do not have to synchronize an entire method.

```java
void increment() {
    // non-critical work

    synchronized (this) {
        count++;
    }

    // more non-critical work
}
```

Only the block requires the lock. This can reduce unnecessary lock contention because other threads are blocked only while the critical section needs protection.

## Synchronizing on an Explicit Object

You can choose a dedicated lock object:

```java
class Counter {
    private final Object lock = new Object();
    private int count;

    void increment() {
        synchronized (lock) {
            count++;
        }
    }
}
```

Here the lock is `lock`, not `this`.

A private lock can be useful when you do not want external code to be able to synchronize on the same object.

## `static synchronized`

A static synchronized method does not synchronize on a particular instance because no instance is required.

```java
class Counter {
    static int count;

    static synchronized void increment() {
        count++;
    }
}
```

It synchronizes on the class object:

```text
Counter.class
     ↓
    🔒
```

Conceptually, its locking behavior is equivalent to:

```java
static void increment() {
    synchronized (Counter.class) {
        count++;
    }
}
```

## Instance Lock vs Class Lock

### Instance synchronized method

```java
synchronized void foo() {
}
```

Locks `this`.

### Static synchronized method

```java
static synchronized void foo() {
}
```

Locks `ClassName.class`.

Therefore an instance synchronized method and a static synchronized method do **not** block each other merely because they belong to the same class: they use different locks.

## `synchronized` and Visibility

Synchronization is not only about making threads take turns.

If Thread A updates shared state while holding a lock and releases that lock, and Thread B later acquires the **same lock**, B can see the changes made by A before the lock was released.

Conceptually:

```text
Thread A                     Thread B

gets lock
   ↓
count = 10
   ↓
releases lock
                              ↓
                         gets same lock
                              ↓
                         reads count
                              ↓
                            10
```

So the two key properties to remember are:

- **Mutual exclusion** — one thread at a time can hold the same lock.
- **Visibility** — changes become visible across the same lock.

The formal Java Memory Model `happens-before` terminology can be introduced later when it becomes useful.

## Exception Handling

If synchronized execution exits because of an exception, the monitor is still released automatically.

```java
synchronized void process() {
    doSomething();
    throw new RuntimeException();
}
```

Conceptually:

```text
acquire lock
     ↓
execute
     ↓
exception
     ↓
lock released
```

## What `synchronized` Does Not Mean

It does **not** mean that only one thread in the entire application can execute synchronized code.

It means:

> **Only one thread can hold a particular monitor lock at a time.**

Different objects can have different locks and therefore allow synchronized code to execute concurrently.

## Mental Model

Whenever you see `synchronized`, ask:

> **What lock is this using?**

- `synchronized void method()` → `this`
- `static synchronized void method()` → `ClassName.class`
- `synchronized (someObject)` → `someObject`

Then:

```text
Thread wants lock
       ↓
Is another thread holding it?
       │
    ┌──┴──┐
   Yes    No
    │      │
  waits  acquires
    │      │
    └──┬───┘
       ↓
   executes
       ↓
   releases
```

## Interview-Level Takeaways

- `synchronized` is about acquiring a lock; it does not mean the method itself owns a unique lock.
- A synchronized instance method locks `this`.
- Multiple synchronized instance methods on the same object use the same intrinsic lock.
- Two different instances have different intrinsic locks.
- A synchronized block can protect only the required critical section.
- `synchronized (someObject)` locks `someObject`.
- A static synchronized method locks the class object (`ClassName.class`).
- Instance and static synchronization use different locks and therefore do not automatically block each other.
- `synchronized` provides mutual exclusion and visibility across the same lock.
- The monitor is released automatically when synchronized execution exits, including because of an exception.
