# `final` in Java

## Overview

- `final` prevents something from being changed further in a specific way.
- Its exact meaning depends on where it is used:
  - `final` variable → cannot be reassigned.
  - `final` method → cannot be overridden.
  - `final` class → cannot be extended.

```text
                     final
                       |
          +------------+------------+
          |            |            |
       variable      method       class
          |            |            |
          v            v            v
   cannot be       cannot be     cannot be
   reassigned      overridden     extended
```

## `final` Variables

A `final` variable cannot be assigned a new value after it has been assigned.

```java
final int x = 10;
x = 20; // compilation error
```

Without `final`:

```java
int x = 10;
x = 20; // valid
```

### Blank Final Variables

A `final` variable does not necessarily have to be initialized at the point of declaration.

```java
final int x;
x = 10;
```

This is valid because `x` is assigned once.

A second assignment is not allowed:

```java
x = 20; // compilation error
```

A `final` variable should therefore be thought of as a variable that can be assigned only once.

## `final` Object References

`final` behaves differently when the variable is a reference to an object.

```java
final List<String> names = new ArrayList<>();
```

The reference cannot be changed to point to another object:

```java
names = new ArrayList<>(); // compilation error
```

But the referenced object may still be mutable:

```java
names.add("Tanay"); // valid
names.add("Rahul"); // valid
```

Conceptually:

```text
names
  |
  | final reference
  v
+----------------+
| ArrayList      |
| "Tanay"        |
| "Rahul"        |
+----------------+
```

- `final` prevents the **reference from being reassigned**.
- It does **not** automatically make the referenced object immutable.

### Key Interview Point

> **A `final` reference does not mean the referenced object is immutable.**

## `final` Methods

A `final` method cannot be overridden by a subclass.

Without `final`:

```java
class Parent {
    void hello() {
        System.out.println("Parent");
    }
}

class Child extends Parent {
    @Override
    void hello() {
        System.out.println("Child");
    }
}
```

With `final`:

```java
class Parent {
    final void hello() {
        System.out.println("Parent");
    }
}

class Child extends Parent {
    @Override
    void hello() { // compilation error
    }
}
```

- A subclass can inherit a `final` method.
- It cannot replace/override that method's implementation.

### Why Use a `final` Method?

- Use it when a particular behavior must not be changed by subclasses.
- A class can still be extended while specific methods remain protected from overriding.

Example:

```java
class PaymentProcessor {
    final void validatePayment() {
        // critical validation
    }
}
```

A subclass can add other behavior but cannot override `validatePayment()`.

## `final` Classes

A `final` class cannot be extended.

```java
final class PaymentProcessor {
}
```

This is illegal:

```java
class StripePaymentProcessor extends PaymentProcessor {
    // compilation error
}
```

The class is effectively saying:

> No class may subclass me.

A familiar Java example is `String`:

```java
public final class String
```

Therefore, you cannot create a subclass of `String`.

## `static final`

`static` and `final` solve different problems.

- `static` → belongs to the class.
- `final` → cannot be reassigned.

Together:

```java
static final int MAX_RETRIES = 3;
```

This is commonly used for constants.

Example:

```java
class Config {
    static final int MAX_RETRIES = 3;
    static final String DEFAULT_ROLE = "USER";
}
```

Usage:

```java
Config.MAX_RETRIES
```

Constants are conventionally named using `UPPER_SNAKE_CASE`.

## `final` vs Immutability

`final` does **not** automatically make an object immutable.

For example:

```java
final List<String> names = new ArrayList<>();
```

The following is still valid:

```java
names.add("Tanay");
names.clear();
```

What is protected is the reference:

```text
names ───────────────> ArrayList
  ^
  |
 final
```

You cannot do:

```java
names = anotherList; // compilation error
```

Therefore:

> **`final` guarantees that a variable/reference cannot be reassigned. It does not automatically make the referenced object immutable.**

Immutability is a separate concept and will be covered separately.

## Interview-Level Takeaways

- `final` variable → cannot be reassigned after assignment.
- A blank `final` variable can be initialized later, as long as it is assigned only once according to Java's definite-assignment rules.
- `final` reference → cannot point to another object, but the referenced object may still be mutable.
- `final` method → cannot be overridden by a subclass.
- `final` class → cannot be extended.
- `static final` → commonly used for constants.
- `final` does **not** automatically mean immutable.
