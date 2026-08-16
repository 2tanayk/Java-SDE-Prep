# Class Loading Basics

## Overview

Before the JVM can use a class, it needs to make the class definition available to the runtime.

The broad lifecycle is:

```text
.class file
   |
   v
Loading
   |
   v
Linking
   |
   v
Initialization
   |
   v
Class ready for use
```

## What Does a Class Loader Do?

- A **Class Loader** is responsible for loading class definitions into the JVM.
- Conceptually, it takes a `.class` file and makes the corresponding class definition available to the JVM.
- For example:

```text
UserService.class
      |
      | Class Loader
      v
UserService class definition
      |
      v
     JVM
```

## Class Loading vs Object Creation

These are two different concepts:

- **Class loading:** Makes the `UserService` class definition available to the JVM.
- **Object creation:** Creates an actual instance of that class.

For example:

```java
UserService service = new UserService();
```

Conceptually:

```text
UserService.class
      |
      | class loading
      v
UserService class available
      |
      | new UserService()
      v
UserService object
```

- One loaded class can be used to create many objects.

## When Does Class Loading Happen?

- Classes are generally loaded **lazily**, meaning they are loaded when required rather than loading every class in the application at startup.
- Conceptually:

```text
JVM starts
   |
   v
Main loaded
   |
   v
main() starts
   |
   v
UserService required
   |
   v
UserService loaded
```

- The exact timing is governed by JVM rules, so avoid the oversimplification that a class is always loaded exactly when a particular source-level statement appears.

## Loading, Linking and Initialization

After a class is required, its lifecycle can be understood in three broad stages:

1. **Loading**
2. **Linking**
3. **Initialization**

### 1. Loading

- The JVM finds the class's bytecode.
- The class loader loads the class definition into the JVM.
- Conceptually:

```text
UserService.class
      |
      v
Class Loader
      |
      v
UserService class definition
```

### 2. Linking

Linking broadly consists of:

- **Verification**
- **Preparation**
- **Resolution**

#### Verification

- The JVM verifies that the loaded bytecode is valid and conforms to JVM requirements.
- Think of this as checking that the bytecode is structurally valid and safe for the JVM to execute.

#### Preparation

- The JVM allocates memory for **class-level/static fields**.
- Static fields receive their **default values** during preparation.
- Explicit initializers have **not** been applied yet.

For example:

```java
class Test {
    static int x = 10;
    static String name = "Tanay";
}
```

During preparation, conceptually:

```text
x    -> 0
name -> null
```

The explicit values `10` and `"Tanay"` are applied later during initialization.

#### Resolution

- The JVM can resolve symbolic references in the class to the actual runtime entities they refer to.
- For example, bytecode can contain symbolic references to another class, method, or field, and resolution connects those references to the corresponding runtime entities.
- For this level of preparation, understand the concept without going into constant-pool or JVM implementation details.

### 3. Initialization

- Initialization is where the JVM executes the class's **static initialization logic**.
- Explicit static field initializers are applied.
- Static initialization blocks are executed.

For example:

```java
class Database {
    static int connectionCount = 10;

    static {
        System.out.println("Database initialized");
    }
}
```

During initialization, conceptually:

```text
connectionCount -> 10

Static block executes:
"Database initialized"
```

## Preparation vs Initialization — Important Interview Distinction

This distinction is worth remembering:

- **Preparation:**
  - Allocates memory for static fields.
  - Gives static fields their default values.
  - Example: `static int x = 10` initially has `x = 0`.

- **Initialization:**
  - Executes static field initializers.
  - Executes static initialization blocks.
  - Example: `static int x = 10` becomes `x = 10` during initialization.

```text
Preparation
    |
    +--> Allocate memory for static fields
    |
    +--> Assign default values
    |
    v
Initialization
    |
    +--> Execute static field initializers
    |
    +--> Execute static blocks
    |
    v
Class initialized
```

### Interview Trap

- If asked **"When is memory allocated for static variables?"** → **Preparation**.
- If asked **"When is `static int x = 10` actually assigned `10`?"** → **Initialization**.

## Basic Class Loader Hierarchy — Awareness Only

Java runtimes have multiple class loaders. You may encounter:

- **Bootstrap Class Loader**
- **Platform Class Loader**
- **Application/System Class Loader**

Conceptually:

```text
Bootstrap
    |
    v
Platform
    |
    v
Application / System
```

- Class loaders also use a **parent delegation mechanism**, where a class-loading request is generally delegated to the parent before the child attempts to load the class itself.
- Detailed hierarchy, delegation mechanics, and custom class-loader implementation are outside the current preparation scope.

## Mental Model

```text
              UserService.class
                     |
                     v
                Class Loader
                     |
                     v
                  Loading
                     |
                     v
                  Linking
             +-------+-------+
             |       |       |
        Verification Preparation Resolution
                     |
                     v
               Initialization
                     |
             +-------+-------+
             |               |
       static fields   static blocks
             |               |
             +-------+-------+
                     |
                     v
               Class ready
                     |
                     v
             new UserService()
                     |
                     v
                Object created
```

## Interview-Level Takeaways

- A **Class Loader** loads class definitions into the JVM.
- **Class loading is not object creation.** A class can be loaded once and used to create many objects.
- Classes are generally loaded lazily when required.
- The broad lifecycle is **Loading → Linking → Initialization**.
- Linking broadly consists of **Verification → Preparation → Resolution**.
- **Preparation** allocates memory for static fields and gives them default values.
- **Initialization** executes explicit static field initializers and static blocks.
- Basic awareness of Bootstrap, Platform, and Application/System class loaders is sufficient for now.
- Detailed class-loader hierarchy and delegation mechanics are intentionally out of scope for the initial SDE 2 preparation.
