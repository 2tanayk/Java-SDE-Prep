# Polymorphism in Java

## Overview

> **Polymorphism means the same reference/contract can represent different concrete implementations, allowing the same operation to produce different behavior depending on the implementation.**

The two major forms commonly discussed in Java are:

```text
Polymorphism
    |
    +----------------------+
    |                      |
Compile-time          Runtime
polymorphism          polymorphism
    |                      |
Method overloading    Method overriding
```

## Runtime Polymorphism

Runtime polymorphism occurs when a parent-class or interface reference points to a concrete implementation and an overridden instance method is dispatched based on the runtime object.

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

Animal animal = new Dog();
animal.speak(); // Dog
```

The important distinction is:

- **Reference type (`Animal`)** controls which members are available through the reference at compile time.
- **Runtime object type (`Dog`)** determines which overridden instance-method implementation executes.

> **Reference type determines what you can call; runtime object determines which overridden implementation runs.**

## Why Polymorphism Is Useful

Polymorphism allows code to depend on an abstraction rather than knowing every concrete implementation.

Instead of:

```java
if (animal instanceof Dog) {
    // bark
} else if (animal instanceof Cat) {
    // meow
}
```

you can write:

```java
void makeAnimalSpeak(Animal animal) {
    animal.speak();
}
```

Then:

```java
makeAnimalSpeak(new Dog());
makeAnimalSpeak(new Cat());
```

The method does not need to know which concrete animal it received.

This is an important practical benefit: **polymorphism reduces coupling to concrete implementations**.

## Interface-Based Polymorphism

Runtime polymorphism does not require class inheritance specifically. It also works through interfaces.

```java
interface Payment {
    void pay();
}

class UpiPayment implements Payment {
    @Override
    public void pay() {
        System.out.println("UPI");
    }
}

class CardPayment implements Payment {
    @Override
    public void pay() {
        System.out.println("Card");
    }
}
```

Now:

```java
void processPayment(Payment payment) {
    payment.pay();
}

processPayment(new UpiPayment());
processPayment(new CardPayment());
```

The same `Payment` abstraction produces different behavior.

This pattern is extremely common in Java backend applications.

## Compile-Time Polymorphism — Method Overloading

Method overloading means multiple methods have the same name but **different parameter lists**.

```java
class Calculator {

    int add(int a, int b) {
        return a + b;
    }

    double add(double a, double b) {
        return a + b;
    }

    int add(int a, int b, int c) {
        return a + b + c;
    }
}
```

The compiler determines which overload is selected:

```java
Calculator c = new Calculator();

c.add(1, 2);
c.add(1.0, 2.0);
c.add(1, 2, 3);
```

Therefore:

> **Overloading is resolved at compile time.**

## What Makes Methods Overloaded?

The **parameter list must differ**.

Valid overloads:

```java
void foo(int x)
void foo(double x)
void foo(int x, int y)
```

because their parameter lists differ.

### Access Modifiers Do Not Determine Overloading

This is valid because the parameter lists differ:

```java
void foo(int x) {
}

private void foo(String x) {
}
```

The `public`, `private`, `protected`, or package-private modifier does not make methods overloaded or non-overloaded.

### Same Parameter List = Duplicate Method

If the parameter list is identical:

```java
void foo(int x) {
}

private void foo(int x) {
}
```

this is a **compilation error**.

Changing only the access modifier does not create an overload.

The same applies to other modifiers such as `static` or `final`.

### Return Type Alone Cannot Overload a Method

This is also illegal:

```java
int foo(int x) {
    return x;
}

double foo(int x) {
    return x;
}
```

The parameter list is identical, and changing only the return type does not create an overload.

> **Overloading requires a different parameter list. Return type, access modifiers, `static`, and `final` do not by themselves create an overload.**

## Overload Resolution Uses Compile-Time Types

Consider:

```java
class Printer {

    void print(Animal animal) {
        System.out.println("Animal");
    }

    void print(Dog dog) {
        System.out.println("Dog");
    }
}
```

Now:

```java
Animal animal = new Dog();
Printer printer = new Printer();

printer.print(animal);
```

The selected overload is:

```text
print(Animal)
```

not `print(Dog)`.

Why? **Overload resolution happens at compile time**, and the declared/reference type of `animal` is `Animal`.

This is a key distinction:

```text
Overloading
    ↓
compile time
    ↓
uses compile-time / declared types

Overriding
    ↓
runtime
    ↓
uses the actual runtime object for instance-method dispatch
```

## Overriding vs Overloading

| | Overloading | Overriding |
|---|---|---|
| Polymorphism | Compile-time | Runtime |
| Relationship | Usually same class | Parent-child / interface implementation |
| Method name | Same | Same |
| Parameters | **Must differ** | Must match according to overriding rules |
| Return type | Cannot be the only difference | Same or covariant |
| Resolution | Compiler | Runtime for instance methods |
| Static | Can participate in overloads | Not overridden; hidden |
| Private | Can participate in overloads | Cannot be overridden |

## Static Methods and Polymorphism

Static methods are not overridden through runtime polymorphism.

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

Parent obj = new Child();
obj.hello(); // Parent
```

Static methods are **hidden**, and selection is based on the reference/class type rather than runtime object dispatch.

```text
Instance method
    ↓
Overriding
    ↓
Runtime polymorphism

Static method
    ↓
Method hiding
    ↓
Class/reference type
```

## Fields Are Not Dynamically Dispatched

Fields do not participate in runtime polymorphism like overridden instance methods.

```java
class Parent {
    int x = 10;
}

class Child extends Parent {
    int x = 20;
}

Parent obj = new Child();
System.out.println(obj.x); // 10
```

Field access is based on the reference type.

Compare:

```text
Method:
obj.method()
→ overridden implementation → runtime object

Field:
obj.field
→ reference type
```

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

`Dog` is a subtype of `Animal`, so the more specific return type is valid.

An unrelated return type is not allowed.

## Practical Mental Model

```text
                    Polymorphism
                         |
              +----------+----------+
              |                     |
        Compile-time            Runtime
              |                     |
         Overloading            Overriding
              |                     |
         compiler chooses       runtime dispatch
         based on declared      based on actual
         /compile-time types    object type
```

For runtime polymorphism:

```text
same abstraction / contract
          ↓
multiple concrete implementations
          ↓
same operation
          ↓
different behavior
```

## Interview-Level Takeaways

- Polymorphism means one abstraction/reference can represent different concrete implementations.
- **Compile-time polymorphism → method overloading.**
- **Runtime polymorphism → method overriding.**
- Overloading requires a **different parameter list**.
- Changing only return type does not overload a method.
- Changing only access modifiers does not overload a method.
- Changing only `static`/instance status or `final` does not overload a method.
- Methods with the same parameter list cannot coexist merely because their modifiers or return types differ.
- Overload resolution happens at compile time and uses compile-time/declared types.
- Overridden instance methods are dynamically dispatched using the runtime object.
- Static methods are hidden, not overridden.
- Fields are not dynamically dispatched.
- Runtime polymorphism works through both class inheritance and interface implementation.
- Interface-based polymorphism is a major mechanism for reducing coupling to concrete implementations.
- Covariant return types allow an overriding method to return the same type or a more specific subtype.
