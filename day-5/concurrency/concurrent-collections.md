# Concurrent Collections

Concurrent collections are designed for safe use by multiple threads without requiring us to manually synchronize every collection operation.

For SDE2 preparation, focus on:

- `ConcurrentHashMap`
- `CopyOnWriteArrayList`
- `BlockingQueue`

## `ConcurrentHashMap`

A normal `HashMap` is not designed for concurrent modification/access by multiple threads.

```java
ConcurrentHashMap<String, Integer> counts =
        new ConcurrentHashMap<>();
```

It is designed for concurrent access and allows multiple threads to operate on the map safely.

### Important: Thread-safe does not mean arbitrary multi-step logic is atomic

This is still a compound operation:

```java
if (!map.containsKey(key)) {
    map.put(key, value);
}
```

Two threads can both observe that the key is absent before either performs the `put`.

When an atomic map operation is needed, use methods provided for that purpose:

```java
map.putIfAbsent(key, value);
map.compute(key, (k, v) -> ...);
map.computeIfAbsent(key, k -> ...);
map.merge(key, value, (old, newValue) -> ...);
```

For interviews, `putIfAbsent()` and `computeIfAbsent()` are particularly useful to know.

## `ConcurrentHashMap` vs `Collections.synchronizedMap`

You may also see:

```java
Map<String, Integer> map =
    Collections.synchronizedMap(new HashMap<>());
```

This synchronizes individual map operations, but compound sequences can still require external synchronization.

`ConcurrentHashMap` is designed specifically for concurrent access and provides useful atomic compound operations such as `putIfAbsent()`.

## `CopyOnWriteArrayList`

`CopyOnWriteArrayList` is useful when there are **many reads and very few writes**.

```java
CopyOnWriteArrayList<String> listeners =
        new CopyOnWriteArrayList<>();
```

The key idea is that when a write occurs, the underlying array is copied. Readers can continue using the existing snapshot while the new version is created.

Conceptually:

```text
Before write:
Readers → [ A B C ]

Writer:
          copy
            ↓
          [ A B C D ]

New readers → new array
Existing readers → old array
```

Because writes involve copying, it is a poor choice for write-heavy workloads.

```text
Many reads + few writes
        ↓
CopyOnWriteArrayList ✓

Many writes
        ↓
CopyOnWriteArrayList ✗
```

## `BlockingQueue`

`BlockingQueue` is useful for **producer-consumer** workflows.

```text
Producer threads
      ↓
BlockingQueue
      ↓
Consumer threads
```

Producer:

```java
queue.put(task);
```

Consumer:

```java
Task task = queue.take();
```

The important behavior is the blocking:

- If the queue is empty, `take()` waits until an element is available.
- For a bounded queue, `put()` can wait until space becomes available when the queue is full.

This lets producers and consumers coordinate without manually implementing the waiting logic.

### Example: background job processing

```text
HTTP requests
      ↓
   Producer
      ↓
BlockingQueue
      ↓
Worker threads
      ↓
Process jobs
```

Workers can simply do:

```java
while (true) {
    Job job = queue.take();
    process(job);
}
```

If there are no jobs, the worker waits. When a producer adds a job, a waiting consumer can proceed.

## When to Use What

| Collection | Main use case |
|---|---|
| `ConcurrentHashMap` | Multiple threads concurrently accessing/updating a map |
| `CopyOnWriteArrayList` | Many reads and very few writes |
| `BlockingQueue` | Producer-consumer / work queues |

Mental model:

```text
Need concurrent Map?
        ↓
ConcurrentHashMap

Need read-heavy concurrent List?
        ↓
CopyOnWriteArrayList

Need workers consuming jobs?
        ↓
BlockingQueue
```

## Interview-Level Takeaways

- Normal collections such as `HashMap` and `ArrayList` are not generally designed for concurrent modification/access.
- `ConcurrentHashMap` is designed for concurrent map access and provides useful atomic operations.
- Thread-safe individual operations do not automatically make arbitrary multi-step sequences atomic.
- Use methods such as `putIfAbsent()` or `computeIfAbsent()` when an atomic map operation is required.
- `CopyOnWriteArrayList` is suited to read-heavy, write-rare workloads because writes copy the underlying array.
- `BlockingQueue` is useful for producer-consumer workflows.
- `take()` waits when a `BlockingQueue` is empty; `put()` can wait when a bounded queue is full.
