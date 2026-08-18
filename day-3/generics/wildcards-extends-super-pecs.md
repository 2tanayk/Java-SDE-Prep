# Wildcards, `? extends`, `? super` & PECS

Wildcards are used when we want to work with a parameterized type without needing to know or name its exact type argument.

The four concepts in this note form one chain:

```text
Wildcards
   │
   ├── ?
   ├── ? extends T
   ├── ? super T
   └── PECS
       ├── Producer → extends
       └── Consumer → super
```

---

## 1. Why Do We Need Wildcards?

Java generics are **invariant** by default.

Even though:

```text
String extends Object
```

this does **not** mean:

```text
List<String> extends List<Object>
```

For example, this does not work as a general way to accept every kind of list:

```java
void printList(List<Object> list) {
    // ...
}
```

A `List<String>` is not a subtype of `List<Object>`.

Wildcards solve this by allowing us to express an unknown type argument.

---

## 2. Unbounded Wildcard — `?`

```java
List<?> list
```

means:

> A `List` of some unknown type.

It could represent:

```java
List<String>
List<Integer>
List<User>
List<Double>
```

Therefore:

```java
void printList(List<?> list) {
    // ...
}
```

can accept lists of different element types.

### Reading from `List<?>`

You can safely read an element as `Object`:

```java
Object value = list.get(0); // ✅
```

You cannot safely assume a more specific type:

```java
String value = list.get(0); // ❌
```

because the actual list could be `List<Integer>` or something else.

### Writing to `List<?>`

You generally cannot add a specific value:

```java
list.add("Tanay"); // ❌
list.add(123);      // ❌
```

because the actual list type is unknown.

`null` is allowed because it is compatible with any reference type:

```java
list.add(null); // ✅
```

Mental model:

```text
List<?>

READ  → Object
WRITE → essentially nothing except null
```

---

## 3. `? extends T`

An upper-bounded wildcard:

```java
List<? extends Number>
```

means:

> A `List` of some unknown type that is `Number` or a subtype of `Number`.

It could be:

```java
List<Integer>
List<Double>
List<Long>
List<Number>
```

Conceptually:

```text
             Number
            /  |   \
           /   |    \
      Integer Double Long

List<? extends Number>
        ↑
Could be any of these lists
```

### Reading

You can safely read values as the upper bound:

```java
Number number = list.get(0); // ✅
```

because every possible element type is a `Number`.

You can also read as `Object`:

```java
Object value = list.get(0); // ✅
```

But you cannot safely read directly as a specific subtype:

```java
Integer value = list.get(0); // ❌
```

because the actual list could be `List<Double>`.

### Writing

You cannot safely add a specific value:

```java
List<? extends Number> numbers = ...;
numbers.add(10); // ❌
```

Why? Because the actual list might be:

```java
List<Double> doubles = new ArrayList<>();
List<? extends Number> numbers = doubles;
```

Adding an `Integer` through `numbers` would violate the actual `List<Double>` type.

`null` remains allowed.

Mental model:

```text
List<? extends Number>

READ  → Number
WRITE → ❌ specific values
```

---

## 4. `? extends` as a Producer

`? extends T` is useful when the collection is **producing values for us**.

Example:

```java
double sum(List<? extends Number> numbers) {
    double total = 0;

    for (Number number : numbers) {
        total += number.doubleValue();
    }

    return total;
}
```

The method does not care whether the caller supplies:

```java
List<Integer>
List<Double>
List<Long>
```

It only needs to read values as `Number`.

Therefore the list is a **Producer**.

> **Producer → `extends`**

---

## 5. `? super T`

A lower-bounded wildcard:

```java
List<? super Integer>
```

means:

> A `List` of some unknown type that is `Integer` or one of its supertypes.

It could represent:

```java
List<Integer>
List<Number>
List<Object>
```

because:

```text
Object
  ↑
Number
  ↑
Integer
```

---

## 6. Writing to `? super T`

You can safely add values of the lower-bound type:

```java
List<? super Integer> list = ...;

list.add(10); // ✅
list.add(20); // ✅
```

Why is this safe?

Regardless of the actual list type:

```text
List<Integer>
List<Number>
List<Object>
```

an `Integer` can be stored in all three.

### Reading from `? super T`

You cannot safely read an element as `Integer`:

```java
Integer value = list.get(0); // ❌
```

The actual list could be:

```java
List<Object>
```

so the element might be some arbitrary `Object`.

You can safely read it as `Object`:

```java
Object value = list.get(0); // ✅
```

Mental model:

```text
List<? super Integer>

WRITE → Integer ✅
READ  → Object
```

---

## 7. `? super` as a Consumer

Suppose a library method creates Dogs and puts them into a caller-provided list:

```java
static void createDogs(List<? super Dog> destination) {
    destination.add(new Dog("Bruno"));
    destination.add(new Dog("Max"));
}
```

The caller can provide:

```java
List<Dog> dogs = new ArrayList<>();
createDogs(dogs);
```

or:

```java
List<Animal> animals = new ArrayList<>();
createDogs(animals);
```

or:

```java
List<Object> objects = new ArrayList<>();
createDogs(objects);
```

The library method does not know which concrete list type it received. But it knows that **all of these lists can safely accept a `Dog`**.

Therefore this is guaranteed:

