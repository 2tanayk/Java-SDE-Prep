# Java SDE 2 Prep

Notes and revision material for SDE 2 Java developer interview preparation.

## Day 1 — Java Fundamentals + OOP
- [JDK / JRE / JVM](day-1/java-fundamentals/jdk-jre-jvm.md)
- [Class Loading Basics](day-1/java-fundamentals/class-loading.md)
- [Class Initialization Order](day-1/java-fundamentals/class-initialization-order.md)
- [`static`](day-1/java-fundamentals/static.md)
- [`final`](day-1/java-fundamentals/final.md)
- [`this` / `super`](day-1/java-fundamentals/this-super.md)
- [Access Modifiers](day-1/java-fundamentals/access-modifiers.md)
- [`==` vs `equals()` vs `hashCode()`](day-1/java-fundamentals/equals-hashcode.md)
- [Inheritance](day-1/java-fundamentals/inheritance.md)
- [Polymorphism](day-1/java-fundamentals/polymorphism.md)
- [Encapsulation](day-1/java-fundamentals/encapsulation.md)
- [Abstraction](day-1/java-fundamentals/abstraction.md)
- [Immutability](day-1/java-fundamentals/immutability.md)
- [Class initialization order](day-1/java-fundamentals/class-initialization-order.md)
- [String, String Pool, StringBuilder & StringBuffer](day-1/java-fundamentals/string-stringbuilder-stringbuffer.md)
- [Nested & Inner Classes](day-1/java-fundamentals/nested-and-inner-classes.md)
- [Exceptions, Errors & Exception Handling](day-1/java-fundamentals/exceptions-errors-exception-handling.md)

## Day 2 — Collections
- [Collection Hierarchy](day-2/collections/collection-hierarchy.md)
- [Core Collections — Usage & Core Operations](day-2/collections/core-collections-usage.md)
- [HashMap Internals](day-2/collections/hashmap-internals.md)
- [Iterator, For-Each & Fail-Fast](day-2/collections/iterator-foreach-fail-fast.md)

## Day 3 — Generics + Java 8
- [Generic Classes & Methods](day-3/generics/generic-classes-methods.md)
- [Generic Bounds](day-3/generics/generic-bounds.md)
- [Wildcards, `? extends`, `? super` & PECS](day-3/generics/wildcards-extends-super-pecs.md)
- [Type erasure](day-3/generics/java_type_erasure.md)
- [Functional Interfaces, Lambdas & Method References](day-3/java-8/functional-interfaces-lambdas-method-references.md)
- [Effectively-final lambda capture](day-3/java-8/java_lambda_capture.md)

## Day 4 — Streams + Optional
- [Streams & Optional](day-3/java-8/streams-optional.md)

## Day 5 — Concurrency
- [Thread Basics & Lifecycle](day-5/concurrency/thread-basics-and-lifecycle.md)
- [Race Conditions, Shared Mutable State & Thread Safety](day-5/concurrency/race-conditions-shared-state-thread-safety.md)
- [`synchronized`](day-5/concurrency/synchronized.md)
- [`volatile`](day-5/concurrency/volatile.md)
- [`ExecutorService` & Thread Pools](day-5/concurrency/executor-service-thread-pools.md)
- [Atomic Classes](day-5/concurrency/atomic-classes.md)
- [Concurrent Collections](day-5/concurrency/concurrent-collections.md)
- [Deadlocks](day-5/concurrency/deadlocks.md)
- [`CompletableFuture`](day-5/concurrency/completable-future.md)

## Day 6 — JVM Memory + Exceptions
- [OutOfMemoryError & StackOverflowError](day-6/jvm-memory-errors/out-of-memory-and-stack-overflow.md)
- [Stack vs Heap, Object Lifecycle, Reachability, GC & Memory Leaks](day-6/jvm-memory-errors/stack-heap-object-lifecycle-gc-memory-leaks.md)

## Day 7 — Modern Java + Design Patterns

## Day 8 — File I/O

### 1. `java.io.File` API Basics

`File` is Java's legacy API for representing a file or directory path. A `File` object represents a pathname; creating the object does not create anything on the filesystem.

```java
File file = new File("data.txt");
```

The actual file can be created with:

```java
file.createNewFile();
```

`createNewFile()` returns `true` if a new file was created and `false` if it already existed, and it can throw `IOException`.

#### File vs Directory

A `File` object can represent either a file or a directory. The filesystem determines what actually exists at that path.

```java
file.exists();
file.isFile();
file.isDirectory();
```

#### File Information

Useful methods include:

```java
file.getName();
file.getPath();
file.getAbsolutePath();
file.length();
file.lastModified();
```

- `getName()` returns the final path component.
- `getPath()` returns the path used to construct the `File`.
- `getAbsolutePath()` resolves the path to an absolute path.
- `length()` returns the file size in bytes for a regular file.
- `lastModified()` returns the last-modified timestamp.

#### Listing a Directory

```java
File directory = new File("/tmp");
String[] names = directory.list();
File[] files = directory.listFiles();
```

`list()` returns entry names, while `listFiles()` returns `File` objects.

```java
for (File file : directory.listFiles()) {
    if (file.isFile()) {
        System.out.println("FILE: " + file.getName());
    } else if (file.isDirectory()) {
        System.out.println("DIR: " + file.getName());
    }
}
```

#### Creating Directories

```java
new File("documents").mkdir();
new File("a/b/c").mkdirs();
```

- `mkdir()` creates a single directory and requires its parent to already exist.
- `mkdirs()` creates the directory and any missing parent directories.

### Key Takeaway

> `File` is primarily a path abstraction, not a file-content abstraction. Reading and writing file contents is handled by stream/reader/writer APIs and, in modern Java, by NIO.2's `Path` and `Files` APIs.

## Recommended Priority Order

### Tier 1 — Nail

### Tier 2 — Know

### Tier 3 — Don't Touch Yet

## Target Level

For each Core topic, aim to answer:

- What is it?
- How does it work?
- When/why would I choose it in production?
