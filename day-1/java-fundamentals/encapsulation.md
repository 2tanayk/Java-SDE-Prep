# Encapsulation in Java

## Overview

> **Encapsulation means bundling an object's state and the operations that manipulate that state together, while controlling how the outside world can access or modify that state.**

The important part is not merely making fields `private`. The object should **control how its internal state can change**.

```java
public class BankAccount {

    private double balance;

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }

        balance += amount;
    }
}
```

The outside world cannot arbitrarily modify `balance`; it has to use the operations exposed by `BankAccount`.

## Why Not Make Everything Public?

With a public field:

```java
class BankAccount {
    public double balance;
}
```

callers can do:

```java
BankAccount account = new BankAccount();
account.balance = -500000;
```

The class has no opportunity to enforce its rules.

With encapsulation:

```java
class BankAccount {
    private double balance;

    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException();
        }

        balance += amount;
    }
}
```

the object controls access to its state.

## Encapsulation Is More Than `private`

`private` is a mechanism used to achieve encapsulation, not the entire concept.

Encapsulation involves:

```text
Hide internal representation
          +
Control access to it
          +
Expose well-defined operations
```

For example:

```java
class Temperature {

    private double celsius;

    public void setCelsius(double celsius) {
        if (celsius < -273.15) {
            throw new IllegalArgumentException(
                "Temperature cannot be below absolute zero"
            );
        }

        this.celsius = celsius;
    }

    public double getCelsius() {
        return celsius;
    }
}
```

The class controls the valid state of the object rather than exposing unrestricted access to `celsius`.

## Encapsulation Protects Invariants

An **invariant** is a condition that should remain true for a valid object.

Examples:

```text
BankAccount:
balance >= 0

Temperature:
celsius >= -273.15

Order:
total >= 0
```

Encapsulation allows the class to protect these rules.

```java
class BankAccount {

    private double balance;

    public void withdraw(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException();
        }

        if (amount > balance) {
            throw new IllegalStateException("Insufficient balance");
        }

        balance -= amount;
    }
}
```

The caller cannot arbitrarily modify `balance`; the class itself maintains the invariant.

## Getters and Setters Are Not Automatically Good Encapsulation

This is technically access-controlled:

```java
class User {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

But blindly exposing setters can still allow uncontrolled mutation.

For example, instead of:

```java
public void setBalance(double balance) {
    this.balance = balance;
}
```

prefer meaningful operations when the object needs to enforce rules:

```java
public void deposit(double amount)
public void withdraw(double amount)
```

The key principle is:

> **Do not blindly generate getters/setters for every field. Expose operations that make sense for the object's domain and allow the object to maintain its own rules.**

## Data Hiding vs Encapsulation

### Data hiding

Means preventing direct access to internal implementation details.

```java
private double balance;
```

### Encapsulation

Is the broader design principle of bundling state and behavior together and controlling access to the object's internals through a well-defined public interface.

Therefore:

```text
Data hiding
    ↓
one mechanism supporting
    ↓
Encapsulation
```

> **`private` helps achieve encapsulation; encapsulation is the broader design principle.**

## Access Modifiers

Java provides:

```text
public
protected
package-private
private
```

For internal implementation details, generally use the **least visibility necessary**.

For example:

```java
class User {

    private String passwordHash;

    public boolean verifyPassword(String password) {
        // ...
    }
}
```

The caller needs the operation `verifyPassword(...)`, not direct access to `passwordHash`.

## Encapsulation Allows Implementation Changes

Suppose the internal representation initially is:

```java
class User {
    private String fullName;

    public String getFullName() {
        return fullName;
    }
}
```

Later, it can change internally to:

```java
class User {
    private String firstName;
    private String lastName;

    public String getFullName() {
        return firstName + " " + lastName;
    }
}
```

Code using:

```java
user.getFullName();
```

does not need to know that the internal representation changed.

> **Encapsulation reduces the amount of code that depends on implementation details.**

## Encapsulation and API Design

Think of a class as having a public API and hidden implementation details:

```text
             BankAccount
        ┌────────────────────┐
        │                    │
Outside │  Public API        │
world → │                    │
        │  deposit()         │
        │  withdraw()        │
        │  getBalance()      │
        │                    │
        ├────────────────────┤
        │ Internal details   │
        │                    │
        │ balance            │
        │ validation rules   │
        │ implementation     │
        │                    │
        └────────────────────┘
```

The outside world should depend on the public API rather than the internal representation.

This makes the code easier to maintain and reduces coupling.

## Encapsulation and Mutable Collections

Making a collection field private does not automatically provide effective encapsulation if you return the mutable collection directly.

Problem:

```java
class Team {

    private List<String> members = new ArrayList<>();

    public List<String> getMembers() {
        return members;
    }
}
```

Now outside code can mutate internal state:

```java
team.getMembers().clear();
```

A safer option may be:

```java
public List<String> getMembers() {
    return List.copyOf(members);
}
```

Or expose meaningful operations:

```java
public void addMember(String member) {
    // validation/business rules
}

public void removeMember(String member) {
    // validation/business rules
}
```

This demonstrates the difference between merely hiding a field and actually controlling access to the object's state.

## Encapsulation vs Immutability

These concepts are related but different.

### Encapsulation

Controls access to state.

An object can be encapsulated and mutable:

```java
class BankAccount {
    private double balance;

    public void deposit(double amount) {
        balance += amount;
    }
}
```

### Immutability

Means the object's state cannot change after construction.

```java
class Money {

    private final BigDecimal amount;

    public Money(BigDecimal amount) {
        this.amount = amount;
    }

    public BigDecimal getAmount() {
        return amount;
    }
}
```

Therefore:

> **Encapsulation is not the same as immutability.**

Immutability can be implemented using encapsulation, but the concepts are distinct.

## Encapsulation vs Abstraction

This is a common interview question.

### Encapsulation

Focuses on:

> **How do I protect/control the object's internal state and implementation?**

Typical tools:

```text
private
access modifiers
controlled methods
```

### Abstraction

Focuses on:

> **What should the user of this component need to know?**

Typical tools:

```text
interfaces
abstract classes
abstract methods
```

A useful distinction:

```text
Encapsulation
→ hides/protects internal details

Abstraction
→ hides unnecessary complexity
  by exposing only essential behavior
```

## Mental Model

Think of a class like a machine:

```text
             ┌─────────────────────┐
             │      BankAccount    │
             │                     │
Outside ───→ │ deposit()           │
world        │ withdraw()          │
             │ getBalance()        │
             │                     │
             │ ─────────────────── │
             │ balance             │
             │ validation rules    │
             │ internal logic      │
             └─────────────────────┘
                       ↑
                Encapsulation
```

The outside world interacts through a controlled interface rather than manipulating the internals directly.

## Interview-Level Takeaways

- **Encapsulation = bundle state + behavior and control access to the internals.**
- `private` is a **mechanism**, not the complete definition of encapsulation.
- Encapsulation protects **invariants**.
- Do not blindly generate getters/setters for everything.
- Prefer meaningful operations when they allow the object to enforce its own rules.
- Encapsulation reduces coupling to internal representation.
- Access modifiers are important tools for achieving encapsulation.
- Leaking mutable internal collections can break effective encapsulation.
- **Encapsulation ≠ immutability.**
- **Data hiding is part of encapsulation, but not synonymous with it.**
- Encapsulation and abstraction are related but solve different problems.

> **Key sentence:** Encapsulation controls access to how an object's state is represented and changed; it allows the object to protect its own invariants and hide implementation details.
