# Generic Bounds

Generic bounds allow us to **restrict the types that a generic type parameter can represent**.

An unbounded type parameter:

```java
<T>
```

means that `T` can represent any reference type.

A bounded type parameter such as:

```java
<T extends Number>
```

means that `T` must be compatible with `Number` or one of its subclasses.

---

## 1. Upper Bounds — `extends`

Suppose we only want a generic method to accept `Number` types:

```java
public static <T extends Number> void printNumber(T value) {
    System.out.println(value);
}
```

Valid examples include:

```java
printNumber(10);       // Integer
printNumber(10.5);     // Double
printNumber(100L);     // Long
```

but:

```java
printNumber("Tanay"); // ❌ String is not a Number
```

Therefore:

```java
<T extends Number>
```

means:

> `T` must be `Number` or a subtype of `Number`.

This is called an **upper bound**.

---

## 2. Why Are Bounds Useful?

A bound does more than restrict which types are accepted. It also tells the compiler what capabilities are guaranteed to exist on `T`.

Without a bound:

```java
public static <T> void calculate(T value) {
    // T could be almost anything
}
```

Java cannot assume that `T` has methods specific to `Number`.

With a bound:

```java
public static <T extends Number> void calculate(T value) {
    double result = value.doubleValue();
}
```

Java knows that every valid `T` is a `Number`, so `Number` methods such as these are available:

```java
value.doubleValue();
value.intValue();
value.longValue();
```

The important mental model is:

```text
<T>
   ↓
compiler knows very little about T

<T extends Number>
   ↓
compiler knows T is a Number
   ↓
Number's guaranteed members are available
```

So bounds serve two purposes:

1. **Restrict valid types.**
2. **Give the compiler guaranteed capabilities on those types.**

---

## 3. Bounds on Generic Classes

A generic class can also have a bound:

```java
class Box<T extends Number> {

    private T value;

    public void set(T value) {
        this.value = value;
    }

    public double getAsDouble() {
        return value.doubleValue();
    }
}
```

Valid usages:

```java
Box<Integer> integers = new Box<>();
Box<Double> doubles = new Box<>();
```

Invalid:

```java
Box<String> strings = new Box<>(); // ❌
```

The class can safely use functionality guaranteed by the bound.

---

## 4. Bounds Can Use Interfaces

`extends` is also used when the bound is an interface.

For example:

```java
<T extends Comparable<T>>
```

This means `T` must implement `Comparable<T>`.

Example:

```java
public static <T extends Comparable<T>> T max(T a, T b) {
    return a.compareTo(b) > 0 ? a : b;
}
```

Now the method can safely call:

```java
a.compareTo(b)
```

because the bound guarantees that `T` provides `compareTo()`.

For example, both `Integer` and `String` implement `Comparable`, so they can be used with this method:

```java
max(10, 20);
max("Tanay", "Rahul");
```

### Important Terminology

In generic bounds, `extends` is used for both classes and interfaces:

```java
<T extends SomeClass>
<T extends SomeInterface>
```

For an interface, interpret the bound as:

> `T` must implement that interface.

Do not over-interpret the keyword as meaning only Java class inheritance. The generic-bound syntax uses `extends` to express the upper bound.

---

## 5. Multiple Bounds

A type parameter can have multiple bounds:

```java
<T extends Number & Comparable<T>>
```

This means `T` must:

```text
extend Number
AND
implement Comparable<T>
```

More generally:

```java
<T extends Class & Interface1 & Interface2>
```

If a class bound is present, it must appear first:

```java
<T extends Number & Serializable & Comparable<T>> // ✅
```

whereas:

```java
<T extends Serializable & Number> // ❌
```

is invalid.

Multiple bounds are less common in everyday backend code, but the syntax and concept are worth recognizing.

---

## 6. The Bound Does Not Mean Only the Exact Type

For:

```java
<T extends Number>
```

`T` can be `Number` itself or a subtype:

```text
Number
 ├── Integer
 ├── Double
 ├── Long
 └── Float
```

So all of these can satisfy the bound:

```java
Box<Number>
Box<Integer>
Box<Double>
Box<Long>
```

The bound describes the **upper limit** of the type parameter's allowed type family.

---

## 7. Do Not Confuse Type Parameter Bounds With Wildcards

This distinction is important and should be kept separate from the wildcard topic.

A **type parameter bound** looks like:

```java
<T extends Number>
```

It introduces a named type parameter `T`.

For example:

```java
public <T extends Number> T process(T value) {
    // T can be used throughout this method
    return value;
}
```

A wildcard looks like:

```java
<? extends Number>
```

and represents an unknown type satisfying that bound.

Similarly, `super` belongs to wildcard syntax:

```java
<? super Integer> // ✅
```

but this is **not valid** generic type-parameter syntax:

```java
<T super Integer> // ❌
```

So remember:

```text
Generic type parameter:
<T extends X>       ✅
<T super X>         ❌

Wildcard:
<? extends X>       ✅
<? super X>         ✅
```

We will cover `?`, `? extends`, and `? super` separately when we study **wildcards**. Do not mix wildcard rules into generic type-parameter bounds.

---

## Core Mental Model

Start with:

```java
<T>
```

meaning:

> `T` can represent any reference type.

Then:

```java
<T extends Number>
```

meaning:

> `T` can represent only a type compatible with `Number`.

The key consequence is:

```text
<T>
   ↓
unknown type

<T extends Number>
   ↓
known to be a Number
   ↓
Number's guaranteed functionality can be used
```

---

## Interview-Level Takeaways

- A generic bound restricts the types a type parameter can represent.
- `<T extends Number>` is an **upper-bounded type parameter**.
- `extends` can be used with both classes and interfaces in generic bounds.
- Bounds let the compiler enforce valid type arguments.
- Bounds also let generic code safely use members guaranteed by the bound.
- Multiple bounds are possible using `&`.
- If a class bound exists, it must appear before interface bounds.
- `<T super X>` does **not** exist.
- `super` is used with wildcards (`<? super X>`), which is a separate topic.
- Do not confuse `<T extends X>` with `<? extends X>`; the former declares a named type parameter, while the latter is wildcard syntax.

> **Core idea:** A generic bound says, "I don't know the exact type, but I guarantee that it satisfies this upper-bound contract." 