```java
destination.add(new Dog("Bruno")); // ✅
```

But this is not:

```java
destination.add(new Animal("Generic Animal")); // ❌
```

because the caller might have provided a `List<Dog>`, and an arbitrary `Animal` is not necessarily a `Dog`.

The list is therefore a **Consumer** of `Dog` values.

> **Consumer → `super`**

### Important Clarification

`? super Dog` does **not** mean:

> "I can add anything above `Dog`."

It means:

> "I do not know the exact list type, but I know that whatever it is, it can safely consume a `Dog`."

---

## 8. PECS — Producer Extends, Consumer Super

The famous rule is:

> **PECS = Producer Extends, Consumer Super**

From the perspective of the code using the generic collection:

```text
Producer
   ↓
I GET values from the collection
   ↓
? extends T
```

and:

```text
Consumer
   ↓
I PUT values into the collection
   ↓
? super T
```

### Producer Example

```java
void processDogs(List<? extends Dog> source) {
    for (Dog dog : source) {
        System.out.println(dog.name);
    }
}
```

The method gets Dogs from the list, so the list is a producer.

### Consumer Example

```java
void addDogs(List<? super Dog> destination) {
    destination.add(new Dog("Bruno"));
}
```

The method puts Dogs into the list, so the list is a consumer.

---

## 9. Classic Combined Example

A method that copies values naturally demonstrates both sides:

```java
void copy(
    List<? super T> destination,
    List<? extends T> source
) {
    // read from source
    // write to destination
}
```

Conceptually:

```text
source
   ↓
produces T
   ↓
? extends T


destination
   ↑
consumes T
   ↑
? super T
```

This is the essence of PECS.

---

## 10. Wildcard vs Generic Type Parameter

Do not confuse:

```java
<T extends Number>
```

with:

```java
<? extends Number>
```

### `<T extends Number>`

This declares a **named type parameter**:

```java
public <T extends Number> T process(T value) {
    return value;
}
```

We can refer to `T` elsewhere in the method.

### `<? extends Number>`

This represents an **unknown type** satisfying that bound:

```java
List<? extends Number> numbers
```

We do not need to name the exact type because we only care that it is some subtype of `Number`.

Mental distinction:

```text
<T extends Number>
        ↓
Named type parameter

<? extends Number>
        ↓
Unknown type
```

---

## 11. Why Use a Wildcard Instead of a Type Parameter?

If a method does not need to refer to the exact type, a wildcard often expresses the intent more directly.

For example:

```java
void printNumbers(List<? extends Number> numbers) {
    for (Number number : numbers) {
        System.out.println(number);
    }
}
```

The method does not care whether the list is `List<Integer>`, `List<Double>`, or `List<Number>`. It only cares that it can safely read `Number`s.

The wildcard communicates:

> **"I don't care what the exact type is."**

---

## 12. Complete Mental Model

### `?`

```text
Unknown type
    ↓
Read as Object
Write basically nothing except null
```

### `? extends T`

```text
Unknown subtype of T
    ↓
Producer
    ↓
Safe to read as T
Not safe to add specific values
```

### `? super T`

```text
Unknown supertype of T
    ↓
Consumer
    ↓
Safe to write T
Safe to read only as Object
```

### PECS

```text
Producer → Extends
Consumer → Super
```

---

## Quick Comparison

| Type | Safe read | Safe write |
|---|---|---|
| `List<?>` | `Object` | Essentially nothing except `null` |
| `List<? extends Number>` | `Number` | No specific values |
| `List<? super Integer>` | `Object` | `Integer` |

---

## Important Traps

### Generics are invariant

```text
String extends Object
```

does not imply:

```text
List<String> extends List<Object>
```

### `? extends T` does not mean you can add any subtype of `T`

Even though `Integer extends Number`, you cannot add an `Integer` to `List<? extends Number>` because the actual list could be `List<Double>`.

### `? super T` does not mean you can add arbitrary supertypes of `T`

For:

```java
List<? super Dog>
```

you can safely add `Dog`, but not an arbitrary `Animal`, because the actual list could be `List<Dog>`.

### PECS is from the method's perspective

Ask:

> Is my code getting values from this structure or putting values into it?

Then:

```text
GET / Producer → extends
PUT / Consumer → super
```

### `super` is for wildcards, not type-parameter bounds

```java
<T super Dog>       // ❌ invalid
<? super Dog>       // ✅ valid
```

---

## Interview-Level Takeaways

- `?` represents an unknown type argument.
- `List<?>` can refer to a list of any reference type, but values can only safely be read as `Object` and specific values cannot generally be added.
- Java generic types are invariant by default.
- `? extends T` is an upper-bounded wildcard representing some unknown subtype of `T` (or `T` itself).
- With `? extends T`, values can safely be read as `T`, but specific values cannot safely be added.
- `? super T` is a lower-bounded wildcard representing some unknown supertype of `T`.
- With `? super T`, values of type `T` can safely be added, while values can only safely be read as `Object`.
- **PECS = Producer Extends, Consumer Super.**
- `<T extends X>` declares a named type parameter; `<? extends X>` represents an unknown type argument.
- `<T super X>` is invalid; `super` is used with wildcards.

> **Best mental model:** `extends` gives you something to take out; `super` gives you somewhere safe to put the bounded type in.
