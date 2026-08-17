# HashMap Internals

`HashMap` is one of the highest-value Java Collections topics for backend development and DSA. The central idea is how a key can be used to find its value efficiently.

At a high level:

```text
key
 ↓
hash
 ↓
bucket
 ↓
entry
 ↓
value
```

The goal of the implementation is to make lookup, insertion, and removal approximately **O(1) on average**.

## 1. The Underlying Data Structure

At its core, a `HashMap` maintains an **array of buckets** (the internal table).

Conceptually:

```text
bucket array

index
  0    ┌──────────┐
       │          │
  1    ├──────────┤
       │          │
  2    ├──────────┤
       │          │
  3    ├──────────┤
       │          │
  ...  │          │
       └──────────┘
```

When we execute:

```java
map.put(42, "Tanay");
```

Java needs to determine which bucket should contain the entry for key `42`.

This is where hashing comes in.

> A `HashMap` is not simply an array. It is an array of buckets, where a bucket can contain multiple entries.

---

## 2. Hashing

Every Java object has a `hashCode()` method.

For example:

```java
Integer key = 42;
int hash = key.hashCode();
```

For `Integer`, the hash code is essentially the integer value:

```text
42 → hashCode() → 42
```

The hash must then be converted into a valid bucket index.

A simplified mental model is:

```text
index = hash % bucketCount
```

For example, with 16 buckets:

```text
42 % 16 = 10
```

so the key could be placed in bucket `10`.

**Important:** this modulo calculation is useful for understanding the idea. Modern Java `HashMap` uses a more specific bitwise calculation because its table sizes are powers of two.

The important conceptual flow is:

```text
hashCode()
    ↓
hash
    ↓
bucket index
```

---

## 3. Collisions

Different keys can map to the same bucket.

For example:

```text
42 → bucket 10
58 → bucket 10
74 → bucket 10
```

This is a **hash collision**.

The map cannot simply overwrite the existing entry, so a bucket can contain multiple entries:

```text
bucket 10

42 → Tanay
58 → Rahul
74 → Amit
```

Historically and in normal collision chains, these entries can be represented as a linked structure. Modern Java can also convert a heavily populated collision chain into a red-black tree.

Conceptually:

```text
Bucket
  │
  ├── Node
  ├── Node
  └── Node
```

or, after treeification:

```text
Bucket
  │
  └── Tree
       ├── Node
       ├── Node
       └── Node
```

Therefore:

> **Collision handling is a fundamental part of how a hash table works.**

---

## 4. What Is Inside an Entry?

Conceptually, a HashMap node contains information similar to:

```text
Node
├── hash
├── key
├── value
└── next
```

For a collision chain:

```text
bucket[10]
    ↓
┌──────────────────────┐
│ hash                 │
│ key = 42             │
│ value = "Tanay"      │
│ next ──────────────┐ │
└────────────────────┘ │
                       ▼
                ┌─────────────────┐
                │ hash            │
                │ key = 58        │
                │ value = "Rahul" │
                │ next = null     │
                └─────────────────┘
```

The `next` reference links entries in a collision chain.

---

## 5. How `put()` Works

Consider:

```java
map.put(42, "Tanay");
```

At a simplified level:

```text
put(42, "Tanay")
       │
       ▼
 key.hashCode()
       │
       ▼
     hash
       │
       ▼
 bucket index
       │
       ▼
 bucket[index]
       │
 ┌─────┴─────┐
 │           │
empty     occupied
 │           │
 ▼           ▼
insert    compare keys
             │
          equals()
```

### If the bucket is empty

The new entry can be inserted directly.

### If the bucket is occupied

Java needs to determine whether the incoming key is already present.

- If the existing key matches, the value is replaced.
- If it is a different key with the same bucket, it is a collision and the entry is added to the collision structure.

This is why `HashMap` depends on both `hashCode()` and `equals()`.

---

## 6. Why Both `hashCode()` and `equals()`?

Suppose:

```java
map.put(key1, "Tanay");
```

and later:

```java
map.get(key2);
```

where:

```java
key1.equals(key2) == true
```

We want `get(key2)` to find the value associated with `key1`.

First, `key2.hashCode()` helps locate the relevant bucket. Then `equals()` identifies the exact matching key inside that bucket.

The conceptual flow is:

```text
key
 ↓
hashCode()
 ↓
bucket
 ↓
equals()
 ↓
exact key
 ↓
value
```

### The Contract

If two objects are equal according to `equals()`, they **must** have the same `hashCode()`.

Otherwise, an equivalent key could be placed in one bucket while the lookup searches another bucket.

