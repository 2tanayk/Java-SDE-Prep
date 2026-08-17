# Class Initialization Order in Java

## Overview

Class initialization and object initialization are different things.

```text
Class level
├── static field initialization
└── static initializer blocks

Object level
├── instance field initialization
├── instance initializer blocks
└── constructor
```

Class-level initialization happens once per class initialization. Object-level initialization happens whenever a new object is created.

## Class Initialization

Static field initializers and static initializer blocks execute in **textual order** within a class.

```java
class Test {

    static int x = 10;

    static {
        System.out.println("Static block");
    }
}
```

Order:

```text
1. Initialize x
2. Execute static block
```

If the order is reversed in the source code, the execution order is reversed as well.

> **Static field initializers and static blocks execute in the order they appear in the class.**

## Class Initialization and Inheritance

Before a class is initialized, its superclass must already have been initialized.

```java
class Parent {
    static {
        System.out.println("Parent static");
    }
}

class Child extends Parent {
    static {
        System.out.println("Child static");
    }
}
```

For:

```java
new Child();
```

the class-level order is:

```text
Parent class initialization
        ↓
Child class initialization
```

Output:

```text
Parent static
Child static
```

## Object Initialization

After the required class initialization has occurred, creating an object performs instance initialization.

Consider:

```java
class Parent {

    static {
        System.out.println("Parent static");
    }

    {
        System.out.println("Parent instance block");
    }

    Parent() {
        System.out.println("Parent constructor");
    }
}

class Child extends Parent {

    static {
        System.out.println("Child static");
    }

    {
        System.out.println("Child instance block");
    }

    Child() {
        System.out.println("Child constructor");
    }
}
```

For:

```java
new Child();
```

the order is:

```text
Parent static
Child static

Parent instance block
Parent constructor

Child instance block
Child constructor
```

The important boundary is:

```text
CLASS INITIALIZATION
        ↓
Parent static stuff
        ↓
Child static stuff
        ↓
OBJECT INITIALIZATION
        ↓
Parent instance stuff
        ↓
Parent constructor
        ↓
Child instance stuff
        ↓
Child constructor
```

## Complete Initialization Order

For a normal:

```java
new Child();
```

where:

```java
Child extends Parent
```

the useful mental model is:

```text
1. Initialize Parent class
       ↓
2. Initialize Child class
       ↓
3. Create/allocate Child object
       ↓
4. Initialize Parent instance state
       ↓
5. Execute Parent constructor
       ↓
6. Initialize Child instance state
       ↓
7. Execute Child constructor
```

Instance state initialization includes instance field initializers and instance initializer blocks.

## Instance Fields and Instance Initializer Blocks

Instance field initializers and instance initializer blocks also execute in **textual order**.

```java
class Parent {

    int a = print("Parent field 1");

    {
        print("Parent block 1");
    }

    int b = print("Parent field 2");

    {
        print("Parent block 2");
    }

    Parent() {
        print("Parent constructor");
    }

    static int print(String message) {
        System.out.println(message);
        return 0;
    }
}
```

The instance initialization order is:

```text
Parent field 1
Parent block 1
Parent field 2
Parent block 2
Parent constructor
```

It is **not**:

```text
all fields
→ all blocks
→ constructor
```

Instead, fields and initializer blocks execute according to their position in the source code, followed by the constructor body.

## Where `super()` Fits

If a child constructor does not explicitly invoke a superclass constructor:

```java
class Child extends Parent {

    Child() {
        System.out.println("Child constructor");
    }
}
```

the compiler implicitly adds:

```java
Child() {
    super();
    System.out.println("Child constructor");
}
```

Therefore, the parent constructor executes before the child constructor body.

Conceptually:

```text
Child constructor execution
        ↓
implicit/explicit super()
        ↓
Parent instance initialization
        ↓
Parent constructor body
        ↓
return to Child constructor
        ↓
Child instance initialization
        ↓
Child constructor body
```

The important point is that constructor chaining ensures the parent portion of the object is initialized before the child portion.

## Static Initialization Happens Only Once

Consider:

```java
new Child();
new Child();
```

The first creation performs:

```text
Parent static
Child static

Parent instance
Parent constructor
Child instance
Child constructor
```

The second creation performs only the object-level initialization:

```text
Parent instance
Parent constructor
Child instance
Child constructor
```

Static initialization does not repeat for every object.

> **Class initialization occurs once per class initialization by a given class loader; instance initialization occurs for every new object.**

## Class Loading vs Class Initialization

Do not confuse these concepts.

```text
Class loading
    ↓
Class linking
    ↓
Class initialization
    ↓
static field initializers / static blocks
```

Class initialization is the phase in which the class's static initialization actually takes place.

Object creation then has its own instance initialization sequence:

```text
Object creation
    ↓
instance field initialization
    ↓
instance initializer blocks
    ↓
constructor
```

## Complete Example

```java
class Parent {

    static int a = print("Parent static field");

    static {
        print("Parent static block");
    }

    int x = print("Parent instance field");

    {
        print("Parent instance block");
    }

    Parent() {
        print("Parent constructor");
    }

    static int print(String s) {
        System.out.println(s);
        return 0;
    }
}

class Child extends Parent {

    static int b = print("Child static field");

    static {
        print("Child static block");
    }

    int y = print("Child instance field");

    {
        print("Child instance block");
    }

    Child() {
        print("Child constructor");
    }
}
```

For:

```java
new Child();
```

the output is:

```text
Parent static field
Parent static block

Child static field
Child static block

Parent instance field
Parent instance block
Parent constructor

Child instance field
Child instance block
Child constructor
```

## Interview-Level Takeaways

- **Class initialization and object initialization are different.**
- Static field initializers and static blocks execute in **textual order**.
- Before a class is initialized, its superclass must already have been initialized.
- Static initialization happens once per class initialization by a given class loader, not once per object.
- Instance field initializers and instance initializer blocks execute in **textual order**.
- Parent instance initialization happens before the parent constructor body.
- The parent constructor completes before the child constructor body executes.
- If a child constructor does not explicitly call `super(...)`, the compiler implicitly inserts `super()` when applicable.
- For `new Child()` where `Child extends Parent`, think:

```text
Parent static
→ Child static
→ Parent instance fields/blocks
→ Parent constructor
→ Child instance fields/blocks
→ Child constructor
```

- **Loading ≠ initialization:** class loading/linking happen before class initialization, while static initialization occurs during class initialization.
