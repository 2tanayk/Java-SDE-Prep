# `this` and `super` in Java

## Overview

- `this` refers to the **current object/current class context**.
- `super` is used to explicitly access the **superclass portion/members of the current object**.
- Both are especially important when working with constructors, inheritance, method overriding, and field shadowing.

```text
this
  ↓
current class / current object

super
  ↓
superclass
```

## `this`

### `this` Refers to the Current Object

```java
class Person {
    String name;

    void printName() {
        System.out.println(this.name);
    }
}
```

If:

```java
Person person = new Person();
person.name = "Tanay";
person.printName();
```

then inside `printName()`:

```java
this
```

refers to the `person` object.

Therefore:

```java
this.name
```

means the `name` field belonging to the current `Person` object.

### `this` for Field/Parameter Name Conflicts

A very common use of `this` is distinguishing an instance field from a constructor parameter or local variable with the same name.

```java
class Person {
    String name;

    Person(String name) {
        this.name = name;
    }
}
```

Here:

- `this.name` → instance field.
- `name` → constructor parameter.

Without `this`:

```java
Person(String name) {
    name = name;
}
```

both references refer to the parameter, so the instance field is not assigned.

### `this` for Calling Instance Methods

You can explicitly call another instance method through `this`:

```java
class Person {

    void hello() {
        this.sayHello();
    }

    void sayHello() {
        System.out.println("Hello");
    }
}
```

In this context:

```java
this.sayHello();
```

means to call `sayHello()` on the current object.

Usually the `this` is optional:

```java
sayHello();
```

has the same meaning in this context.

## `this(...)` — Calling Another Constructor

`this(...)` calls another constructor of the **same class**.

```java
class Person {

    String name;
    int age;

    Person() {
        this("Unknown", 0);
    }

    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

The constructor chain is:

```text
Person()
   |
   | this("Unknown", 0)
   v
Person(String, int)
```

### Important Rule

- A `this(...)` constructor invocation **must be the first statement** in the constructor.

Invalid:

```java
Person() {
    System.out.println("Hello");
    this("Unknown", 0); // compilation error
}
```

## `super`

`super` is used to explicitly access members or constructor behavior from the superclass.

### `super.field`

Consider:

```java
class Animal {
    String name = "Animal";
}

class Dog extends Animal {
    String name = "Dog";

    void printNames() {
        System.out.println(name);
        System.out.println(super.name);
    }
}
```

Conceptually:

```text
Dog object
  |
  +-- Dog.name    = "Dog"
  |
  +-- Animal.name = "Animal"
```

Therefore:

- `name` → `Dog.name` in this context.
- `super.name` → explicitly accesses the superclass's `name` field.

## `super.method()` — Calling the Parent Implementation

`super` is commonly used after overriding a method when the child still wants to invoke the parent's implementation.

```java
class Animal {

    void speak() {
        System.out.println("Animal speaks");
    }
}

class Dog extends Animal {

    @Override
    void speak() {
        System.out.println("Dog barks");
        super.speak();
    }
}
```

Calling:

```java
dog.speak();
```

results conceptually in:

```text
Dog.speak()
   |
   +--> "Dog barks"
   |
   +--> super.speak()
             |
             v
       Animal.speak()
```

## `super(...)` — Calling the Parent Constructor

`super(...)` invokes a constructor of the superclass.

```java
class Animal {

    Animal(String name) {
        System.out.println(name);
    }
}

class Dog extends Animal {

    Dog(String name) {
        super(name);
    }
}
```

Here:

```java
super(name);
```

means:

> Call the matching constructor of the parent class.

### Important Rule

- A `super(...)` constructor invocation **must be the first statement** in the constructor.

## What Happens When `super()` Is Not Written?

If a constructor does not explicitly invoke another constructor using `this(...)` or `super(...)`, Java implicitly inserts a call to the superclass's no-argument constructor, provided an accessible no-argument superclass constructor exists.

For example:

```java
class Animal {

    Animal() {
        System.out.println("Animal");
    }
}

class Dog extends Animal {