> **Hashing narrows the search; equality confirms the match.**

---

## 7. Why `hashCode()` Alone Is Not Enough

`hashCode()` is not guaranteed to be unique.

For example:

```text
key A → hash 100
key B → hash 100
```

Both keys can therefore land in the same bucket even though they are not equal.

So:

```text
hashCode()
    ↓
"Look in this bucket."

     then

equals()
    ↓
"This is / is not the exact key."
```

Never think of `hashCode()` as a unique identifier.

---

## 8. How `get()` Works

Consider:

```java
map.get(42);
```

### Step 1 — Calculate the hash

```text
42
 ↓
hashCode()
 ↓
hash
```

### Step 2 — Find the bucket

```text
hash
 ↓
bucket index
 ↓
bucket 10
```

### Step 3 — Search the bucket

If there is no collision, the matching entry may be found immediately.

If the bucket contains multiple entries:

```text
bucket 10

42 → Tanay
58 → Rahul
74 → Amit
```

Java compares the requested key against candidate keys using equality until the matching key is found.

Conceptually:

```text
42.equals(42)? → true
```

Then the corresponding value is returned.

---

## 9. Why Is `HashMap` O(1) Average?

Instead of searching every entry:

```text
search every element → O(n)
```

hashing narrows the search:

```text
key
 ↓
hash
 ↓
bucket
 ↓
small number of candidates
```

If keys are distributed reasonably well, each bucket contains relatively few entries, so the amount of work is approximately constant.

Therefore:

> `HashMap.get()`, `put()`, and `remove()` are typically **O(1) on average**.

This is an average-case expectation, not a guarantee that every individual operation always takes constant time.

---

## 10. Treeification in Modern Java

A heavily populated collision bucket can degrade lookup performance if it remains a linked chain:

```text
A → B → C → D → E → ...
```

Searching the chain can become:

```text
O(n)
```

Modern Java `HashMap` can convert a sufficiently large collision chain into a **red-black tree**.

Conceptually:

```text
Before:

bucket
  ↓
A → B → C → D → E

After:

bucket
  ↓
      C
     / \
    B   E
   /   /
  A   D
```

Searching a balanced tree is approximately:

```text
O(log n)
```

in the number of entries in that bucket.

For SDE-2 preparation, the important point is:

> **Modern HashMap can treeify heavily populated collision buckets to improve lookup behavior.**

You do not need to memorize every implementation threshold at this stage.

---

## 11. Resizing

Imagine a table with only 16 buckets and a continuously growing number of entries.

Eventually the buckets become increasingly crowded, which increases the chance of collisions.

To maintain good performance, `HashMap` **resizes** its internal table when its size reaches the relevant threshold.

Conceptually:

```text
Before:

16 buckets
┌─┬─┬─┬─┬─┬─┬─┬─┐
│ │●│ │●│●│ │ │●│
└─┴─┴─┴─┴─┴─┴─┴─┘

        ↓ resize

Larger table
┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┐
│ │ │●│ │ │●│ │ │●│ │ │ │●│ │ │ │
└─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┘
```

Entries must be redistributed according to the new table size.

Most insertions are cheap, but an insertion that triggers a resize can require work proportional to the number of entries.

Therefore:

> **`HashMap.put()` is O(1) average/amortized, but resizing can make an individual insertion O(n).**

---

## 12. Load Factor

The **load factor** controls how full the hash table is allowed to become before resizing.

Conceptually:

```text
load factor = number of entries / number of buckets
```

The default load factor of Java's `HashMap` is:

```text
0.75
```

So, conceptually, with 16 buckets:

```text
16 × 0.75 = 12
```

The relevant resize threshold is therefore around 12 entries.

The load factor represents a trade-off:

### Higher load factor

```text
less memory
more entries per bucket
potentially more collisions
```

### Lower load factor

```text
more memory
fewer entries per bucket
potentially fewer collisions
```

`0.75` is a practical balance between memory usage and collision frequency.

---

## 13. Why HashMap Capacities Are Powers of Two

Java's `HashMap` uses table sizes that are powers of two. This allows efficient bucket-index calculation using bitwise operations.

A simplified conceptual comparison is:

```text
hash % capacity
```

versus the power-of-two optimization:

```text
hash & (capacity - 1)
```

For example, with:

```text
capacity = 16
capacity - 1 = 15
```

Java can use the bitwise relationship to efficiently obtain the bucket index.

For interview preparation, the important relationship is:

```text
capacity is a power of 2
        ↓
efficient bucket index calculation
```

