# Immutability in Java

## Overview

> **An immutable object is an object whose observable state cannot change after construction.**

For example, `String` is immutable.

```java
String s = "hello";
s = s.concat(" world");
```

`concat()` does not modify the original `"hello"` object. It creates another `String`, and the variable `s` is reassigned to refer to it.

```text
Original object
"hello"
    ↓
    unchanged

New object
"hello world"
```

## `final` Does NOT Mean Immutable

This is one of the most important distinctions in Java.

```java
final List<String> names = new ArrayList<>();
```

This is not allowed:

```java
names = new ArrayList<>(); // compilation error
```

because `final` prevents the reference from being reassigned.

But this is allowed:

```java
names.add("Tanay"); // valid
```

because the referenced `ArrayList` is still mutable.

```text
final reference
      ↓
cannot point to another object

referenced object
      ↓
may still be mutable
```

Therefore:

> **`final` makes a reference/field non-reassignable. It does not make the referenced object immutable.**

## Example of a Mutable Class with `final` Fields

```java
class Person {

    private final List<String> hobbies;

    Person(List<String> hobbies) {
        this.hobbies = hobbies;
    }

    public List<String> getHobbies() {
        return hobbies;
    }
}
```

This class is **not immutable**.

The caller can do:

```java
Person person = new Person(new ArrayList<>());
person.getHobbies().add("Coding");
```

The internal state changes even though the field is `final`.

## Typical Requirements for an Immutable Class

A typical immutable class should:

- Prevent subclass-based modification when appropriate, commonly by making the class `final`.
- Make fields `private`.
- Make fields `final`.
- Initialize state through the constructor.
- Avoid setters/mutators that change state.
- Defensively copy mutable input objects.
- Never expose mutable internal state in a way that allows callers to modify it.
- Prefer immutable field types.
- Consider deep immutability when fields refer to other objects.

## Defensive Copying

Suppose:

```java
public final class Person {

    private final List<String> hobbies;

    public Person(List<String> hobbies) {
        this.hobbies = hobbies;
    }
}
```

This is unsafe because the caller may retain the original list:

```java
List<String> hobbies = new ArrayList<>();
hobbies.add("Coding");

Person person = new Person(hobbies);

hobbies.add("Gaming");
```

Now the internal state of `person` has also changed because both references point to the same mutable list.

```text
caller list
    ↓
ArrayList A
    ↑
    |
person.hobbies
```

### Fix: Copy the Input

```java
public Person(List<String> hobbies) {
    this.hobbies = new ArrayList<>(hobbies);
}
```

Now:

```text
caller list
    ↓
ArrayList A

constructor copies
    ↓
ArrayList B
    ↑
    |
person.hobbies
```

Mutating the caller's original list no longer changes the `Person` object's internal list.

### Prefer `List.copyOf()` When Appropriate

```java
public Person(List<String> hobbies) {
    this.hobbies = List.copyOf(hobbies);
}
```

`List.copyOf()` gives the class an unmodifiable list and also prevents the caller's mutable list from becoming the internal representation.

## Exposing Internal Collections Safely

Consider:

```java
public final class Person {

    private final List<String> hobbies;

    public Person(List<String> hobbies) {
        this.hobbies = List.copyOf(hobbies);
    }

    public List<String> getHobbies() {
        return hobbies;
    }
}
```

It is **safe to return `hobbies` directly in this specific implementation** because `List.copyOf()` stored an unmodifiable list internally.

The important rule is not:

> "Never return the internal reference."

The correct rule is:

> **Never expose mutable internal state in a way that allows the caller to modify the object.**

Because `hobbies` is already unmodifiable, this is safe:

```java
person.getHobbies().add("Gaming"); // UnsupportedOperationException
```

Alternatively, this is also safe:

```java
public List<String> getHobbies() {
    return List.copyOf(hobbies);
}
```

although making another copy may be unnecessary when the internal list is already safely unmodifiable.

### Unsafe Version

This is unsafe:

```java
public Person(List<String> hobbies) {
    this.hobbies = hobbies;
}

public List<String> getHobbies() {
    return hobbies;
}
```

because:

```java
person.getHobbies().add("Gaming");
```

can mutate the internal state.

## Complete Immutable Example

```java
public final class Person {

    private final String name;
    private final List<String> hobbies;

    public Person(String name, List<String> hobbies) {
        this.name = name;
        this.hobbies = List.copyOf(hobbies);
    }

    public String getName() {
        return name;
    }

    public List<String> getHobbies() {
        return hobbies;
    }
}
```

The state cannot be modified through the exposed API.

## Deep Immutability

A `final` field can still point to a mutable object.

For example:

