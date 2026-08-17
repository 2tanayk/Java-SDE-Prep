# Collection Hierarchy

## What Are Collections?

A **collection** is an object that holds a group of other objects.

Instead of managing individual values separately:

```java
String user1 = "Tanay";
String user2 = "Rahul";
String user3 = "Amit";
```

we can represent the group as a collection:

```java
List<String> users = new ArrayList<>();

users.add("Tanay");
users.add("Rahul");
users.add("Amit");
```

Java provides the **Collections Framework**: a set of interfaces, implementations, and utility classes for storing and manipulating groups of objects.

Common abstractions include:

- `List`
- `Set`
- `Queue`
- `Deque`
- `Map`

The framework gives us standard, reusable implementations instead of requiring us to implement data structures such as dynamic arrays, linked lists, hash tables, trees, and queues ourselves.

## Program to the Interface

A key idea throughout the Collections Framework is separating **what we need** from **how it is implemented**.

```java
List<String> users = new ArrayList<>();
```

Here:

- `List` describes the required abstraction/behavior.
- `ArrayList` is the concrete implementation.

Conceptually:

```text
List        → what I need
ArrayList   → how I want it implemented
```

This is why collection variables are commonly declared using interfaces rather than concrete classes.

> **Program to the interface, not the implementation.**

The choice of concrete implementation still matters because different implementations have different performance characteristics and behavior.

## The Big Picture

The most important hierarchy to understand is:

```text
Iterable [I]
    │
    ▼
Collection [I]
    │
    ├── List [I]
    │    ├── ArrayList [C]
    │    ├── LinkedList [C]
    │    └── Vector [C]
    │
    ├── Set [I]
    │    ├── HashSet [C]
    │    │    └── LinkedHashSet [C]
    │    └── SortedSet [I]
    │         └── NavigableSet [I]
    │              └── TreeSet [C]
    │
    └── Queue [I]
         ├── PriorityQueue [C]
         └── Deque [I]
              ├── ArrayDeque [C]
              └── LinkedList [C]
```

`Map` is part of the Java Collections Framework but is **not** a subtype of `Collection`:

```text
Map [I]
 ├── HashMap [C]
 │    └── LinkedHashMap [C]
 └── SortedMap [I]
      └── NavigableMap [I]
           └── TreeMap [C]
```

Legend:

- `[I]` = interface
- `[AC]` = abstract class
- `[C]` = concrete class

The concrete classes shown above are the implementations that are most useful to recognize at the SDE-2 level. There are also abstract skeletal implementation classes in the JDK, discussed below.

## `Iterable`

`Iterable<T>` is the abstraction for objects that can provide an `Iterator` over their elements.

It is what allows an object to be used with the enhanced `for` loop:

```java
for (String user : users) {
    System.out.println(user);
}
```

Conceptually, the enhanced `for` loop uses an iterator:

```java
Iterator<String> iterator = users.iterator();

while (iterator.hasNext()) {
    String user = iterator.next();
}
```

You normally do not need to write this manually, but understanding the relationship is useful:

```text
Iterable [I]
    ↓
Collection [I]
    ↓
List / Set / Queue
```

## `Collection`

`Collection<E>` is the core interface representing a **group of individual elements**.

It provides common operations such as:

```java
add()
remove()
contains()
size()
isEmpty()
clear()
iterator()
```

For example:

```java
Collection<String> names = new ArrayList<>();

names.add("Tanay");
names.add("Rahul");

System.out.println(names.size());
System.out.println(names.contains("Tanay"));
```

`Collection` does not by itself specify all the semantics we may want from a collection. Its subinterfaces provide more specific contracts.

## `List`

A `List` represents an **ordered sequence of elements**.

Important properties:

- Elements have a defined position/index.
- Positional access is supported.
- Duplicate elements are allowed.

Example:

```java
List<String> names = new ArrayList<>();

names.add("Tanay");
names.add("Tanay");
names.add("Rahul");

System.out.println(names.get(1));
```

The list can contain:

```text
[Tanay, Tanay, Rahul]
```

Common implementations:

```text
List [I]
 ├── ArrayList [C]
 ├── LinkedList [C]
 └── Vector [C]
```

`ArrayList` and `LinkedList` are Core topics for Day 2.

## `Set`

A `Set` represents a collection where **duplicate elements are not allowed**.

Example:

```java
Set<String> names = new HashSet<>();

names.add("Tanay");
names.add("Rahul");
names.add("Tanay");
```

The set contains only one occurrence of `Tanay`.

An important trap is to say that a `Set` simply "doesn't maintain order." The ordering behavior depends on the implementation.

### `HashSet`

```java
Set<String> names = new HashSet<>();
```

- Does not guarantee iteration order.
- Enforces uniqueness.

### `LinkedHashSet`

```java
Set<String> names = new LinkedHashSet<>();
```

- Enforces uniqueness.
- Maintains insertion order during iteration.

### `TreeSet`

```java
Set<String> names = new TreeSet<>();
```

- Enforces uniqueness.
- Maintains elements according to their sorted ordering.

The hierarchy is:

```text
Set [I]
 ├── HashSet [C]
 │    └── LinkedHashSet [C]
 └── SortedSet [I]
      └── NavigableSet [I]
           └── TreeSet [C]
```

For this roadmap, `HashSet` and `TreeSet` are Core; `LinkedHashSet` is Secondary.

## `Queue`

A `Queue` represents a collection designed around **processing elements**.

The classic queue model is FIFO:

