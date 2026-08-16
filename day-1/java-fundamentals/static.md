# `static` in Java

## Overview

- `static` means a member belongs to the **class** rather than to individual objects.
- Static members are associated with the class itself and can generally be accessed without creating an instance.

```text
                    CLASS
                      |
          +-----------+-----------+
          |                       |
       static                  instance
          |                       |
     belongs to class        belongs to object
          |                       |
     one shared copy          one copy per object
          |                       |
     Class.member             object.member
```

## Static Variables

- A static variable has **one shared copy per class**.
- All objects of that class access the same static variable.

```java
class Counter {
    static int count = 0;

    void increment() {
        count++;
    }
}
```

```java
Counter c1 = new Counter();
Counter c2 = new Counter();

c1.increment();
c2.increment();

System.out.println(Counter.count); // 2
```

Conceptually:

```text
             Counter class
                  |
              static count
                  |
          +-------+-------+
          |       |       |
       object1 object2 object3
```

- Prefer accessing a static field through the class name:

```java
Counter.count
```

- Although Java may allow access through an instance reference in some cases, that style is misleading because the field belongs to the class, not the object.

## Static Methods

- A static method belongs to the class rather than an individual object.
- It can be called without creating an object.

```java
class MathUtils {
    static int add(int a, int b) {
        return a + b;
    }
}
```

Usage:

```java
MathUtils.add(10, 20);
```

- Static methods are common in utility APIs, for example:

```java
Math.max(10, 20);
Integer.parseInt("123");
```

## Static Methods and Instance Members

- A static method cannot directly access instance fields or instance methods.
- Instance members belong to a particular object, but a static method does not have an associated object.

```java
class Person {
    String name;

    static void printName() {
        System.out.println(name); // Does not compile
    }
}
```

- The problem is that there can be many `Person` objects, each with a different `name`.
- A static method such as `Person.printName()` has no particular `Person` instance to use.

Static methods **can** directly access static members:

```java
class Person {
    String name;
    static int count;

    static void printCount() {
        System.out.println(count); // Valid
    }
}
```

### Rule to Remember

> **A static context cannot directly access instance members because it has no implicit object (`this`) associated with it.**

## Static Blocks

- A **static initialization block** is a block declared with `static`:

```java
class Database {
    static {
        System.out.println("Database initialized");
    }
}
```

- Static blocks execute as part of **class initialization**.
- Static initialization happens at the class level, not once for every object created from the class.

For example:

```java
class Test {
    static {
        System.out.println("Initialized");
    }
}

Test t1 = new Test();
Test t2 = new Test();
Test t3 = new Test();
```

- The static initialization block does not execute once per object.
- The class's static initialization occurs when the class is initialized.

## Static Initialization Order

This is an important interview detail.

- Static field initializers and static initialization blocks are both part of **class initialization**.
- They execute in **textual/source order**.
- It is **not** correct to think of initialization as "execute all static field initializers first, then execute all static blocks."

Example:

```java
class Test {

    static int x = initializeX();

    static {
        System.out.println("Block");
    }

    static int y = initializeY();
}
```

Execution order:

```text
1. initializeX()
2. static block
3. initializeY()
```

If the block appears first:

```java
class Test {

    static {
        System.out.println("Block");
    }

    static int x = initializeX();
}
```

Then the order is:

```text
1. static block
2. initializeX()
```

### The Rule

> **During class initialization, static field initializers and static blocks execute in the order they appear in the class.**

## `static` and Class Initialization

This connects directly to class loading.

The broad lifecycle is:

```text
.class
  |
  v
Loading
  |
  v
Linking
  |
  v
Preparation
  |
  +--> Static fields get memory
  +--> Static fields get default values
  |
  v
Initialization
  |
  +--> Static field initializers
  +--> Static blocks
  |    (in source order)
  |
  v
Class initialized
```

For:

```java
class Test {
    static int x = 10;
}
```

### During Preparation

- Memory is allocated for `x`.
- `x` receives its default value:

```text
x = 0
```

### During Initialization

- The explicit initializer is executed:

```text
x = 10
```

Therefore:

- **Preparation:** memory allocation + default values for static fields.
- **Initialization:** explicit static field initialization + static blocks, in source order.

## `static final`

- `static final` is commonly used to declare constants.
- `static` means the field belongs to the class.
- `final` means the field cannot be reassigned after initialization.

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

- Constants are conventionally named using `UPPER_SNAKE_CASE`.

## `static` Does Not Mean "Global"

- It is tempting to describe `static` as Java's version of a global variable, but that is not a good mental model.
- Java does not have traditional global variables.
- A static field still belongs to a specific class and is accessed through that class.

```java
class Config {
    static String environment;
}
```

- `environment` is a field of `Config`; it is not a language-level global variable.

## Static Nested Classes — Awareness Only

You may encounter:

```java
class Outer {
    static class Inner {
    }
}
```

- This is a **static nested class**.
- It is a different use of `static` from static fields and static methods.
- Detailed nested-class behavior is outside the scope of this topic.

## Interview-Level Takeaways

- `static` means **class-level**, rather than instance-level.
- A static variable has **one shared copy per class**.
- A static method can be called without creating an object.
- A static method cannot directly access instance members because it has no implicit `this` object.
- Static methods can directly access static members.
- Static blocks execute during **class initialization**.
- Static initialization happens at the class level, not once per object.
- During **Preparation**, memory is allocated for static fields and they receive default values.
- During **Initialization**, explicit static field initializers and static blocks execute **in source order**.
- `static final` is commonly used for constants.
- `static` does not mean "global".
