# Inheritance in Java

## Overview

Inheritance is one of the core principles of object-oriented programming.

> **Inheritance allows a class to inherit accessible properties and behavior from another class and extend or specialize that behavior.**

## The IS-A Relationship

Inheritance should represent a genuine **IS-A** relationship.

```text
Dog       IS-A Animal
Car       IS-A Vehicle
Developer IS-A Employee
```

> **Do not use inheritance merely for code reuse.** If the relationship is not genuinely IS-A, composition is often a better choice.

## Single Class Inheritance

Java supports **single inheritance of classes**. A class can directly extend only one class.

```java
class Dog extends Animal {
}
```

A class cannot directly extend multiple classes.

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

Common methods such as `equals()`, `hashCode()`, and `toString()` originate from `Object`.

## What Does the Child Inherit?

A child can use accessible inherited state and behavior from its superclass, while adding its own state and behavior.

Private parent members are not directly accessible from the child.

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

### `@Override`

`@Override` tells the compiler that the programmer intends the method to override a superclass method. The annotation does not make overriding happen; it asks the compiler to verify that an actual override exists.

## Runtime Polymorphism

Inheritance enables runtime polymorphism through overridden instance methods.

```java
Animal animal = new Dog();
animal.speak(); // Dog implementation
```

- **Reference type:** `Animal` — controls which members are available through the reference at compile time.
- **Runtime object type:** `Dog` — determines which overridden instance method implementation executes.

This is **runtime polymorphism** / **dynamic method dispatch**.

## `super` and Inheritance

`super` lets a child explicitly invoke superclass behavior:

```java
class Dog extends Animal {
    @Override
    void speak() {
        System.out.println("Dog");
        super.speak();
    }
}
```

`super.speak()` explicitly invokes the superclass implementation.

## Constructors Are Not Inherited

Constructors are **not inherited**. A child constructor must explicitly or implicitly invoke an appropriate parent constructor.

```java
class Child extends Parent {
    Child() {
        super(10);
    }
}
```

The parent constructor initializes the superclass portion of the object; it is not inherited by the child.

## Private Methods and Overriding

Private methods cannot be overridden because they are not accessible to subclasses. A same-named method in the child is a new method, not an override.

## Static Methods: Hiding, Not Overriding

Static methods belong to the class, so they are not overridden through runtime polymorphism. They are **hidden**.

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

## `final` and Inheritance

- A `final` class cannot be extended.
- A `final` method cannot be overridden.

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

class Dog extends Animal {
    @Override
    void speak() {
        System.out.println("Bark");
    }
}
```

A concrete subclass provides the abstract behavior while inheriting concrete behavior.

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

Relevant relationships:

```text
class      --extends-->    class
class      --implements-> interface
interface  --extends-->    interface
```

## The Diamond Problem

The **Diamond Problem** describes ambiguity that can occur with multiple inheritance when a class inherits the same behavior or state through multiple parent classes.

Imagine Java allowed:

```text
       A
      / \
     B   C
      \ /
       D
```

If both `B` and `C` override `hello()`, then `D.hello()` would have an ambiguous choice:

```text
B.hello() ?
C.hello() ?
```

Java avoids this by **not allowing multiple inheritance of classes**.

### Why Multiple Interfaces Are Allowed

Interfaces were traditionally contracts rather than inherited class state/implementation, so a class can implement multiple interfaces:

```java
interface Flyable {
    void fly();
}

interface Swimmable {
    void swim();
}

class Duck implements Flyable, Swimmable {
    public void fly() {}
    public void swim() {}
}
```

### Default Methods Make Interface Conflicts Possible

Modern interfaces can have `default` methods, so multiple interfaces can provide competing implementations:

```java
interface A {
    default void hello() {
        System.out.println("A");
    }
}

interface B {
    default void hello() {
        System.out.println("B");
    }
}

class C implements A, B {
    // compilation error unless conflict is resolved
}
```

The implementing class must resolve the conflict:

```java
class C implements A, B {
    @Override
    public void hello() {
        System.out.println("C");
    }
}
```

Or explicitly choose one interface's default implementation:

```java
class C implements A, B {
    @Override
    public void hello() {
        A.super.hello();
    }
}
```

### Class Method vs Interface Default Method

If a class inherits an actual method from a superclass and also gets a conflicting `default` method from an interface, the **class method wins**.

```java
class A {
    void hello() {
        System.out.println("A");
    }
}

interface B {
    default void hello() {
        System.out.println("B");
    }
}

class C extends A implements B {
}

C c = new C();
c.hello(); // A
```

### Mental Model

```text
Multiple classes
      ↓
Diamond problem
      ↓
Ambiguous inherited state/behavior
      ↓
Java disallows multiple class inheritance
```

For interfaces:

```text
Multiple interfaces
      ↓
Contracts/capabilities
      ↓
Multiple interfaces allowed
      ↓
Default-method conflict?
      ↓
