# Streams & Optional

Java Streams provide a declarative way to process sequences of elements through a pipeline of operations. `Optional<T>` represents a value that may or may not be present and provides composable operations for handling absence.

---

## 1. Stream Basics

A Stream is **not a collection or data structure**. It is a processing pipeline over a source of elements.

For example:

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6);

List<Integer> result = numbers.stream()
        .filter(n -> n % 2 == 0)
        .map(n -> n * 2)
        .collect(Collectors.toList());
```

Conceptually:

```text
numbers
   ↓
filter(even)
   ↓
map(double)
   ↓
collect
   ↓
result
```

A stream does not modify the original collection simply by applying stream operations.

---

## 2. Stream Pipeline

A stream pipeline consists of:

```text
Source
  ↓
Intermediate operations
  ↓
Terminal operation
```

Common sources include:

```java
list.stream();
set.stream();
Arrays.stream(array);
Stream.of(1, 2, 3);
```

Example:

```java
numbers.stream()
        .filter(...)
        .map(...)
        .collect(...);
```

The source produces the stream, intermediate operations build the processing pipeline, and the terminal operation produces the final result or side effect.

---

## 3. Intermediate vs Terminal Operations

### Intermediate Operations

Examples:

```java
filter()
map()
flatMap()
sorted()
distinct()
limit()
```

Intermediate operations:

- process or transform stream elements
- return another `Stream`
- are generally **lazy**
- do not cause the pipeline to execute by themselves

Example:

```java
Stream<Integer> stream = numbers.stream()
        .filter(n -> n % 2 == 0)
        .map(n -> n * 2);
```

At this point, the pipeline has been built but has not been consumed by a terminal operation.

### Terminal Operations

Examples:

```java
collect()
reduce()
forEach()
count()
findFirst()
findAny()
anyMatch()
allMatch()
```

Terminal operations:

- consume the stream
- produce a final result or side effect
- trigger execution of the pipeline

A stream is generally **single-use** after a terminal operation:

```java
Stream<Integer> stream = numbers.stream();

stream.count();
stream.toList(); // ❌ stream has already been consumed
```

If another pipeline is needed, create another stream from the source.

---

## 4. Lazy Evaluation

Intermediate operations are lazy.

For example:

```java
Stream<Integer> stream = numbers.stream()
        .filter(n -> {
            System.out.println("filter: " + n);
            return n % 2 == 0;
        });
```

Nothing is actually processed yet because there is no terminal operation.

Execution begins when a terminal operation is invoked:

```java
stream.collect(Collectors.toList());
```

Conceptually:

```text
stream()
   ↓
filter()
   ↓
map()
   ↓
        ← pipeline is lazy
   ↓
collect()
   ↓
execution starts
```

Laziness also allows the Stream implementation to process elements through the pipeline efficiently rather than necessarily materializing an intermediate collection after every operation.

For example:

```java
numbers.stream()
        .filter(n -> n % 2 == 0)
        .map(n -> n * 2)
        .collect(Collectors.toList());
```

Conceptually, an element can move through the pipeline:

```text
1 → filter → rejected
2 → filter → map → 4 → collect
3 → filter → rejected
4 → filter → map → 8 → collect
```

---

# Stream Operations

## 5. `filter`

`filter()` keeps elements that satisfy a condition.

```java
List<Integer> even = numbers.stream()
        .filter(n -> n % 2 == 0)
        .collect(Collectors.toList());
```

Result:

```text
[2, 4, 6]
```

It uses a `Predicate`, which connects directly to the functional-interface topic:

```text
Predicate<T>
    ↓
T → boolean
```

Mental model:

> **Should this element stay?**

---

## 6. `map`

`map()` transforms each element into another value.

```java
List<Integer> doubled = numbers.stream()
        .map(n -> n * 2)
        .collect(Collectors.toList());
```

The output type does not need to be the same as the input type.

For example:

```java
List<String> names = users.stream()
        .map(User::getName)
        .collect(Collectors.toList());
