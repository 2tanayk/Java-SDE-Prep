# Core Collections — Usage & Core Operations

For practical backend development, the most important collection implementations to be comfortable with are:

- `ArrayList` — ordered collection of elements
- `HashSet` — unique elements
- `HashMap` — key → value lookup

The goal here is to know their purpose, common operations, and typical time complexity rather than memorize every API method.

## Quick Selection Guide

```text
                     What do I need?
                           │
             ┌─────────────┼──────────────┐
             │             │              │
             ▼             ▼              ▼
       Ordered data    Unique data    Key → Value
             │             │              │
             ▼             ▼              ▼
         ArrayList       HashSet       HashMap
```

---

# 1. ArrayList

## What Is It?

`ArrayList<E>` is a **resizable array**.

Unlike a normal Java array, whose size is fixed after creation:

```java
String[] users = new String[10];
```

an `ArrayList` can grow dynamically:

```java
List<String> users = new ArrayList<>();

users.add("Tanay");
users.add("Rahul");
users.add("Amit");
```

Conceptually:

```text
[ Tanay ][ Rahul ][ Amit ]
    0        1        2
```

Because it is array-backed, `ArrayList` provides efficient positional access.

## Core Operations

### Add

```java
users.add("Tanay");
```

Adds an element to the end.

- Typical complexity: **O(1) amortized**
- Occasionally the backing array must grow and elements must be copied, making that particular operation O(n).

### Get by Index

```java
String user = users.get(2);
```

- Complexity: **O(1)**

The array backing allows direct access to an element by index.

### Replace

```java
users.set(2, "Vivek");
```

Replaces the element at the specified index.

- Complexity: **O(1)**

### Remove by Index

```java
users.remove(2);
```

Removing from the middle can require subsequent elements to shift left:

```text
Before:
[ A ][ B ][ C ][ D ][ E ]
          ↑
        remove

After:
[ A ][ B ][ D ][ E ]
```

- Typical complexity: **O(n)**

### Search

```java
users.contains("Rahul");
users.indexOf("Rahul");
```

A value search may have to inspect every element.

- Complexity: **O(n)**

### Size

```java
users.size();
```

- Complexity: **O(1)**

## ArrayList Complexity Summary

| Operation | Typical Complexity |
|---|---:|
| `add(element)` | O(1) amortized |
| `get(index)` | O(1) |
| `set(index, value)` | O(1) |
| `remove(index)` | O(n) |
| `contains(value)` | O(n) |
| `indexOf(value)` | O(n) |
| `size()` | O(1) |

## Backend Usage

`ArrayList` is commonly used when dealing with groups of objects such as:

```java
List<User> users = userRepository.findAll();
List<Order> orders = ...;
List<String> roles = new ArrayList<>();
```

It is generally the default `List` implementation unless there is a specific reason to choose another implementation.

---

# 2. HashSet

## What Is It?

`HashSet<E>` is a collection designed for **unique elements**.

```java
Set<Integer> ids = new HashSet<>();

ids.add(101);
ids.add(102);
ids.add(103);
ids.add(101);
```

The duplicate `101` is not added.

The fundamental property is:

> **A `Set` guarantees uniqueness.**

`HashSet` does not guarantee iteration order.

## Core Operations

### Add

```java
boolean added = ids.add(101);
```

`add()` returns:

- `true` — the element was added
- `false` — the element was already present

Typical complexity: **O(1) average**

### Check Existence

```java
ids.contains(101);
```

Typical complexity: **O(1) average**

This is a major reason to use a `HashSet` when the primary requirement is fast membership checking.

Compare:

```java
List<Integer> ids = ...;
ids.contains(101);    // O(n)
```

with:

```java
Set<Integer> ids = ...;
ids.contains(101);    // O(1) average
```

### Remove

```java
ids.remove(101);
```

Typical complexity: **O(1) average**

### Size

```java
ids.size();
```

- Complexity: **O(1)**

### Iterate