```text
A → B → C → D
↑
next to process
```

If elements are inserted in this order:

```text
A, B, C
```

then a normal FIFO queue processes:

```text
A, B, C
```

However, an important nuance is:

> A `Queue` does not necessarily mean strict FIFO ordering.

For example, `PriorityQueue` orders elements according to its priority rules rather than simply using insertion order.

## `Deque`

`Deque` means **double-ended queue**.

It allows insertion and removal at both ends:

```text
        front
          ↓
     A  B  C  D  E
          ↑
         back
```

For example:

```java
deque.addFirst(...);
deque.addLast(...);

deque.removeFirst();
deque.removeLast();
```

A `Deque` can therefore be used as a queue or, with the appropriate operations, as a stack.

The Day 2 roadmap treats `Deque` as a Secondary topic.

## Why Is `Map` Separate?

A common misconception is to imagine:

```text
Collection
    └── Map
```

This is incorrect.

`Map<K, V>` is part of the Java Collections Framework, but it does **not** extend `Collection<E>`.

The reason is that the abstractions are fundamentally different.

A `Collection` represents a group of individual elements:

```text
Collection<E>
    ↓
 element
 element
 element
```

A `Map` represents key-value mappings:

```text
Map<K, V>
    ↓
 key → value
 key → value
 key → value
```

For example:

```java
List<String> users = ...;
```

contains individual elements:

```text
Tanay
Rahul
Amit
```

whereas:

```java
Map<Integer, String> users = ...;
```

contains mappings:

```text
101 → Tanay
102 → Rahul
103 → Amit
```

Therefore:

> **`Map` is part of the Collections Framework, but `Map` is not a `Collection`.**

The Map hierarchy relevant to this roadmap is:

```text
Map [I]
 ├── HashMap [C]
 │    └── LinkedHashMap [C]
 └── SortedMap [I]
      └── NavigableMap [I]
           └── TreeMap [C]
```

`HashMap` is a Core topic; `LinkedHashMap` is Secondary; `TreeMap` is Core.

## Abstract Classes in the Collections Hierarchy

The JDK also provides abstract skeletal implementation classes that contain reusable behavior for concrete collection implementations.

Important examples include:

```text
AbstractCollection [AC]
AbstractList       [AC]
AbstractSet        [AC]
AbstractQueue      [AC]
AbstractMap        [AC]
```

For example, the list hierarchy includes:

```text
AbstractList [AC]
 ├── ArrayList [C]
 └── Vector [C]

AbstractSequentialList [AC]
 └── LinkedList [C]
```

The exact implementation inheritance is less important for the current preparation than recognizing the distinction between interfaces, abstract skeletal implementations, and concrete collection classes.

> **Do not spend time memorizing every abstract class in the JDK.** For SDE-2 preparation, the interface and concrete implementation relationships are much more important.

## `Collection` vs `Collections`

These names are easy to confuse but refer to different things.

### `Collection`

`Collection<E>` is an **interface** representing a group of elements.

### `Collections`

`Collections` is a **utility class** containing static helper methods for working with collections.

Examples:

```java
Collections.sort(list);
Collections.reverse(list);
Collections.shuffle(list);
```

Therefore:

```text
Collection  → interface
Collections → utility class
```

## `Arrays`

`Arrays` is another utility class, but it is primarily for working with arrays.

Examples:

```java
Arrays.sort(array);
Arrays.binarySearch(array, value);
Arrays.asList(...);
```

Do not confuse:

```text
Collection  → interface
Collections → collection utility class
Arrays      → array utility class
```

## The Mental Model

When choosing a collection, first identify the behavior you need.

```text
"I need an ordered sequence."
        ↓
      List

"I need unique elements."
        ↓
       Set

"I need elements waiting to be processed."
        ↓
      Queue

"I need operations from both ends."
        ↓
      Deque

"I need to associate one thing with another."
        ↓
       Map
```

Then choose the implementation based on the required behavior and performance characteristics:

```text
List  → ArrayList / LinkedList

Set   → HashSet / LinkedHashSet / TreeSet

Queue → PriorityQueue / ...

Deque → ArrayDeque / LinkedList

Map   → HashMap / LinkedHashMap / TreeMap
```

## Interview-Level Takeaways

- The **Collections Framework** is broader than the `Collection` interface.
- `Iterable` is the abstraction that supports iteration and the enhanced `for` loop.
- `Collection` represents a group of individual elements.
- `List` represents an ordered sequence and allows duplicates.
- `Set` represents unique elements; ordering depends on the implementation.
- `HashSet` does not guarantee iteration order.
- `LinkedHashSet` maintains insertion order.
- `TreeSet` maintains sorted ordering.
- `Queue` is designed around processing elements; do not assume every queue is strictly FIFO.
- `Deque` supports operations at both ends.
- `Map` stores key-value mappings and does **not** extend `Collection`.
- `Map` is still part of the Java Collections Framework.
- `Collection` and `Collections` are different: the former is an interface, the latter is a utility class.
- Abstract collection classes such as `AbstractList` and `AbstractMap` provide reusable skeletal implementations; they are less important to memorize than the main interfaces and concrete implementations.
- Prefer declaring variables against collection interfaces where practical:

```java
List<String> users = new ArrayList<>();
Map<Long, String> usersById = new HashMap<>();
```

> **Core mental model:** first choose the abstraction (`List`, `Set`, `Queue`, `Deque`, or `Map`) based on the behavior you need; then choose the concrete implementation based on its characteristics and trade-offs.