```

Here:

```text
User
 ↓
getName()
 ↓
String
```

Mental model:

> **What should this element become?**

Conceptually:

```text
map: T → R
```

This connects directly to:

```java
Function<T, R>
```

---

## 7. `filter` vs `map`

```text
filter
   ↓
decides whether an element stays

map
   ↓
changes what the element becomes
```

Example:

```java
numbers.stream()
        .filter(n -> n > 10)
        .map(n -> n * 2);
```

means:

```text
Numbers
   ↓
keep only values > 10
   ↓
double those values
```

---

## 8. `flatMap`

`flatMap()` is useful when one input element can produce multiple output elements and we want to flatten the result.

Suppose:

```java
class User {
    List<String> roles;
}
```

and we have:

```java
List<User> users;
```

Using `map()`:

```java
users.stream()
        .map(User::getRoles)
```

produces conceptually:

```text
Stream<List<String>>
```

For example:

```text
[
  [ADMIN, USER],
  [USER],
  [ADMIN, AUDITOR]
]
```

If we want one flat sequence:

```java
List<String> roles = users.stream()
        .flatMap(user -> user.getRoles().stream())
        .collect(Collectors.toList());
```

Result:

```text
[ADMIN, USER, USER, ADMIN, AUDITOR]
```

Mental model:

```text
map
    one element → one result

flatMap
    one element → multiple results
    then flatten them
```

Conceptually:

```text
map:
T → R

flatMap:
T → Stream<R>
then flatten
```

Another simple example:

```java
List<List<Integer>> numbers = List.of(
        List.of(1, 2),
        List.of(3, 4),
        List.of(5, 6)
);
```

```java
numbers.stream()
        .map(List::stream);       // Stream<Stream<Integer>>
```

versus:

```java
numbers.stream()
        .flatMap(List::stream);   // Stream<Integer>
```

which produces the flat sequence:

```text
1, 2, 3, 4, 5, 6
```

---

## 9. `collect`

`collect()` is a terminal operation used to accumulate stream elements into a result.

Example:

```java
List<Integer> result = numbers.stream()
        .filter(n -> n % 2 == 0)
        .collect(Collectors.toList());
```

Common collectors include:

```java
Collectors.toList()
Collectors.toSet()
Collectors.toMap(...)
Collectors.groupingBy(...)
Collectors.joining(...)
```

Modern Java also supports:

```java
stream.toList()
```

for simply obtaining an unmodifiable list.

For SDE-2 preparation, the important concept is that `collect()` is a terminal operation and `Collector`s define how stream elements are accumulated into the final result.

---

## 10. `reduce`

`reduce()` combines multiple stream elements into a single result.

For example, sum:

```java
int sum = numbers.stream()
        .reduce(0, (a, b) -> a + b);
```

Conceptually:

```text
0
 ↓
0 + 1 = 1
 ↓
1 + 2 = 3
 ↓
3 + 3 = 6
 ↓
6 + 4 = 10
```

Another example:

```java
int max = numbers.stream()
        .reduce(Integer.MIN_VALUE, Integer::max);
```

Mental model:

```text
many elements
     ↓
combine repeatedly
     ↓
one result
```

A useful comparison:

```text
map
   → many values transformed into many values

filter
   → many values → fewer values

reduce
   → many values → one value
```

---

## 11. `groupingBy`

`groupingBy()` is particularly useful in backend code for grouping objects by a key.

Suppose:

```java
class Employee {
    String name;
    String department;
}
```

We want:

```text
department → employees
```

Use:

```java
Map<String, List<Employee>> byDepartment =
        employees.stream()
                .collect(Collectors.groupingBy(
                        Employee::getDepartment
                ));
```

Conceptually:

```text
Engineering → [Tanay, Rahul]
Finance     → [Amit, Priya]
HR          → [John]
```

The classifier:

```java
Employee::getDepartment
```

determines the key.

### `groupingBy` With a Downstream Collector

We can also group and perform another aggregation:

```java
Map<String, Long> countByDepartment =
        employees.stream()
                .collect(Collectors.groupingBy(
                        Employee::getDepartment,
                        Collectors.counting()
                ));
