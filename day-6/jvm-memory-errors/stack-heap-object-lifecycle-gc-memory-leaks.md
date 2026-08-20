# Stack vs Heap, Object Lifecycle, Reachability, GC & Memory Leaks

These concepts provide the foundation for understanding `OutOfMemoryError` and `StackOverflowError`.

## Stack vs Heap

A useful conceptual model of the JVM is:

```text
JVM Process
│
├── Heap
│    └── Objects
│
└── Threads
     └── Each thread has its own Stack
          └── Stack frames
```

### Stack

Each thread has its own stack. Method invocations create stack frames containing execution state and local variables. When a method returns, its frame is removed.

Because each thread has its own stack, multiple threads have separate stacks but share the heap.

### Heap

The heap is the main shared memory area where objects are allocated.

```java
User user = new User();
```

Conceptually, `user` is a reference held by the executing context and the `User` object is allocated on the heap.

Avoid the oversimplification that "all primitives are on the stack and all objects are on the heap." The useful conceptual distinction is that stack frames hold execution state/local variables while heap memory holds objects and their instance state. Actual JVM implementation details can be more nuanced.

## Why StackOverflowError Happens

Every method invocation needs a stack frame. Excessive call depth therefore consumes stack space.

```java
static void recurse() {
    recurse();
}
```

The call chain keeps growing until the thread cannot accommodate another stack frame and throws `StackOverflowError`.

## Object Lifecycle

A useful conceptual lifecycle is:

```text
Creation
   ↓
Object is reachable
   ↓
Object is used
   ↓
References disappear
   ↓
Object becomes unreachable
   ↓
Object becomes eligible for GC
   ↓
Memory is eventually reclaimed
```

For example:

```java
void process() {
    User user = new User();
}
```

When `process()` returns, the local `user` reference disappears. If no other references point to the `User`, the object becomes unreachable and eligible for garbage collection.

**Eligible for GC does not mean immediately collected.**

## References & Reachability

Consider:

```java
User a = new User();
User b = a;
```

There is one `User` object and two references to it. If `b = null`, the object remains reachable through `a`. If both references are removed and no other reachable path points to the object, it becomes eligible for GC.

### GC Roots

Garbage collection determines reachability by tracing from a set of JVM **GC roots**. Simplified examples include references from active thread stacks and static references.

The key idea is:

> An object is reachable if there is a path from a GC root to that object.

GC cares about **reachability**, not whether the business logic still considers an object useful.

## Garbage Collection Basics

Garbage collection automatically identifies unreachable objects and eventually reclaims their memory.

GC is not simply "delete objects when a reference count reaches zero." A tracing collector can handle cycles.

For example:

```java
Node a = new Node();
Node b = new Node();

a.next = b;
b.next = a;

a = null;
b = null;
```

The two nodes reference each other, but neither is reachable from a GC root anymore. They can therefore be collected.

### `System.gc()`

```java
System.gc();
```

is only a request/hint to the JVM. It does not guarantee that GC runs immediately, so application logic should not depend on it for deterministic memory cleanup.

## Memory Leaks in Java

A Java memory leak commonly means:

> Objects remain reachable through references even though the application no longer logically needs them, preventing GC from reclaiming them.

Reachable and useful are not the same thing.

### Static Collection Leak

```java
class EventStore {
    private static final List<Event> events = new ArrayList<>();

    static void add(Event event) {
        events.add(event);
    }
}
```

If the application continually adds events without removing them:

```text
GC Root
  ↓
EventStore.class
  ↓
static events
  ↓
ArrayList
  ↓
Event
Event
Event
...
```

The events remain reachable, so GC cannot reclaim them. If this continues indefinitely, heap usage can grow until an `OutOfMemoryError` occurs.

### Listener / Callback Leak

If a long-lived publisher stores listeners and a listener is never unregistered, the publisher can keep the listener and everything reachable through it alive much longer than intended.

### Cache Leak

An unbounded cache can have the same problem:

```java
Map<String, User> cache = new HashMap<>();
```

If entries are continuously added without eviction, the referenced objects remain reachable. Production caches therefore commonly need bounds, TTLs, eviction, or explicit invalidation depending on the use case.

## Static References & Object Lifetime

`static` does **not** inherently mean "memory leak" or "never garbage collected."

The issue is that a static field generally has a very long lifetime because it is associated with class-level state.

For:

```java
class Cache {
    static User user = new User();
}
```

the `User` can become eligible for GC if the static reference is removed or replaced and no other reachable path points to the object:

```java
Cache.user = null;
```

or:

```java
Cache.user = new User("Alice");
```

If the old object has no other reachable reference, it becomes eligible for GC.

A static reference can also cease to matter if the relevant class/classloader becomes unloadable. In a normal application, however, the practical mental model is that static state lives for approximately the lifetime of the class/application.

### Why Static Mutable State Is Risky

Compare:

```text
Local reference
    ↓
method ends
    ↓
reference disappears
```

with:

```text
Instance field
    ↓
owning object becomes unreachable
    ↓
field and referenced graph can become unreachable
```

and:

```text
Static field
    ↓
long class-level lifetime
    ↓
referenced objects can remain reachable
```

This is why static mutable collections, caches, listeners, services, threads, and similar references deserve particular scrutiny.

The right principle is not "never use static":

> **Be careful with static mutable state because it can give referenced objects an unnecessarily long lifetime and prevent garbage collection.**

## Memory Leak vs OutOfMemoryError

These are not the same thing.

A memory leak is a **problem/cause**: objects that should have become collectible remain reachable.

`OutOfMemoryError` is a **failure**: the JVM cannot satisfy a memory allocation request.

A leak can eventually cause OOM:

```text
Memory leak
    ↓
Heap usage keeps increasing
    ↓
GC cannot reclaim the retained objects
    ↓
Useful free memory decreases
    ↓
Allocation eventually fails
    ↓
OutOfMemoryError
```

But OOM can also happen without a traditional memory leak, such as when a legitimate operation requests an allocation that the JVM cannot satisfy.

## Complete Mental Model

```text
                    JVM
                     │
          ┌──────────┴──────────┐
          │                     │
        STACK                  HEAP
          │                     │
     per-thread            shared by threads
          │                     │
    method frames           objects
          │                     │
          ↓                     ↓
    StackOverflow          Reachability
                              │
                     ┌────────┴────────┐
                     │                 │
                 reachable         unreachable
                     │                 │
                     │                 ↓
                     │          eligible for GC
                     │                 │
                     │                 ↓
                     │          memory reclaimed
                     │
                     └── cannot be collected
                              │
                              ↓
                     possible memory leak
                              │
                              ↓
                    possible OutOfMemoryError
```

## Interview-Level Takeaways

- Stack is per-thread and primarily manages method invocation state; heap is shared and is where objects are allocated.
- A method call creates a stack frame; excessive call depth can cause `StackOverflowError`.
- An object goes from creation → reachable/usable → unreachable → eligible for GC → eventually reclaimed.
- GC determines collectability primarily through reachability from GC roots.
- Becoming unreachable means **eligible** for GC, not immediately collected.
- GC is not simple reference counting; unreachable cyclic object graphs can also be collected.
- `System.gc()` is only a request/hint and does not guarantee immediate collection.
- A Java memory leak commonly occurs when unwanted objects remain reachable through long-lived references.
- Static mutable collections, caches, and listener registries are common leak patterns.
- `static` does not inherently cause leaks; the concern is the long lifetime of static references.
- A memory leak can eventually cause `OutOfMemoryError`, but OOM does not necessarily imply a memory leak.
