# JDK, JRE and JVM

## Overview

Java execution can be understood through three related concepts:

- **JDK (Java Development Kit):** Provides the tools required to develop Java applications, along with the runtime needed to execute them.
- **JRE (Java Runtime Environment):** Conceptually, the environment required to run Java applications — the JVM plus the Java runtime libraries.
- **JVM (Java Virtual Machine):** The runtime engine that executes Java bytecode.

> Modern Java note: standalone JRE distributions are no longer provided as a separate Oracle JDK product starting with JDK 11. The JRE concept is still useful for understanding the Java runtime architecture.

## The Java Execution Flow

When we write Java source code, it does not execute directly on the CPU.

```text
Hello.java
    |
    | javac
    v
Hello.class
(bytecode)
    |
    | JVM
    v
Machine / native code
    |
    v
CPU
```

The important distinction is that `javac` compiles Java source code into **bytecode**, while the JVM executes that bytecode on the target machine.

## JDK

The JDK is what a developer needs to **build and run** Java applications.

Conceptually:

```text
JDK
├── Development tools
│   ├── javac
│   ├── javadoc
│   ├── jar
│   └── jdb
│
└── Runtime
    └── JVM + Java runtime libraries
```

The most important tool for this topic is `javac`:

```bash
javac Hello.java
```

This produces:

```text
Hello.class
```

The `.class` file contains JVM bytecode.

## JRE

The JRE is the conceptual runtime environment needed to **run** a Java application.

```text
JRE
├── JVM
└── Java runtime libraries
```

The JVM itself executes bytecode, while the Java runtime libraries provide standard APIs used by applications, such as `String`, `List`, `HashMap`, `Thread`, `Optional`, and file APIs.

## JVM

The JVM is the engine responsible for executing Java bytecode.

The JVM is **platform-specific**, while Java bytecode is designed to be **platform-independent**.

For example, the same bytecode can conceptually be run using:

```text
             Java bytecode
                  |
        +---------+---------+
        |                   |
        v                   v
    Linux JVM          Windows JVM
        |                   |
        v                   v
   Linux/CPU          Windows/CPU
```

This is the key idea behind Java's "write once, run anywhere" model: the same compiled bytecode can run on different platforms when an appropriate JVM implementation exists for that platform.

## Interpretation

The JVM can execute bytecode using an **interpreter**.

The interpreter executes bytecode instructions sequentially. It does **not** read the original `.java` source file line by line.

For example, one Java source statement can correspond to multiple bytecode instructions:

```text
Java source
int z = x + y;

        |
        | javac
        v

Bytecode (conceptual)
load x
load y
add
store z

        |
        v

Interpreter
execute instruction 1
execute instruction 2
execute instruction 3
execute instruction 4
```

Therefore, "line by line" is only a rough beginner's approximation. More accurately:

> **The JVM interpreter fetches and executes bytecode instructions one at a time.**

The JVM never interprets the original `.java` source code; `javac` has already converted it into `.class` bytecode.

## JIT Compilation

JIT stands for **Just-In-Time compilation**.

Instead of interpreting all bytecode forever, the JVM can identify code that is executed frequently and compile that **hot code** into native machine code at runtime.

Conceptually:

```text
                 Bytecode
                    |
                    v
               Interpreter
                    |
             Application runs
                    |
                    v
        Frequently executed code
              ("hot" code)
                    |
                    v
              JIT compiler
                    |
                    v
          Native machine code
                    |
                    v
                   CPU
```

Why not JIT-compile everything immediately? Compilation itself costs time and resources. If a method is executed only once, spending significant effort compiling it may not be worthwhile. The JVM can therefore start executing code and use JIT compilation where runtime execution makes the optimization worthwhile.

### Interpreter vs JIT

A useful mental model is:

- **Interpreter:** "Execute this bytecode instruction now."
- **JIT:** "This code is being executed frequently; compile it into native machine code so it can execute more efficiently."

Interpretation and JIT compilation are therefore not necessarily competing alternatives. They can be part of the same runtime execution process.

## Complete Mental Model

```text
                 Hello.java
                     |
                   javac
                     |
                     v
              Hello.class
               (bytecode)
                     |
                     v
                    JVM
                     |
             +-------+-------+
             |               |
       Interpreter       JIT compiler
             |               |
             |          hot code only
             |               |
             +-------+-------+
                     |
                     v
             Native execution
                     |
                     v
                    CPU
```

## Interview-Level Takeaways

- **JVM:** Executes Java bytecode.
- **JRE:** Conceptually, JVM + Java runtime libraries required to run Java applications.
- **JDK:** Development tools + runtime needed to develop and run Java applications.
- `javac` compiles `.java` source into `.class` bytecode.
- The JVM executes **bytecode**, not Java source code.
- The interpreter executes bytecode instructions sequentially; it does not literally execute Java source lines.
- JIT can compile frequently executed (hot) code into native machine code at runtime.
- Java bytecode is platform-independent; the JVM implementation is platform-specific.
- Deep JIT internals and JVM optimization mechanics are outside the initial SDE 2 preparation scope.
