# Abstraction in Java

## Overview

> **Abstraction means exposing only the essential behavior of an object while hiding unnecessary implementation details.**

The key question abstraction answers is:

> **What does the caller actually need to know, and what implementation details can be hidden?**

For example, a payment system may expose:

```java
payment.pay(1000);
```

without requiring the caller to know the details of UPI validation, bank communication, transaction handling, card networks, etc.

## Abstraction vs Encapsulation

These concepts are related but solve different problems.

### Encapsulation

> **How do I protect/control the object's internal state and implementation?**

Typical mechanisms:

```text
private
access modifiers
controlled mutation
```

### Abstraction

> **What should the outside world actually need to know?**

Typical Java mechanisms:

```text
interfaces
abstract classes
abstract methods
```

Useful distinction:

```text
Encapsulation
→ hides/protects internal details

Abstraction
→ hides unnecessary complexity by exposing only essential behavior
```

> **Encapsulation protects the internals; abstraction exposes only what is necessary to use the object.**

## Real-World Analogy

A car exposes:

```text
steering wheel
accelerator
brake
gear
```

The driver does not need to understand:

```text
fuel injection
engine timing
combustion
transmission internals
```

The controls form an abstraction over the complicated implementation underneath.

## Abstraction in Java

Java primarily provides two mechanisms for abstraction:

```text
                Abstraction
                     |
             +-------+-------+
             |               |
      Abstract class      Interface
```

## Abstract Classes

An abstract class is a class intended to act as a base class and **cannot itself be instantiated**.

```java
abstract class Animal {

    abstract void speak();

    void eat() {
        System.out.println("Eating");
    }
}
```

An abstract class can contain both:

- Abstract methods.
- Concrete methods.
- State.
- Constructors.

A concrete subclass provides the missing abstract behavior:

```java
class Dog extends Animal {

    @Override
    void speak() {
        System.out.println("Bark");
    }
}
```

Then:

```java
Dog dog = new Dog();
dog.speak();
dog.eat();
```

But this is not allowed:

```java
Animal animal = new Animal(); // compilation error
```

because `Animal` is abstract.

## Why Make a Class Abstract?

An abstract class is useful when the base type represents a general concept that should not be instantiated directly.

For example:

```text
Employee
   |
   +---- Developer
   +---- Manager
   +---- Tester
```

Each employee type may calculate salary differently:

```java
abstract class Employee {
    abstract double calculateSalary();
}

class Developer extends Employee {
    @Override
    double calculateSalary() {
        return 100000;
    }
}

class Manager extends Employee {
    @Override
    double calculateSalary() {
        return 150000;
    }
}
```

The abstract class establishes the common abstraction while subclasses provide specialized behavior.

## Abstract Methods

An abstract method has no implementation in the abstract class:

```java
abstract class Animal {
    abstract void speak();
}
```

It effectively says:

> **Every concrete subclass must provide this behavior.**

If a subclass does not implement the abstract method, that subclass must itself be abstract:

```java
abstract class Dog extends Animal {
}
```

Otherwise, compilation fails.

## Interfaces as Abstraction

Interfaces are another major abstraction mechanism.

```java
interface Payment {
    void pay(double amount);
}
```

An implementation provides the details:

```java
class UpiPayment implements Payment {

    @Override
    public void pay(double amount) {
        System.out.println("Processing UPI payment");
    }
}
```

Another implementation:

```java
class CardPayment implements Payment {

    @Override
    public void pay(double amount) {
        System.out.println("Processing card payment");
    }
}
```

The caller only needs to know about:

```text
Payment
   |
  pay()
```

not the internal implementation of UPI or card processing.

## Abstraction + Polymorphism

These two concepts work particularly well together.

```java
void processPayment(Payment payment) {
    payment.pay(1000);
}
```

The caller can pass:

```java
processPayment(new UpiPayment());
processPayment(new CardPayment());
```

The method depends only on the `Payment` abstraction, while runtime polymorphism selects the concrete implementation.

```text
Abstraction
     ↓
defines a contract
     ↓
Polymorphism
     ↓
allows different implementations
     ↓
loose coupling
```

## Abstraction Does Not Mean "No Implementation"

An abstract class can contain concrete implementation:

```java
abstract class Animal {

    abstract void speak();

    void eat() {
        System.out.println("Eating");
    }
}
```

Therefore, abstraction does **not** mean that everything must be abstract.

The goal is to hide implementation details that the caller does not need to know while exposing essential behavior.

## Abstract Class vs Interface

### Abstract Class

