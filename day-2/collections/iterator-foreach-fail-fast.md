# Iterator, For-Each & Fail-Fast Iteration

## Iterator

`Iterator` provides a standard way to traverse a collection one element at a time.

```java
List<String> users = List.of("Tanay", "Rahul", "Amit");

Iterator<String> iterator = users.iterator();

while (iterator.hasNext()) {
    String user = iterator.next();
    System.out.println(user);
}
```

The two core methods to remember are:

```java
iterator.hasNext(); // Is there another element?
iterator.next();    // Return the next element
```

The relationship is:

```text
Collection
    ↓
iterator()
    ↓
Iterator
    ↓
hasNext() / next()
```

An iterator is particularly useful when an element needs to be **removed while the collection is being traversed**.

```java
Iterator<String> it = users.iterator();

while (it.hasNext()) {
    String user = it.next();

    if (user.equals("Rahul")) {
        it.remove(); // safe iterator-controlled removal
    }
}
```

---

## For-Each

The enhanced `for` loop is the convenient way to iterate over a collection:

```java
for (String user : users) {
    System.out.println(user);
}
```

Conceptually, for collections, this uses an iterator behind the scenes:

```java
Iterator<String> it = users.iterator();

while (it.hasNext()) {
    String user = it.next();
    // ...
}
```

Therefore:

> **For-each is cleaner syntax for ordinary traversal; an explicit `Iterator` is useful when you need iterator-specific behavior such as safe removal.**

---

## Fail-Fast Concept

Suppose a collection is being traversed using an iterator and the collection is structurally modified directly during that traversal:

```java
for (String user : users) {
    if (user.equals("Rahul")) {
        users.remove(user); // ❌
    }
}
```

This can result in:

```text
ConcurrentModificationException
```

The idea is that the iterator has assumptions about the collection's structure. If the collection unexpectedly changes underneath it, continuing the traversal could produce incorrect or inconsistent behavior.

Fail-fast behavior therefore aims to detect the unexpected structural modification and stop early rather than silently continuing.

Conceptually:

```text
Iterator starts
     ↓
collection state = X
     ↓
iteration begins
     ↓
collection is structurally modified externally
     ↓
iterator detects unexpected modification
     ↓
ConcurrentModificationException
```

### Why Is Fail-Fast Useful?

It is primarily a **bug-detection mechanism**.

Without fail-fast behavior, accidental modification during traversal could potentially result in skipped elements, duplicate processing, or otherwise incorrect traversal.

With fail-fast behavior, the problem becomes visible immediately through an exception instead of potentially producing silently incorrect results.

> **Fail-fast means: detect an unexpected structural modification during iteration and fail rather than continue with potentially invalid traversal state.**

### Important: It Is Not Thread Safety

Do not interpret fail-fast as meaning that the collection is thread-safe.

Fail-fast is primarily about detecting unexpected modification during iteration. It is a **best-effort safety mechanism**, not a synchronization mechanism and not a guarantee against concurrent modification by multiple threads.

---

## Why `Iterator.remove()` Works

This is the key distinction:

```java
Iterator<String> it = users.iterator();

while (it.hasNext()) {
    String user = it.next();

    if (user.equals("Rahul")) {
        it.remove(); // ✅
    }
}
```

The iterator itself is performing the removal, so it can keep its internal traversal state consistent.

Compare:

```text
users.remove(...)
      ↓
external structural modification
      ↓
❌ may trigger fail-fast behavior
```

with:

```text
iterator.remove()
      ↓
iterator-controlled modification
      ↓
✅ supported
```

### Practical Rule

> **Do not structurally modify a collection through the collection itself while iterating over it. If you need to remove the current element during iteration, use `Iterator.remove()`.**

---

## What If I Need to Add While Iterating?

A normal `Iterator` does not provide an `add()` operation.

For a `List`, use `ListIterator` when you need to modify the list during traversal:

```java
ListIterator<String> it = users.listIterator();

while (it.hasNext()) {
    String user = it.next();

    if (user.equals("Rahul")) {
        it.add("Amit"); // ✅
    }
}
```

`ListIterator` supports:

```text
ListIterator
├── next()
├── previous()
├── remove()
├── add()
└── set()
```

So the useful mental model is:

```text
Need to remove while iterating
        ↓
Iterator.remove()

Need to add/remove/replace while iterating a List
        ↓
ListIterator
```

`ListIterator` is specifically for `List` implementations; a generic `Iterator` does not provide the same add/set capabilities.

### Simpler Alternative

If modification does not need to happen during traversal, it is often clearer to collect the elements to add and modify the collection afterward:

```java
List<String> toAdd = new ArrayList<>();

for (String user : users) {
    if (someCondition(user)) {
        toAdd.add("Amit");
    }
}

users.addAll(toAdd);
```

---

## One More Important Distinction

Iterator-based modification is specifically relevant to **structural modification** of the collection.

Changing the state of an object contained inside the collection is different:

```java
for (User user : users) {
    user.setActive(false);
}
```

This does not structurally modify the `users` collection; it modifies the objects stored inside it.

---

## Quick Summary

```text
Iterator
→ Standard way to traverse a collection.

for-each
→ Convenient syntax for collection traversal; uses Iterator underneath.

Iterator.remove()
→ Safe way to remove the current element during iteration.

Fail-fast
→ Detects unexpected structural modification during iteration
  and may throw ConcurrentModificationException.

ListIterator
→ Use for List traversal when you need add/remove/set operations.
```

### Core Interview Takeaways

- `Iterator` provides `hasNext()` and `next()` for traversal.
- Enhanced `for-each` uses an iterator for collections.
- Direct structural modification of a collection during iteration can cause `ConcurrentModificationException`.
- Fail-fast is intended to expose this kind of programming error early.
- Fail-fast is **not** a thread-safety mechanism.
- `Iterator.remove()` is the supported way to remove the current element during iterator traversal.
- `ListIterator` adds `add()` and `set()` capabilities for lists.
- If modification can happen outside the traversal, doing it afterward is often simpler and clearer.
