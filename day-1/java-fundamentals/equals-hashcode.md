# `==` vs `equals()` vs `hashCode()`

## Overview

These three concepts are closely related to object identity, logical equality, and hash-based collections.

- `==` → identity/reference comparison for objects; value comparison for primitives.
- `equals()` → logical equality between objects, as defined by the class.
- `hashCode()` → integer hash value used by hash-based collections such as `HashMap` and `HashSet`.

## `==`

### Primitives

For primitives, `==` compares the actual values.

```java
int a = 10;
int b = 10;

System.out.println(a == b); // true
```

### Objects

For object references, `==` checks whether both references point to the **exact same object**.

```java
Person p1 = new Person("Tanay");
Person p2 = new Person("Tanay");

System.out.println(p1 == p2); // false
```

Conceptually:

```text
p1 ───────> Person("Tanay")

p2 ───────> Person("Tanay")
```

The objects contain the same data but are different objects.

If two references point to the same object:

```java
Person p1 = new Person("Tanay");
Person p2 = p1;

System.out.println(p1 == p2); // true
```

Conceptually:

```text
p1 ─────┐
        ├────> same Person object
p2 ─────┘
```

### Rule

> **For objects, `==` checks reference identity: are these references pointing to the exact same object?**

## `equals()`

`equals()` is used to determine **logical equality** between objects.

The result depends on the implementation of `equals()` in the class.

### `Object.equals()`

Every Java class ultimately inherits from `Object`.

If a class does not override `equals()`, it inherits `Object.equals()`.

The behavior of `Object.equals()` is essentially:

```java
public boolean equals(Object obj) {
    return this == obj;
}
```

Therefore, the default `equals()` implementation behaves like reference identity comparison.

For example:

```java
class Person {
    String name;
}

Person p1 = new Person();
Person p2 = new Person();

System.out.println(p1.equals(p2)); // false
```

Because `Person` has not overridden `equals()`, the inherited `Object.equals()` effectively checks:

```java
p1 == p2
```

If:

```java
Person p2 = p1;
```

then:

```java
p1.equals(p2); // true
```

because both references point to the same object.

### Why Override `equals()`?

A class can override `equals()` when it needs a definition of **logical equality**.

For example, a `Person` class might define two people as equal when they have the same ID.

```java
class Person {
    private Long id;
    private String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Person other)) return false;

        return Objects.equals(id, other.id);
    }
}
```

Now two different objects can be logically equal:

```java
Person p1 = new Person(1L, "Tanay");
Person p2 = new Person(1L, "Rahul");

System.out.println(p1.equals(p2)); // true
```

The class's equality definition determines the result.

## `String` Example

`String` overrides `equals()` to compare its contents.

```java
String s1 = new String("hello");
String s2 = new String("hello");

System.out.println(s1 == s2);      // false
System.out.println(s1.equals(s2)); // true
```

- `s1 == s2` → different objects, so `false`.
- `s1.equals(s2)` → same string content, so `true`.

This is the classic example of the difference between **identity** and **logical equality**.

## `hashCode()`

Every Java object has a `hashCode()` method.

```java
int hash = object.hashCode();
```

- It returns an integer hash value based on the object's `hashCode()` implementation.
- Hash-based collections use this value to help organize and locate objects.
- A hash code is **not a unique object ID**.

Different objects can have the same hash code. This is called a **hash collision** and is allowed.

```text
Object A → hash 42
Object B → hash 42
```

The collection can use `equals()` to distinguish objects that collide.

## The `equals()` / `hashCode()` Contract

The most important rule is:

> **If two objects are equal according to `equals()`, they must have the same `hashCode()`.**

Formally:

```text
a.equals(b) == true
        ↓
a.hashCode() == b.hashCode()
```

However, the reverse is **not guaranteed**:

```text
a.hashCode() == b.hashCode()
        ↓
does NOT imply
a.equals(b) == true
```

Different objects can have the same hash code because of collisions.

## Why `HashMap` and `HashSet` Care

Hash-based collections use both `hashCode()` and `equals()`.

Conceptually, for a lookup such as:

```java
map.get(key);
```

the process is roughly:

```text
key
 ↓
hashCode()
 ↓
determine candidate bucket/location
 ↓
compare candidate keys
 ↓
equals()
 ↓
matching key found
```

Therefore:

- `hashCode()` helps determine **where to look**.
- `equals()` determines **whether the candidate key is actually equal**.

This avoids having to compare a lookup key against every key in a large collection.

## Why Overriding `equals()` Without `hashCode()` Is a Bug

Suppose a class overrides `equals()` based on `id` but does not override `hashCode()` consistently.

Then two logically equal objects may produce different hash codes.

For example:

```java
class Person {
    Long id;

    @Override
    public boolean equals(Object o) {
        // compares id
    }

    // hashCode() is not overridden
}
```

You can end up with:

```text
p1.equals(p2) == true

but

p1.hashCode() != p2.hashCode()
```

Now hash-based collections may look in different locations for the two objects.

For example, a `HashSet` can fail to recognize a logically equal object as already present, or a `HashMap` lookup can fail unexpectedly.

### Rule

> **Whenever you override `equals()`, you should also override `hashCode()` consistently with the equality definition.**

## Mutable Hash-Based Keys

Be careful when fields used by `equals()` and `hashCode()` can change after an object has been inserted into a `HashMap` or `HashSet`.

For example:

```java
class Person {
    int id;

    @Override
    public boolean equals(Object o) {
        // based on id
    }

    @Override
    public int hashCode() {
        return Integer.hashCode(id);
    }
}
```

Then:

```java
Person p = new Person(1);

Map<Person, String> map = new HashMap<>();
map.put(p, "Tanay");
```

If the key is later mutated:

```java
p.id = 2;
```

its hash code may change.

The map placed the entry using the old hash value, so a later lookup can fail:

```java
map.get(p);
```

Therefore:

> **Mutable fields that participate in `equals()`/`hashCode()` are dangerous when the object is used as a key in hash-based collections.**

## The Three Concepts Side-by-Side

| Operation | What it answers |
|---|---|
| `==` | Are these the same primitive value / same object reference? |
| `equals()` | Are these objects logically equal according to the class's equality definition? |
| `hashCode()` | What hash value represents this object for use by hash-based collections? |

For objects:

```text
p1 == p2
    ↓
same object?

p1.equals(p2)
    ↓
same logical equality?

p1.hashCode()
    ↓
hash value used by hash-based collections
```

## Interview-Level Takeaways

- For primitives, `==` compares values.
- For object references, `==` compares identity/reference.
- `equals()` is used for logical equality.
- `Object.equals()` effectively behaves like `this == obj` unless a class overrides it.
- `String` overrides `equals()` to compare content.
- If `a.equals(b)` is `true`, `a.hashCode()` **must equal** `b.hashCode()`.
- Equal hash codes do **not** guarantee that `equals()` is true.
- Different objects can have the same hash code because of collisions.
- `HashMap` and `HashSet` use `hashCode()` to narrow the search and `equals()` to determine equality.
- Overriding `equals()` without a consistent `hashCode()` implementation is a bug for hash-based collections.
- Mutable fields used in `equals()`/`hashCode()` can cause problems when the object is used as a hash-based collection key.