Use an abstract class when you have a genuine common base and want to share things such as:

- State.
- Constructors.
- Common implementation.
- Common behavior.
- Abstract behavior that subclasses must provide.

Example:

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

Here, `Employee` is a genuine base type. Developers and other employees can share state (`name`), constructor logic, and common behavior (`clockIn`), while subclasses provide their own salary calculation.

### Interface

Use an interface when you want to define a contract or capability that potentially unrelated classes can satisfy.

Example:

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

`Employee` and `Invoice` do not need to belong to the same class hierarchy. They simply satisfy the `Payable` capability.

### Mental Shortcut

```text
Abstract class
      ↓
"What are you?"
      ↓
Employee
   ├── Developer
   ├── Manager
   └── Tester

Interface
      ↓
"What can you do?"
      ↓
Payable
   ├── Employee
   └── Invoice
```

A class can use both:

```java
class Developer extends Employee implements Payable {
    // shared Employee state/behavior
    // + Payable contract
}
```

## Abstraction vs Interface

Do not equate the abstraction principle with the Java `interface` keyword.

Similarly:

```text
Encapsulation
    ↓
private / access modifiers
```

does not mean:

```text
private = encapsulation
```

Likewise:

```text
Abstraction
    ↓
interfaces / abstract classes
```

does not mean:

```text
interface = abstraction
```

The language features are **mechanisms used to achieve the design principle**.

## Abstraction + Encapsulation

These principles often work together.

```java
class BankAccount {

    private double balance;

    public void withdraw(double amount) {
        if (amount > balance) {
            throw new IllegalStateException();
        }

        balance -= amount;
    }
}
```

### Encapsulation

```java
private double balance;
```

protects the internal state.

### Abstraction

```java
withdraw(amount)
```

provides a meaningful operation without requiring the caller to know how withdrawal is implemented.

## Benefits of Abstraction

### Reduced Complexity

Consumers do not need to understand implementation details.

### Loose Coupling

Code can depend on:

```java
Payment
```

rather than a particular concrete implementation.

### Replaceable Implementations

An implementation can be replaced without changing code that depends only on the abstraction.

For example:

```text
NotificationService
       |
       +-- EmailNotificationService
       +-- SmsNotificationService
```

Code using `NotificationService` does not need to know which concrete implementation is being used.

### Easier Testing

A different implementation can be supplied for tests instead of the real external service.

## Abstraction in Backend Design

A common Java backend pattern is:

```java
interface NotificationService {
    void send(String message);
}
```

with multiple implementations:

```java
class EmailNotificationService implements NotificationService {
    @Override
    public void send(String message) {
        // email implementation
    }
}

class SmsNotificationService implements NotificationService {
    @Override
    public void send(String message) {
        // SMS implementation
    }
}
```

Other components depend on:

```java
NotificationService
```

rather than directly depending on one concrete implementation.

This idea becomes especially important when learning **dependency inversion** and **dependency injection** in Spring.

## The Four OOP Principles

The four commonly discussed OOP principles are:

```text
                 OOP
                  |
       +----------+----------+----------+
       |          |          |          |
 Encapsulation Inheritance Polymorphism Abstraction
```

### Encapsulation

Controls access to internal state and implementation details.

### Inheritance

Extends an existing class through an IS-A relationship.

### Polymorphism

Allows one abstraction/reference to represent different concrete implementations and produce different behavior.

### Abstraction

Exposes essential behavior while hiding unnecessary implementation complexity.

These principles often work together rather than existing independently.

## Interview-Level Takeaways

- **Abstraction = expose essential behavior while hiding unnecessary implementation details.**
- Interfaces and abstract classes are major Java mechanisms for implementing abstraction.
- An abstract class cannot be instantiated.
- Abstract classes can contain both abstract and concrete methods.
- Abstract classes can have state and constructors.
- A concrete subclass must implement inherited abstract methods or itself be abstract.
- Interfaces define contracts/capabilities that implementations satisfy.
- Abstraction allows code to depend on **contracts rather than concrete implementations**.
- Abstraction + polymorphism is a major source of loose coupling.
- **Abstraction is not the same thing as an interface**; an interface is a language mechanism used to achieve abstraction.
- **Abstraction is not the same thing as encapsulation.**
- Abstract class → useful for a common base with shared state/implementation.
- Interface → useful for a contract/capability that potentially unrelated classes can implement.
- A class can extend one class and implement multiple interfaces.
- **Encapsulation asks:** "How do I protect my internals?"
- **Abstraction asks:** "What does the outside world actually need to know?"
