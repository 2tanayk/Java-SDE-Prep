# Inheritance in Java

## Overview

Inheritance is one of the core principles of object-oriented programming.

> **Inheritance allows a class to inherit accessible properties and behavior from another class and extend or specialize that behavior.**

```java
class Animal {
    void eat() {
        System.out.println("Eating");
    }
}

class Dog extends Animal {
    void bark() {
        System.out.println("Barking");
    }
}
```

Now:

```java
Dog dog = new Dog();

dog.eat();  // inherited from Animal
dog.bark(); // defined by Dog
```

## The IS-A Relationship

Inheritance should represent a genuine **IS-A** relationship.

```text
Dog       IS-A Animal
Car       IS-A Vehicle
Developer IS-A Employee
```

> **Do not use inheritance merely for code reuse.** If the relationship is not genuinely IS-A, composition is often a better choice.

## What Does the Child Inherit?

A child can use accessible inherited state and behavior from its superclass, while adding its own state and behavior.

```java
class Parent {
    int x;

    void hello() {
    }
}

class Child extends Parent {
    int y;

    void world() {
    }
}
```

Conceptually:

```text
Child object
+-----------------+
| Parent state    |
| x               |
+-----------------+
| Child state     |
| y               |
+-----------------+
```

Private parent members are not directly accessible from the child:

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

## Single Class Inheritance

Java supports **single inheritance of classes**.

```java
class Dog extends Animal {
}
```

A class cannot directly extend multiple classes:

```java
class Dog extends Animal, LivingThing { // compilation error
}
```

However, a class can implement multiple interfaces:

```java
class Duck implements Flyable, Swimmable {
}
```

## Every Class Ultimately Inherits from `Object`

For:

```java
class Person {
}
```

Java conceptually has:

```text
Object
  |
  v
Person
```

Every Java class ultimately derives from `Object` (except `Object` itself).

Common methods such as these originate from `Object`:

```java
equals()
hashCode()
toString()
```

This connects directly to the `equals()` / `hashCode()` topic.

## Method Overriding

A subclass can provide its own implementation of an inherited instance method.

```java
class Animal {
    void speak() {
        System.out.println("Animal sound");
    }
}

class Dog extends Animal {
    @Override
    void speak() {
        System.out.println("Bark");
    }
}
```

This is **method overriding**.

## `@Override`

`@Override` tells the compiler that the programmer intends the method to override a superclass method.

```java
@Override
void speak() {
    System.out.println("Bark");
}
```

The annotation does not make overriding happen; it asks the compiler to verify that an actual override exists.

For example, a typo is caught:

```java
@Override
void speek() { // compilation error
}
```

Without `@Override`, an accidental signature mismatch could create a new method instead of overriding the intended one.

## Runtime Polymorphism

Inheritance enables runtime polymorphism through overridden instance methods.

```java
class Animal {
    void speak() {
        System.out.println("Animal");
    }
}

class Dog extends Animal {
    @Override
    void speak() {
        System.out.println("Dog");
    }
}
```

Then:

```java
Animal animal = new Dog();
animal.speak();
```

executes:

```text
Dog.speak()
```

not `Animal.speak()`.

- Reference type: `Animal`.
- Runtime object type: `Dog`.
- Overridden instance-method dispatch uses the runtime object's implementation.

This is **runtime polymorphism** / **dynamic method dispatch**.

## Reference Type vs Actual Object Type

Given:

```java
Animal animal = new Dog();
```

The reference type controls which members are available through the reference at compile time:

```java
animal.speak(); // valid
animal.bark();  // compilation error if bark() exists only on Dog
```

The runtime object determines which overridden instance method implementation executes.

## Parent Reference → Child Object

```java
Animal animal = new Dog();
```

This is valid because:

```text
Dog IS-A Animal
```

Downcasting is possible when valid:

```java
Dog dog = (Dog) animal;
dog.bark();
```

Detailed casting rules are covered separately.

## `super` and Inheritance

`super` lets a child explicitly invoke superclass behavior:

```java
class Animal {
    void speak() {
        System.out.println("Animal");
    }
}

class Dog extends Animal {
    @Override
    void speak() {
        System.out.println("Dog");
        super.speak();
    }
}
```

Calling `dog.speak()` produces:

```text
Dog
Animal
```

`super.speak()` explicitly invokes the superclass implementation.

## Constructors Are Not Inherited

Constructors are **not inherited**.

```java
class Parent {
    Parent(int x) {
    }
}

class Child extends Parent {
    Child() {
        super(10);
    }
}
```

The child constructor must explicitly or implicitly invoke an appropriate parent constructor.

```text
Child constructor
      |
      v
super(...)
      |
      v
Parent constructor
```

The parent constructor initializes the superclass portion of the object; it is not inherited by the child.

## Private Methods and Overriding

Private methods cannot be overridden because they are not accessible to subclasses.

```java
class Parent {
    private void hello() {
    }
}

class Child extends Parent {
    private void hello() {
    }
}
```

The `hello()` in `Child` is a **new method**, not an override of `Parent.hello()`.

## Static Methods: Hiding, Not Overriding

Static methods belong to the class, so they are not overridden through runtime polymorphism.

```java
class Parent {
    static void hello() {
        System.out.println("Parent");
    }
}

class Child extends Parent {
    static void hello() {
        System.out.println("Child");
    }
}

Parent p = new Child();
p.hello(); // Parent
```

Static methods are **hidden**, rather than overridden.

```text
Instance method
    ↓
overriding
    ↓
runtime dispatch

Static method
    ↓
method hiding
    ↓
reference/class type
```