```

Result conceptually:

```text
Engineering → 2
Finance     → 2
HR          → 1
```

General structure:

```java
groupingBy(
    classifier,
    downstreamCollector
)
```

This pattern is worth recognizing because it appears frequently in real Java backend code.

---

# Optional

## 12. What Is `Optional`?

`Optional<T>` represents a value that **may or may not be present**.

Instead of an API returning a potentially-null value:

```java
User findById(Long id);
```

an API can make the possibility of absence explicit:

```java
Optional<User> findById(Long id);
```

Conceptually:

```text
Optional<User>
      │
      ├── User exists
      │      ↓
      │   [ User ]
      │
      └── User doesn't exist
             ↓
          [ empty ]
```

The key benefit is that the API communicates absence through its type rather than relying on an undocumented `null` convention.

---

## 13. Creating an Optional

A value can be wrapped with:

```java
Optional<String> name = Optional.of("Tanay");
```

An empty Optional:

```java
Optional<String> name = Optional.empty();
```

If a value may itself be null, `Optional.ofNullable()` is commonly used:

```java
Optional<String> name = Optional.ofNullable(possiblyNullName);
```

`Optional.of(null)` throws `NullPointerException`, while `ofNullable(null)` produces `Optional.empty()`.

---

## 14. End-to-End Backend Example

Suppose a repository provides:

```java
class UserRepository {

    Optional<User> findById(Long id) {
        User user = database.find(id);

        if (user == null) {
            return Optional.empty();
        }

        return Optional.of(user);
    }
}
```

The repository communicates:

```text
Database result
      ↓
User exists → Optional.of(user)
No user     → Optional.empty()
```

A service can then safely compose operations.

Suppose `User` has an address and `Address` has a city:

```java
String city = repository.findById(id)
        .map(User::getAddress)
        .map(Address::getCity)
        .orElse("Unknown");
```

Conceptually:

```text
Optional<User>
      ↓
map(User::getAddress)
      ↓
Optional<Address>
      ↓
map(Address::getCity)
      ↓
Optional<String>
      ↓
orElse("Unknown")
      ↓
String
```

This avoids manually checking for `null` after every step.

---

## 15. Optional `map()` and Short-Circuiting

Optional also has `map()`.

If the Optional contains a value, the mapping function runs:

```text
Optional<User>
      ↓
map(User::getName)
      ↓
Optional<String>
```

If the Optional is empty, the mapping function is **not executed** and the result remains empty:

```text
Optional.empty()
      ↓
map(...)
      ↓
Optional.empty()
```

Multiple intermediate operations therefore propagate emptiness:

```java
String result = Optional.<User>empty()
        .map(User::getName)
        .map(String::toUpperCase)
        .orElse("Unknown");
```

Conceptually:

```text
Optional.empty()
      ↓
map → skipped
      ↓
Optional.empty()
      ↓
map → skipped
      ↓
Optional.empty()
      ↓
orElse("Unknown")
      ↓
"Unknown"
```

This is an important mental model:

> **An empty Optional short-circuits the chain. Intermediate functions are not executed, and the empty state continues until an operation such as `orElse`, `orElseGet`, or `orElseThrow` handles it.**

---

## 16. Avoid Overusing `isPresent()` + `get()`

This is technically valid:

```java
if (user.isPresent()) {
    User value = user.get();
}
```

but it often turns Optional back into a more verbose null-check pattern.

Prefer composable operations when appropriate:

```text
Need to transform? → map()
Need to filter?    → filter()
Need fallback?     → orElse() / orElseGet()
Need exception?    → orElseThrow()
```

The goal is not to use Optional everywhere, but to make absence explicit and handle it cleanly.

---

## 17. `orElse`

If an Optional is empty, `orElse()` provides a fallback value:

```java
String result = name.orElse("Unknown");
```

If the Optional contains a value, that value is returned.

### Important: `orElse` Is Eager

The argument to `orElse()` is evaluated before the method receives it:

```java
String result = name.orElse(getDefaultName());
```

Even if `name` is already present, `getDefaultName()` is evaluated.

Conceptually:

```text
getDefaultName()
       ↓