```java
for (Integer id : ids) {
    System.out.println(id);
}
```

Do not rely on a particular order. `HashSet` does **not** guarantee insertion order or sorted order.

## HashSet and `equals()` / `hashCode()`

A critical connection to remember from Java fundamentals is:

```text
HashSet
   ↓
hashing + equality
   ↓
hashCode() + equals()
```

When storing custom objects in a `HashSet`, correct `equals()` / `hashCode()` implementations are essential for the set's uniqueness semantics to work correctly.

This is also fundamental to understanding `HashMap`.

## HashSet Complexity Summary

| Operation | Typical Complexity |
|---|---:|
| `add()` | O(1) average |
| `contains()` | O(1) average |
| `remove()` | O(1) average |
| `size()` | O(1) |
| Iteration | O(n) |

These are **typical/average** complexities. Hash-based collections can have more expensive behavior in collision-heavy cases.

## Backend Usage

Use `HashSet` when uniqueness or fast membership checking is the main requirement:

```java
Set<Long> userIds = new HashSet<>();
Set<String> roles = new HashSet<>();
Set<String> permissions = new HashSet<>();
```

---

# 3. HashMap

## What Is It?

`HashMap<K, V>` stores **key-value mappings**:

```text
key → value
```

Example:

```java
Map<Long, User> usersById = new HashMap<>();

usersById.put(101L, user1);
usersById.put(102L, user2);
usersById.put(103L, user3);
```

Conceptually:

```text
101 → User(Tanay)
102 → User(Rahul)
103 → User(Amit)
```

The key is used to efficiently locate the associated value.

## Core Operations

### `put()`

```java
usersById.put(101L, user1);
```

Typical complexity: **O(1) average**

Map keys are unique. If the key already exists:

```java
usersById.put(101L, user2);
```

the old value is replaced:

```text
101 → user1
```

becomes:

```text
101 → user2
```

Important:

> **Keys are unique; values do not have to be.**

### `get()`

```java
User user = usersById.get(101L);
```

Typical complexity: **O(1) average**

This is the primary use case for a `HashMap`: efficient lookup by key.

Instead of repeatedly searching a list:

```java
for (User user : users) {
    if (user.getId().equals(id)) {
        // found user
    }
}
```

you can build an index:

```java
Map<Long, User> usersById = ...;
```

and perform:

```java
usersById.get(id);
```

### `containsKey()`

```java
usersById.containsKey(101L);
```

Checks whether a key exists.

- Typical complexity: **O(1) average**

### `containsValue()`

```java
usersById.containsValue(user);
```

`HashMap` is optimized around keys, not value lookup. Searching values generally requires traversing the values.

- Complexity: **O(n)**

### `remove()`

```java
usersById.remove(101L);
```

- Typical complexity: **O(1) average**

### `size()`

```java
usersById.size();
```

- Complexity: **O(1)**

### `getOrDefault()`

Useful when a fallback value is needed:

```java
int count = counts.getOrDefault("Tanay", 0);
```

If the key exists, its value is returned; otherwise `0` is returned.

A common counting pattern is:

```java
Map<String, Integer> counts = new HashMap<>();

for (String word : words) {
    counts.put(
        word,
        counts.getOrDefault(word, 0) + 1
    );
}
```

### `putIfAbsent()`

```java
map.putIfAbsent("Tanay", 100);
```

Inserts the mapping only if the key is not already present.

### Iteration

When both key and value are needed, use `entrySet()`:

```java
for (Map.Entry<Long, User> entry : usersById.entrySet()) {
    Long id = entry.getKey();
    User user = entry.getValue();
}
```

For keys only:

```java
for (Long id : usersById.keySet()) {
    // ...
}
```

For values only:

```java
for (User user : usersById.values()) {
    // ...
}
```

In general, prefer `entrySet()` when you need both the key and value because the mapping is already represented by each `Map.Entry`.

## HashMap Complexity Summary