## `final` and Inheritance

A `final` class cannot be extended:

```java
final class Animal {
}

class Dog extends Animal { // compilation error
}
```

A `final` method cannot be overridden:

```java
class Animal {
    final void breathe() {
    }
}
```

Therefore:

```text
final class  → cannot be extended
final method → cannot be overridden
```

## Abstract Classes and Inheritance

Abstract classes are designed to be used as base classes.

```java
abstract class Animal {
    abstract void speak();

    void eat() {
        System.out.println("Eating");
    }
}
```

A concrete subclass provides the abstract behavior while inheriting concrete behavior:

```java
class Dog extends Animal {
    @Override
    void speak() {
        System.out.println("Bark");
    }
}
```

## Interface Inheritance

Interfaces can form inheritance hierarchies too:

```java
interface Animal {
    void eat();
}

interface Mammal extends Animal {
    void giveBirth();
}
```

A class implementing `Mammal` must satisfy both contracts:

```java
class Dog implements Mammal {
    public void eat() {
    }

    public void giveBirth() {
    }
}
```

Relevant relationships:

```text
class      --extends-->    class
class      --implements-> interface
interface  --extends-->    interface
```

## Covariant Return Types

When overriding a method, Java allows the overriding method to return the **same type or a more specific subtype** of the parent's return type.

This is called a **covariant return type**.

Example:

```java
class Animal {
    Animal create() {
        return new Animal();
    }
}

class Dog extends Animal {
    @Override
    Dog create() {
        return new Dog();
    }
}
```

The parent returns `Animal`, while the child returns `Dog`.

This is valid because:

```text
Dog IS-A Animal
```

An unrelated return type is not allowed:

```java
class Dog extends Animal {
    @Override
    String create() { // compilation error
        return "Dog";
    }
}
```

`String` is not a subtype of `Animal`.

### Why Is This Useful?

Covariant returns allow an overriding method to provide a more specific result without forcing callers using the child type to cast it.

```java
Dog dog = new Dog();
Dog created = dog.create();
```

### Interview Rule

> **An overriding method may return the same type as the parent method or a subtype (more specific type) of the parent's return type.**

## Inheritance vs Composition

Do not use inheritance merely because two classes share code.

This is conceptually wrong:

```java
class Car extends Engine {
}
```

because:

```text
Car IS NOT-A Engine
```

A car **has an engine**, so composition is more appropriate:

```java
class Car {
    private Engine engine;
}
```

Therefore:

```text
Inheritance:
Dog IS-A Animal

Composition:
Car HAS-A Engine
```

Composition is often preferable when the relationship is not genuinely IS-A.

## Inheritance as an OOP Principle

The commonly discussed four OOP principles are:

```text
                 OOP
                  |
       +----------+----------+----------+
       |          |          |          |
 Encapsulation Inheritance Polymorphism Abstraction
```

### Encapsulation

Hide internal state and implementation details using mechanisms such as access modifiers.

### Inheritance

Extend an existing class through an IS-A relationship:

```java
class Dog extends Animal
```

### Polymorphism

Allow the same reference/contract to represent different concrete implementations:

```java
Animal animal = new Dog();
animal.speak(); // Dog implementation
```

### Abstraction

Expose essential behavior while hiding implementation details using mechanisms such as:

```java
abstract class Animal
```

or:

```java
interface Payment
```

A useful relationship is:

```text
Inheritance
    ↓
extends
    ↓
Method overriding
    ↓
Runtime polymorphism
```

## Relationship with Abstract Classes and Interfaces

### Abstract class

Use when you have a genuine base-class relationship and want to share:

- State.
- Constructors.
- Common implementation.
- Common behavior.
- Abstract behavior that subclasses must provide.

Example:

```text
Employee
   |
   +-- Developer
   +-- Manager
   +-- Tester
```

### Interface

Use when you want to define a contract/capability that potentially unrelated classes can satisfy.

Example:

```text
Payable
   |
   +-- UpiPayment
   +-- CreditCardPayment
   +-- Invoice
```

The key design question is:

> **Do I need a shared base with state/implementation, or do I need a contract/capability that different classes can satisfy?**

## Interview-Level Takeaways

- Inheritance allows a class to inherit accessible behavior/state from a superclass and extend it.
- Inheritance should model a genuine **IS-A** relationship.
- Java supports **single inheritance of classes**.
- A class can implement multiple interfaces.
- Every Java class ultimately derives from `Object`.
- Method overriding allows a subclass to provide its own implementation of an inherited instance method.
- `@Override` asks the compiler to verify that a method actually overrides a superclass method.
- Runtime polymorphism/dynamic method dispatch selects the overridden instance method based on the runtime object.
- In `Animal animal = new Dog()`, the reference type controls compile-time member availability, while the runtime object determines overridden instance-method behavior.
- Constructors are **not inherited**.
- `super(...)` invokes a superclass constructor.
- `super.method()` invokes the superclass implementation.
- Private methods cannot be overridden.
- Static methods are **hidden**, not overridden.
- A `final` class cannot be extended.
- A `final` method cannot be overridden.
- Abstract classes are useful as shared base classes.
- Interfaces can also form inheritance hierarchies.
- **Covariant return types** allow an overriding method to return the same type or a more specific subtype of the parent's return type.
- Inheritance is not simply a code-reuse mechanism; use composition for **HAS-A** relationships.
- Inheritance is one of the four commonly discussed OOP principles, alongside encapsulation, polymorphism, and abstraction.