evaluate argument
       ↓
orElse(value)
       ↓
Optional already has value
       ↓
return existing value
```

The default computation still happened.

---

## 18. `orElseGet`

`orElseGet()` accepts a `Supplier`:

```java
String result = name.orElseGet(() -> getDefaultName());
```

The supplier is invoked only if the Optional is empty.

Conceptually:

```text
Optional has value?
    │
    ├── YES → return actual value
    │          Supplier is NOT invoked
    │
    └── NO  → invoke Supplier
              return computed fallback
```

Therefore:

```text
orElse(value)
    → eager default evaluation

orElseGet(() -> value)
    → lazy default evaluation
```

### Why This Matters

If the fallback is expensive:

```java
name.orElse(fetchFromDatabase());
```

the database call occurs even when `name` is present.

Prefer:

```java
name.orElseGet(() -> fetchFromDatabase());
```

when the fallback should only be computed if necessary.

---

## 19. Stream vs Optional Mental Model

Both APIs support composable operations, but they represent different things:

```text
Stream
    ↓
sequence of potentially many elements

Optional
    ↓
zero or one value
```

They both have operations such as `map()` and `filter()` because the same transformation idea can be applied to a sequence or to a potentially absent single value.

---

# Core Cheat Sheet

| Concept | Mental model |
|---|---|
| Stream | Pipeline for processing elements |
| Pipeline | Source → intermediate operations → terminal operation |
| Intermediate | Returns Stream; generally lazy |
| Terminal | Consumes stream and produces result/side effect |
| `filter` | Keep elements matching a predicate |
| `map` | Transform `T → R` |
| `flatMap` | Transform `T → Stream<R>` and flatten |
| `collect` | Accumulate stream into a result |
| `reduce` | Combine many elements into one result |
| `groupingBy` | Group elements by a classifier/key |
| `Optional<T>` | Zero or one value; absence is explicit |
| Optional `map` | Transform present value; skip function if empty |
| `orElse` | Fallback value; eager evaluation |
| `orElseGet` | Fallback supplier; lazy evaluation |

## Stream Operations to Internalize

```text
filter  → "Should this element stay?"

map     → "What should this element become?"

flatMap → "This element gives me multiple elements; flatten them."
```

## Optional Operations to Internalize

```text
map()       → transform if present
filter()    → keep if condition passes
orElse()    → eager fallback value
orElseGet() → lazy fallback Supplier
orElseThrow() → fail if absent
```

## Interview-Level Takeaways

- A Stream is a processing pipeline, not a collection.
- A stream pipeline consists of a source, intermediate operations, and a terminal operation.
- Intermediate operations such as `filter`, `map`, and `flatMap` are lazy and return another Stream.
- A terminal operation triggers execution and consumes the stream.
- Streams are generally single-use after a terminal operation.
- `filter` selects elements using a predicate.
- `map` transforms each element and can change its type.
- `flatMap` handles one-to-many transformations and flattens the resulting streams.
- `collect` accumulates stream elements into a result.
- `reduce` combines many elements into one result.
- `groupingBy` groups elements according to a classifier and can use downstream collectors for further aggregation.
- `Optional<T>` makes the possibility of an absent value explicit.
- An empty Optional short-circuits intermediate `map`/`filter` operations; their functions are not executed.
- `orElse()` eagerly evaluates its fallback argument.
- `orElseGet()` evaluates its fallback lazily through a `Supplier` and is preferable when fallback computation is expensive or has side effects.

> **Core mental model:** Streams process many elements through lazy pipelines; Optional represents zero or one value and lets you compose operations while propagating absence safely.