You do not need to memorize the exact bit manipulation unless specifically asked about implementation details.

---

## 14. Complete `put()` Mental Model

Putting everything together:

```text
                 PUT / GET
                     │
                     ▼
                   Key
                     │
                     ▼
                hashCode()
                     │
                     ▼
               hash calculation
                     │
                     ▼
                bucket index
                     │
                     ▼
              table[bucket]
                     │
             ┌───────┴────────┐
             │                │
          no collision     collision
             │                │
             ▼                ▼
           Node        linked list / tree
                              │
                              ▼
                           equals()
                              │
                              ▼
                        exact key match
                              │
                              ▼
                            value
```

Around this core mechanism, `HashMap` also has:

```text
HashMap
 │
 ├── bucket array
 ├── hashing
 ├── collision handling
 ├── equals() / hashCode()
 ├── resizing
 ├── load factor
 └── treeification
```

---

# 15. Why HashSet Uses the Same Principles

`HashSet` is backed by a `HashMap` internally.

Conceptually:

```text
HashSet<E>
     ↓
HashMap<E, Object>
```

When you write:

```java
Set<String> set = new HashSet<>();
set.add("Tanay");
```

the set can be thought of as storing the element as a key with a dummy/presence value:

```text
"Tanay" → PRESENT
"Rahul" → PRESENT
"Amit"  → PRESENT
```

The actual value is not important. The keys provide the uniqueness semantics.

Therefore the same mechanism applies:

```text
HashSet
   ↓
HashMap
   ↓
hashCode()
   ↓
bucket
   ↓
equals()
```

This is why correct `equals()` / `hashCode()` implementations are important for custom objects in both `HashSet` and `HashMap`.

---

# 16. DSA Connection

This same idea is the basis of the **hash table** data structure used throughout DSA.

Suppose we repeatedly need to ask:

> "Have I seen this value before?"

With a list:

```text
contains(X) → O(n)
```

With a hash set:

```java
Set<Integer> seen = new HashSet<>();
```

we typically get:

```text
seen.contains(X) → O(1) average
seen.add(X)      → O(1) average
```

This is why hash-based structures appear constantly in DSA problems.

Common patterns include:

### Detect duplicates

```java
Set<Integer> seen = new HashSet<>();
```

### Two Sum / indexed lookup

```java
Map<Integer, Integer> seen = new HashMap<>();
```

### Frequency counting

```java
Map<Character, Integer> frequency = new HashMap<>();
```

### Grouping

```java
Map<String, List<String>> groups = new HashMap<>();
```

The underlying idea is the same:

```text
"I need to associate/find something quickly."
                ↓
           Hash table
```

---

# Key Traps / Clarifications

### `hashCode()` is not a unique identifier

Different keys can have the same hash code. Collisions are expected and must be handled.

### `hashCode()` and `equals()` have different jobs

```text
hashCode() → narrows down the bucket
 equals()  → confirms the exact key
```

### O(1) does not mean guaranteed O(1)

`HashMap` operations are typically O(1) on average. Collisions and resizing can make individual operations more expensive.

### HashMap is not just an array

It is an array/table of buckets, with collision structures hanging from those buckets.

### HashSet is closely tied to HashMap

`HashSet` is backed by a `HashMap`, which is why their hashing and equality behavior is fundamentally the same.

---

# Interview-Level Takeaways

- `HashMap` uses a table/array of buckets internally.
- A key's `hashCode()` is used to determine where to look.
- Multiple keys can map to the same bucket: this is a **collision**.
- Collisions are handled using linked structures and, in sufficiently populated buckets, treeification into a red-black tree.
- `equals()` is used to determine whether a candidate key is actually the requested key.
- If `a.equals(b)` is `true`, `a.hashCode()` and `b.hashCode()` must be equal.
- `hashCode()` alone does not uniquely identify a key.
- `HashMap.get()` / `put()` / `remove()` are typically O(1) average.
- A collision-heavy bucket can make lookup more expensive; treeification improves the behavior of heavily populated buckets.
- Resizing occurs as the table becomes sufficiently full and requires redistribution of entries.
- The default `HashMap` load factor is `0.75`.
- HashMap capacities use powers of two, enabling efficient bucket-index calculation.
- `HashSet` is backed by a `HashMap` and therefore relies on the same hashing/equality principles.
- The core mental model is:

```text
key
 ↓
hashCode()
 ↓
bucket
 ↓
equals()
 ↓
exact entry
 ↓
value
```

> **Best mental shortcut:** `hashCode()` gets you to the neighborhood; `equals()` finds the exact house.