| Operation | Typical Complexity |
|---|---:|
| `put()` | O(1) average |
| `get()` | O(1) average |
| `containsKey()` | O(1) average |
| `remove()` | O(1) average |
| `containsValue()` | O(n) |
| `size()` | O(1) |
| `getOrDefault()` | O(1) average |
| `putIfAbsent()` | O(1) average |

Again, these are **average/typical** complexities. The internal collision behavior of `HashMap` will be covered separately.

## HashMap and `equals()` / `hashCode()`

The same fundamental relationship seen with `HashSet` applies here:

```text
HashMap
   ↓
hashing + equality
   ↓
hashCode() + equals()
```

For a custom object used as a key, the `equals()` / `hashCode()` contract is critical. A violation can result in mappings that cannot be retrieved or behaved with as expected.

This is one of the highest-value areas to understand before going deeper into `HashMap` internals.

## Backend Usage

`HashMap` is extremely common in backend development:

```java
Map<Long, User> usersById;
Map<String, String> headers;
Map<String, Object> attributes;
Map<String, List<Order>> ordersByCustomer;
Map<Long, Permissions> permissionsByUser;
```

Typical use cases include:

- Fast lookup/indexing by an identifier
- Grouping data by a key
- Counting/frequency maps
- Building temporary lookup tables
- Representing key-value configuration or attributes

---

# ArrayList vs HashSet vs HashMap

| Characteristic | `ArrayList` | `HashSet` | `HashMap` |
|---|---|---|---|
| Primary purpose | Ordered elements | Unique elements | Key → value mappings |
| Duplicates | Allowed | Not allowed | Keys not allowed; values allowed |
| Ordering | List order/index | No guaranteed iteration order | No guaranteed iteration order |
| Lookup | By index or value | By element | By key |
| Typical lookup | O(1) by index / O(n) by value | O(1) average | O(1) average by key |
| Typical add/insert | O(1) amortized at end | O(1) average | O(1) average |
| Main mechanism | Resizable array | Hashing | Hashing |

## Backend-Oriented Decision Rule

Use:

```text
ArrayList
    ↓
"I need an ordered sequence of objects."
```

```text
HashSet
    ↓
"I need unique objects / fast membership checks."
```

```text
HashMap
    ↓
"I need to find a value efficiently using a key."
```

The choice should be driven by the **behavior you need**, not simply by which collection is familiar.

## Important Complexity Caveat

When we say `HashSet` or `HashMap` operations are **O(1) average**, that is not the same as saying they are mathematically guaranteed to take constant time in every possible situation.

Hash-based collections can experience collisions, and collision behavior can make an individual operation more expensive.

For now, the important mental model is:

```text
ArrayList → O(1) index access
HashSet   → O(1) average membership
HashMap   → O(1) average key lookup
```

The detailed `HashMap` mechanics—hashing, buckets, collisions, resizing, and related implementation details—will be covered separately.

## Interview-Level Takeaways

- `ArrayList` is a resizable, array-backed `List`.
- `ArrayList.get(index)` is O(1).
- `ArrayList` value search is O(n).
- `ArrayList` insertion at the end is O(1) amortized; resizing can make an individual insertion O(n).
- `ArrayList` removal from the middle can be O(n) because elements may need to shift.
- `HashSet` enforces uniqueness and does not guarantee iteration order.
- `HashSet.contains()` is O(1) average.
- `HashSet.add()` returns `false` when the element is already present.
- `HashSet` relies on hashing and equality, making correct `equals()` / `hashCode()` implementations important for custom objects.
- `HashMap` stores key-value mappings and has unique keys.
- `HashMap.get()` and `containsKey()` are O(1) average.
- `HashMap.containsValue()` is O(n) because values are not the structure's lookup key.
- `HashMap.put()` replaces the existing value when the key already exists.
- `getOrDefault()` and `putIfAbsent()` are useful everyday `HashMap` operations.
- Use `entrySet()` when iterating over both keys and values.
- Hash-based complexity figures are average/typical; collision behavior will be studied separately.
