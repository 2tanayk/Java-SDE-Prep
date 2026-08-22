# Deadlocks

## Core Idea

A **deadlock** occurs when two or more threads wait for each other forever, so none of them can proceed.

The classic pattern involves two locks:

```java
Object lockA = new Object();
Object lockB = new Object();
```

Thread A:

```java
synchronized (lockA) {
    synchronized (lockB) {
        // work
    }
}
```

Thread B:

```java
synchronized (lockB) {
    synchronized (lockA) {
        // work
    }
}
```

Possible execution:

```text
Thread A                    Thread B

acquires lockA              acquires lockB
     ↓                           ↓
tries lockB                 tries lockA
     ↓                           ↓
   WAIT                        WAIT
     ↑                           ↑
     └──────── forever ──────────┘
```

A waits for B's lock, while B waits for A's lock. Neither can proceed.

## The Important Pattern

Deadlocks commonly arise from **inconsistent lock acquisition order**:

```text
Thread A:
    Lock 1
    ↓
    wants Lock 2

Thread B:
    Lock 2
    ↓
    wants Lock 1
```

The problem is not simply using multiple locks; the dangerous part is the circular waiting caused by acquiring them in different orders.

## Preventing Deadlocks

The simplest and most important strategy is:

> **Always acquire multiple locks in the same order.**

For example, establish:

```text
Lock A → Lock B
```

Then every thread follows that order:

```java
synchronized (lockA) {
    synchronized (lockB) {
        // ...
    }
}
```

Now if another thread wants both locks, it also attempts `lockA` first. It may wait for `lockA`, but it cannot simultaneously hold `lockB` while waiting for `lockA`, so the circular wait is avoided.

## Practical Example: Bank Transfer

Suppose a transfer needs to lock two accounts:

```java
synchronized (accountA) {
    synchronized (accountB) {
        transfer();
    }
}
```

Another transfer doing the reverse order can cause a deadlock:

```text
Transfer 1              Transfer 2

lock Account A          lock Account B
       ↓                       ↓
want Account B          want Account A
       ↓                       ↓
   WAIT                    WAIT
```

A practical solution is to establish a deterministic ordering, such as always locking the account with the smaller ID first.

Then both directions acquire:

```text
Account 5 → Account 10
```

regardless of which account is the source or destination.

## `tryLock()`

With `synchronized`, attempting to acquire an unavailable lock causes the thread to wait.

With `Lock` implementations such as `ReentrantLock`, `tryLock()` can attempt to acquire a lock without waiting forever:

```java
if (lock.tryLock()) {
    try {
        // work
    } finally {
        lock.unlock();
    }
}
```

Conceptually:

```text
Try Lock A
    ↓
success?
 ┌──┴──┐
Yes    No
 ↓      ↓
try B  give up/retry
```

`tryLock()` can therefore be useful in designs where indefinite waiting should be avoided.

## Deadlock vs Race Condition

Do not confuse these concepts.

### Race condition

Threads execute concurrently and interfere with shared state, potentially producing an incorrect result.

```text
A ──┐
    ├── shared state → incorrect result
B ──┘
```

### Deadlock

Threads become stuck waiting for each other.

```text
A → holds Lock 1 → waits for Lock 2
B → holds Lock 2 → waits for Lock 1
```

Mental distinction:

```text
Race condition → potentially wrong result
Deadlock       → threads stuck / no progress
```

## Practical Prevention

- Acquire multiple locks in a consistent global order.
- Avoid unnecessary nested locks.
- Keep critical sections small.
- Use timed/non-blocking lock acquisition where appropriate, such as `tryLock()`.
- Be careful when calling external code while holding locks.

## Interview-Level Takeaways

- A deadlock occurs when threads wait for each other indefinitely.
- The classic example is Thread A holding Lock 1 while waiting for Lock 2, while Thread B holds Lock 2 while waiting for Lock 1.
- Inconsistent lock ordering is a common cause.
- The most important prevention technique is to acquire multiple locks in a consistent order.
- `tryLock()` can help avoid indefinite waiting when using explicit lock APIs.
- A race condition can produce an incorrect result; a deadlock prevents progress.