```java
class Address {
    String city;

    Address(String city) {
        this.city = city;
    }
}

public final class Person {

    private final Address address;

    public Person(Address address) {
        this.address = address;
    }

    public Address getAddress() {
        return address;
    }
}
```

`Person` is not truly immutable because the `Address` object can still be modified:

```java
person.getAddress().city = "Mumbai";
```

Therefore:

> **For deep immutability, objects reachable through an immutable object's public API must also be immutable or otherwise safely protected from mutation.**

This is why immutable field types are preferable.

## Common Immutable Java Types

Examples include:

```text
String
Integer
Long
BigDecimal
LocalDate
LocalDateTime
```

Using immutable objects as fields makes immutable class design easier.

## Benefits of Immutability

### Thread Safety

Immutable objects are naturally easier to share between threads because their state cannot be changed after construction.

```text
Thread A ──┐
           ↓
     immutable object
           ↑
Thread B ──┘
```

Neither thread can mutate the object's state.

> Immutability does not solve every concurrency problem, but it removes a major category of shared-state mutation concerns.

### Predictability

Once constructed, the object's state does not unexpectedly change elsewhere in the program.

### Safe Sharing

Immutable objects can be passed between components without worrying that another component will mutate them.

### Safe Map Keys

Objects used as `HashMap` keys should not have their `equals()`/`hashCode()`-relevant state changed while they are stored in the map.

If a key's state changes after insertion, its hash code may change and the map may no longer find the entry in the expected bucket.

Immutable objects are therefore excellent candidates for map keys.

`String` is a classic example.

## Immutability and the String Pool

Because `String` objects are immutable, Java can safely share identical interned strings.

```java
String a = "hello";
String b = "hello";
```

They may refer to the same interned string object:

```text
a ──────┐
        ↓
     "hello"
        ↑
b ──────┘
```

If strings were mutable, changing the shared object through one reference could unexpectedly affect the other reference. Immutability makes this sharing safe.

## Methods on Immutable Objects Return New Objects

Consider:

```java
String s = "hello";
s = s.toUpperCase();
```

`toUpperCase()` does not modify the original string. It returns another `String`, and the reference `s` is reassigned.

The same pattern appears with Java date/time types:

```java
LocalDate date = LocalDate.of(2026, 8, 17);
date = date.plusDays(1);
```

`plusDays()` returns another `LocalDate`; it does not mutate the existing object.

## Immutability vs Encapsulation

These concepts are related but different.

### Encapsulation

Controls access to state.

An object can be encapsulated and mutable:

```java
class BankAccount {
    private double balance;

    public void deposit(double amount) {
        balance += amount;
    }
}
```

The state is hidden but can change.

### Immutability

Prevents the object's state from changing after construction.

Therefore:

> **Encapsulation controls access; immutability controls whether state can change.**

## Immutability vs `final`

Another common interview question:

> **Does `final` make an object immutable?**

**No.**

```java
final List<String> list = new ArrayList<>();

list = new ArrayList<>(); // compilation error
list.add("hello");        // valid
```

The distinction is:

```text
final
 ↓
reference cannot be reassigned

immutability
 ↓
object's observable state cannot change
```

These are different guarantees.

## Immutable Class Checklist

```text
        Immutable Class
              |
      +-------+-------+
      |               |
   Structure        Behavior
      |               |
   final class     no setters
      |
 private final fields
      |
 constructor initialization
      |
 defensive copies
      |
 no mutable state leaked
      |
 immutable/deeply protected fields
```

For a typical immutable class:

- Make the class `final` when appropriate.
- Make fields `private final`.
- Initialize all state during construction.
- Do not provide mutators/setters.
- Defensively copy mutable inputs.
- Do not expose mutable internal state.
- Returning an internal reference is fine **if that referenced object is already immutable/unmodifiable**.
- Prefer immutable field types.
- Consider deep immutability for nested objects.

## Interview-Level Takeaways

- **Immutable object → observable state cannot change after construction.**
- `final` does **not** mean immutable.
- `final` prevents a reference/field from being reassigned; it does not necessarily prevent mutation of the referenced object.
- `String` is immutable.
- Immutable objects are easier to share safely between threads.
- Immutability makes objects predictable and safe to share.
- Be careful when using mutable objects as `HashMap` keys.
- Defensive copying is important when immutable objects contain mutable objects.
- Do not leak mutable internal state through getters.
- Returning an internal reference is **not inherently wrong**; it is safe when the referenced object is already immutable/unmodifiable.
- `List.copyOf()` is useful for creating an unmodifiable internal list from caller-provided data.
- Think about **deep immutability**, not just `private final` fields.
- **Encapsulation ≠ immutability:** encapsulation controls access; immutability prevents state changes.

> **Key sentence:** An immutable object is an object whose observable state cannot change after construction; `final` helps enforce this for references, but by itself does not make the referenced objects immutable.