    Dog() {
        System.out.println("Dog");
    }
}
```

Conceptually, Java treats the child constructor as:

```java
Dog() {
    super();
    System.out.println("Dog");
}
```

So:

```text
new Dog()
   |
   v
Animal()
   |
   v
Dog()
```

The parent constructor executes before the child constructor body.

## `this(...)` and Implicit `super()`

This is an important constructor-chain detail.

Consider:

```java
class Child extends Parent {

    Child() {
        this(10);
    }

    Child(int x) {
        // no explicit super(...)
    }
}
```

The first constructor does **not** get an implicit `super()` because it already delegates to another constructor using `this(10)`.

Instead, the constructor chain is:

```text
Child()
   |
   | this(10)
   v
Child(int x)
   |
   | implicit super()
   v
Parent()
```

So if `Child(int x)` does not explicitly call `super(...)`, Java inserts `super()` there, assuming `Parent()` exists and is accessible.

Conceptually:

```java
Child() {
    this(10);
}

Child(int x) {
    super(); // implicit
    // ...
}
```

### Key Rule

> **`this(...)` does not itself call the parent constructor. It delegates to another constructor in the same class. The constructor chain eventually reaches a constructor that invokes `super(...)` explicitly or implicitly.**

For example:

```text
this()
  |
  v
this(...)
  |
  v
this(...)
  |
  v
super()
  |
  v
Parent constructor
```

This is why `this(...)` and `super(...)` are alternatives as the first constructor statement, but the constructor chain must ultimately initialize the superclass.

## Parent Without a No-Argument Constructor

Consider:

```java
class Animal {

    Animal(String name) {
    }
}

class Dog extends Animal {

    Dog() {
    }
}
```

Java attempts to insert:

```java
super();
```

but `Animal()` does not exist.

Therefore this produces a compilation error.

The child must explicitly invoke an available parent constructor:

```java
class Dog extends Animal {

    Dog() {
        super("Dog");
    }
}
```

## `this(...)` vs `super(...)`

| Syntax | Meaning |
|---|---|
| `this.field` | Current object's instance field |
| `this.method()` | Method call on the current object |
| `this(...)` | Calls another constructor in the same class |
| `super.field` | Accesses a superclass field |
| `super.method()` | Invokes the superclass method implementation |
| `super(...)` | Calls a superclass constructor |

The simplest mental model is:

```text
this
  ↓
CURRENT class / object

super
  ↓
PARENT class
```

## `this(...)` and `super(...)` Cannot Both Be Used as the First Statement

Both constructor invocations must be the first statement, so a constructor cannot contain both:

```java
Person() {
    this("Tanay");
    super(); // compilation error
}
```

A constructor can choose one:

```java
Person() {
    this("Tanay");
}
```

or:

```java
Dog() {
    super("Animal");
}
```

## `super` Does Not Mean a Separate Parent Object

If you have:

```java
Dog dog = new Dog();
```

there is one `Dog` object. `super` does not refer to a separate `Animal` object.

A useful conceptual model is:

```text
             Dog object
        +-------------------+
        | Dog state         |
        |                   |
        | Animal state      |
        +-------------------+
```

`super` provides a way for the child class to explicitly access superclass members or constructor behavior associated with that same object.

## Interview-Level Takeaways

- `this` refers to the current object/current class context.
- `this.field` is commonly used to distinguish an instance field from a parameter/local variable with the same name.
- `this.method()` invokes a method on the current object.
- `this(...)` invokes another constructor in the same class.
- `super.field` accesses a superclass field.
- `super.method()` invokes the superclass implementation of an overridden method.
- `super(...)` invokes a superclass constructor.
- `this(...)` and `super(...)` must be the first statement in a constructor.
- If a constructor does not explicitly call `this(...)` or `super(...)`, Java implicitly inserts `super()` when an accessible no-argument superclass constructor exists.
- If a constructor uses `this(...)`, the implicit `super()` is not inserted into that constructor; it is eventually reached through the delegated constructor chain.
- `this(...)` does not itself invoke the parent constructor; it delegates to another constructor in the same class.
- Parent construction happens before the child constructor body.
