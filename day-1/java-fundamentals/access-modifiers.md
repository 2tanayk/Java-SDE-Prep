# Access Modifiers in Java

## Overview

Access modifiers control **where a class, field, method, or constructor can be accessed from**.

Java has four access levels:

- `private`
- **package-private** — no modifier/keyword
- `protected`
- `public`

Conceptually, from most restrictive to least restrictive:

```text
private
   ↓
package-private
   ↓
protected
   ↓
public
```

## `private`

- `private` is the most restrictive access level.
- A private member can only be accessed inside the class where it is declared.

```java
class Person {

    private String name;

    private void validateName() {
        // ...
    }
}
```

Inside `Person`, both members are accessible:

```java
void print() {
    System.out.println(name); // valid
    validateName();           // valid
}
```

Outside the class:

```java
Person person = new Person();

person.name;           // compilation error
person.validateName(); // compilation error
```

### Rule

> **`private` → declaring class only.**

## Package-Private

- Package-private access is specified by using **no access modifier at all**.

```java
class Person {

    String name;

    void printName() {
    }
}
```

- `name` and `printName()` are package-private.
- Package-private members can be accessed by classes in the **same package**.
- They cannot be accessed by classes in a different package.

Example:

```text
com.example.user
├── Person.java
└── UserService.java
```

`UserService` can access package-private members of `Person`.

But:

```text
com.example.user
    Person

com.example.service
    UserService
```

`UserService` cannot access those package-private members because it belongs to a different package.

### Rule

> **No modifier → package-private → same package only.**

## `public`

- `public` is the least restrictive access level.
- A public member can be accessed from anywhere where its containing type is accessible.

```java
public class Person {

    public String name;

    public void printName() {
    }
}
```

Other packages can access the public members:

```java
Person person = new Person();

person.name;        // valid
person.printName(); // valid
```

### Rule

> **`public` → accessible from anywhere where the type is accessible.**

## `protected`

`protected` provides access to:

- Classes in the **same package**.
- Subclasses, including subclasses in a **different package**, subject to an important cross-package restriction.

Example superclass:

```java
package animals;

public class Animal {
    protected String name;
}
```

A class in the same package can access it:

```java
package animals;

class AnimalService {

    void print(Animal animal) {
        System.out.println(animal.name); // valid
    }
}
```

A subclass in another package can access the inherited protected member through the subclass context:

```java
package dogs;

import animals.Animal;

class Dog extends Animal {

    void printName() {
        System.out.println(name); // valid
    }
}
```

## The `protected` Cross-Package Trap

This is a common interview trap.

Suppose:

```java
package animals;

public class Animal {
    protected String name;
}
```

and a subclass exists in another package:

```java
package dogs;

public class Dog extends Animal {

    void test(Animal animal) {
        System.out.println(animal.name); // compilation error
    }
}
```

This is **not** allowed merely because `Dog` is a subclass of `Animal`.

However, access through the subclass/inherited context is allowed:

```java
class Dog extends Animal {

    void test() {
        System.out.println(this.name); // valid
    }
}
```

and:

```java
class Dog extends Animal {

    void test(Dog dog) {
        System.out.println(dog.name); // valid
    }
}
```

The useful mental model is:

> **For cross-package subclass access, `protected` gives the subclass access to the inherited member through the subclass context; it does not provide unrestricted access through an arbitrary superclass reference.**

## Access Modifier Comparison

| Modifier | Same class | Same package | Subclass, different package | Other package |
|---|---:|---:|---:|---:|
| `private` | ✅ | ❌ | ❌ | ❌ |
| package-private | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅* | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

`*` Cross-package `protected` access is subject to the subclass/inheritance-context restriction described above.

## Access Modifiers on Top-Level Classes

A **top-level class** can be:

```java
public class Person {
}
```

or package-private:

```java
class Person {
}
```

A top-level class cannot be declared:

```java
private class Person { }   // invalid
protected class Person { } // invalid
```

For the current level of preparation, remember:

> **Top-level classes → `public` or package-private.**

Nested classes have additional possibilities, but those are outside this topic.

## Access Modifiers and Encapsulation

Access modifiers are an important mechanism for **encapsulation** and API design.

For example:

```java
public class PaymentService {

    public void processPayment() {
        validatePayment();
    }

    private void validatePayment() {
        // internal implementation
    }
}
```

The intended public API is:

```text
PaymentService
      |
      └── public processPayment()
```

while implementation details remain hidden:

```text
private validatePayment()
```

A useful design principle is:

> **Expose the smallest API that callers actually need and keep implementation details as restricted as practical.**

## Access vs Inheritance

Do not confuse:

> **Can this member be inherited?**

with:

> **Can this member be accessed directly?**

For example:

```java
class Parent {
    private int x;
}

class Child extends Parent {

    void test() {
        System.out.println(x); // compilation error
    }
}
```

Even though `Child` extends `Parent`, it cannot directly access `Parent.x` because `x` is private to `Parent`.

Controlled access can instead be provided through a method:

```java
class Parent {

    private int x;

    protected int getX() {
        return x;
    }
}
```

Then:

```java
class Child extends Parent {

    void test() {
        System.out.println(getX()); // valid
    }
}
```

## Interview-Level Takeaways

- Java has four access levels: `private`, package-private, `protected`, and `public`.
- Package-private means **no access modifier is specified**.
- `private` → accessible only inside the declaring class.
- Package-private → accessible within the same package.
- `protected` → accessible within the same package and by subclasses, with a special restriction for cross-package access.
- `public` → accessible from anywhere where the containing type is accessible.
- The cross-package `protected` rule is a common interview trap.
- Top-level classes can only be `public` or package-private.
- Access modifiers are a major tool for **encapsulation and API design**.
- `private` members are not directly accessible from subclasses merely because the subclass inherits from the parent.