Implementing class must resolve it
```

## Interface Fields and State

An interface **can declare fields**, but they are not per-object instance state.

Every interface field is implicitly:

```java
public static final
```

For example:

```java
interface Config {
    int MAX_RETRIES = 3;
}
```

is effectively:

```java
interface Config {
    public static final int MAX_RETRIES = 3;
}
```

Therefore `Config.MAX_RETRIES` is a constant belonging to the interface, not a separate instance field inside every implementing object.

### Interface Field Name Conflicts

Two interfaces can declare fields with the same name:

```java
interface A {
    int VALUE = 10;
}

interface B {
    int VALUE = 20;
}

class C implements A, B {
}
```

An unqualified reference through `C` is ambiguous:

```java
System.out.println(C.VALUE); // compilation error
```

Resolve it by qualifying the interface:

```java
System.out.println(A.VALUE); // 10
System.out.println(B.VALUE); // 20
```

Fields are static constants, so this is a **name ambiguity**, not runtime method dispatch.

### Interface Constants Can Refer to Mutable Objects

The field reference itself is still `public static final`, so it cannot be replaced:

```java
interface Config {
    List<String> VALUES = new ArrayList<>();
}

Config.VALUES = new ArrayList<>(); // compilation error
```

But the referenced object can still be mutated:

```java
Config.VALUES.add("hello"); // valid
```

This is the same `final` reference vs mutable object distinction.

## Covariant Return Types

When overriding a method, Java allows the overriding method to return the **same type or a more specific subtype** of the parent's return type.

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

This is valid because `Dog` is a subtype of `Animal`.

An unrelated return type is not allowed.

### Interview Rule

> **An overriding method may return the same type as the parent method or a subtype (more specific type) of the parent's return type.**

## Inheritance vs Composition

Do not use inheritance merely because two classes share code.

```java
class Car extends Engine { // conceptually wrong
}
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

## Inheritance as an OOP Principle

The commonly discussed four OOP principles are:

```text
                 OOP
                  |
       +----------+----------+----------+
       |          |          |          |
 Encapsulation Inheritance Polymorphism Abstraction
```

- **Encapsulation:** hide internal state/implementation details using access control.
- **Inheritance:** extend an existing class through an IS-A relationship.
- **Polymorphism:** allow the same reference/contract to represent different concrete implementations.
- **Abstraction:** expose essential behavior while hiding implementation details through mechanisms such as abstract classes and interfaces.

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

**Example — shared state + shared behavior:**

```java
abstract class Employee {
    protected String name;

    protected Employee(String name) {
        this.name = name;
    }

    public void clockIn() {
        System.out.println(name + " clocked in");
    }

    public abstract double calculateSalary();
}

class Developer extends Employee {
    public Developer(String name) {
        super(name);
    }

    @Override
    public double calculateSalary() {
        return 100000;
    }
}
```

Here, `Employee` is a genuine base type. Developers and other employees share **state (`name`)**, **constructor logic**, and **common behavior (`clockIn`)**, while subclasses provide their own salary calculation.

### Interface

Use when you want to define a contract/capability that potentially unrelated classes can satisfy.

**Example — shared capability, unrelated classes:**

```java
interface Payable {
    void pay();
}

class Employee implements Payable {
    @Override
    public void pay() {
        System.out.println("Paying salary");
    }
}

class Invoice implements Payable {
    @Override
    public void pay() {
        System.out.println("Paying invoice");
    }
}
```

`Employee` and `Invoice` are not naturally in the same class hierarchy, but both can satisfy the **`Payable` capability**.

The key design question is:

> **Do I need a shared base with state/implementation, or do I need a contract/capability that different classes can satisfy?**

A useful mental shortcut:

```text
Abstract class
    ↓
"What are you?"
    ↓
Employee → Developer

Interface
    ↓
"What can you do?"
    ↓
Employee → Payable
Invoice  → Payable
```

A class can also combine both:

```java
class Developer extends Employee implements Payable {
    // shared Employee state/behavior
    // + Payable contract
}
```

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
- The **Diamond Problem** explains why Java does not allow multiple inheritance of classes; multiple inherited implementations/state could create ambiguity.
- Multiple interfaces are allowed, but conflicting `default` methods must be explicitly resolved by the implementing class.
- An inherited class method takes precedence over a conflicting interface `default` method.
- Interface fields exist, but are implicitly `public static final`; they are constants, not per-object instance state.
- If multiple interfaces declare fields with the same name, access is ambiguous and must be qualified (`A.VALUE` vs `B.VALUE`).
- An interface constant can refer to a mutable object; `final` prevents replacing the reference, not mutating the referenced object.
- **Covariant return types** allow an overriding method to return the same type or a more specific subtype of the parent's return type.
- Inheritance is not simply a code-reuse mechanism; use composition for **HAS-A** relationships.
- Inheritance is one of the four commonly discussed OOP principles, alongside encapsulation, polymorphism, and abstraction.
