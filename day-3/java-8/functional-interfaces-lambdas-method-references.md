# Functional Interfaces, Lambdas & Method References

These three concepts form a closely related Java 8 feature set:

```text
Functional Interface
       ↓
   defines the contract
       ↓
     Lambda
       ↓
   provides implementation
       ↓
 Method Reference
       ↓
shorter syntax when an existing method already fits
```

---

## 1. Functional Interfaces

A **functional interface** is an interface with exactly **one abstract method** (SAM — Single Abstract Method).

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}
```

The `@FunctionalInterface` annotation is not what makes the interface functional. It tells the compiler that the interface is intended to have exactly one abstract method and asks the compiler to enforce that constraint.

This is invalid:

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
    int subtract(int a, int b); // ❌
}
```

### Default and Static Methods Don't Count

A functional interface can still contain default and static methods:

```java
@FunctionalInterface
interface Calculator {

    int calculate(int a, int b);  // abstract — counts

    default void log() {
        System.out.println("calculating");
    }

    static void info() {
        System.out.println("Calculator");
    }
}
```

Only the abstract method counts toward the functional-interface requirement.

---

## 2. Why Functional Interfaces?

Before lambdas, implementing a small piece of behavior often required an anonymous class:

```java
Calculator addition = new Calculator() {
    @Override
    public int calculate(int a, int b) {
        return a + b;
    }
};
```

Functional interfaces provide the target type for a lambda, allowing the same behavior to be expressed much more concisely:

```java
Calculator addition = (a, b) -> a + b;
```

The relationship is:

```text
Functional Interface
        ↓
defines one abstract operation
        ↓
Lambda provides its implementation
```

---

## 3. Lambdas

A lambda is a concise way of providing an implementation for a functional interface's abstract method.

Given:

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}
```

we can write:

```java
Calculator addition = (a, b) -> a + b;
```

Then:

```java
int result = addition.calculate(10, 20);
```

The lambda:

```java
(a, b) -> a + b
```

is the implementation of:

```java
calculate(int a, int b)
```

---

## 4. Lambda Syntax

General form:

```java
(parameters) -> expression
```

Examples:

```java
(a, b) -> a + b
x -> x * 2
() -> System.out.println("Hello")
```

### Expression Body

A single expression can be used directly:

```java
x -> x * 2
```

If the functional method returns a value, the expression's result is returned automatically.

### Block Body

A lambda can also have a block body:

```java
x -> {
    int result = x * 2;
    return result;
}
```

With a block body, an explicit `return` is needed when the functional method has a return value.

---

## 5. Target Typing

Java can infer the parameter types of a lambda from its **target functional-interface type**.

For example:

```java
Calculator addition = (a, b) -> a + b;
```

The compiler sees:

```java
int calculate(int a, int b);
```

and therefore knows:

```text
a → int
b → int
return → int
```

You could explicitly write:

```java
Calculator addition = (int a, int b) -> a + b;
```

but normally this is unnecessary.

The important mental model is:

```text
Lambda
   ↓
needs a target functional interface
   ↓
compiler knows parameter and return types
```

---

## 6. Lambdas Don't Exist in Isolation

In Java, a lambda is interpreted in the context of a target functional-interface type.

For example:

```java
Calculator calc = (a, b) -> a + b;
```

The compiler knows what the lambda means because it knows `Calculator` and its single abstract method.

Similarly:

```java
Runnable task = () -> System.out.println("Hello");
```

means the lambda implements `Runnable.run()`.

Think:

```text
Lambda
   ↓
target functional interface
   ↓
compiler knows what operation the lambda represents
```

---

## 7. Built-in Functional Interfaces

Java provides common functional interfaces in `java.util.function`.

The four most important ones for backend development and Streams are:

### `Predicate<T>`

Takes a `T` and returns a `boolean`.

```java
Predicate<Integer> isEven = x -> x % 2 == 0;
```

Conceptually:

```java
boolean test(T value)
```

### `Function<T, R>`

Takes a `T` and produces an `R`.

```java
Function<String, Integer> length =
        s -> s.length();
```

Conceptually:

```java
R apply(T value)
```

### `Consumer<T>`

Takes a `T` and returns nothing.

```java
Consumer<String> printer =
        s -> System.out.println(s);
```

Conceptually:

```java
void accept(T value)
```

### `Supplier<T>`

Takes no arguments and produces a `T`.

```java
Supplier<Double> random =
        () -> Math.random();
```

Conceptually:

```java
T get()
```

### Quick Reference

```text
Predicate<T>  → T → boolean
Function<T,R> → T → R
Consumer<T>   → T → void
Supplier<T>   → () → T
```

These interfaces are heavily used by the Java Streams API.

---

## 8. Passing Behavior With Lambdas

Functional interfaces allow behavior to be passed into methods rather than hard-coding the behavior.

For example:

```java
static void process(
        List<Integer> numbers,
        Predicate<Integer> condition) {

    for (Integer number : numbers) {
        if (condition.test(number)) {
            System.out.println(number);
        }
    }
}
```

Different callers can supply different behavior:

```java
process(numbers, x -> x > 10);
process(numbers, x -> x % 2 == 0);
process(numbers, x -> x < 0);
```

The method stays the same while the behavior changes.

This is one of the major practical benefits of functional interfaces and lambdas.

---

# Method References

## 9. What Is a Method Reference?

A method reference is a concise way of expressing a lambda when an existing method already provides the required behavior.

For example:

```java
names.forEach(name -> System.out.println(name));
```

can become:

```java
names.forEach(System.out::println);
```

The method reference means:

> Use this existing method as the implementation of the required functional operation.

A useful transformation to recognize is:

```text
x -> someObject.someMethod(x)
        ↓
someObject::someMethod
```

---

## 10. Four Common Forms of Method References

### 1. Static Method

Syntax:

```java
ClassName::staticMethod
```

Example:

```java
Function<String, Integer> parser =
        Integer::parseInt;
```

Equivalent lambda:

```java
Function<String, Integer> parser =
        s -> Integer.parseInt(s);
```

---

### 2. Instance Method on a Specific Object

Syntax:

```java
object::instanceMethod
```

Example:

```java
String prefix = "Hello ";

Function<String, String> greet =
        prefix::concat;
```

Equivalent lambda:

```java
Function<String, String> greet =
        s -> prefix.concat(s);
```

Here `prefix` is the specific object whose method is being referenced.

---

### 3. Instance Method on an Arbitrary Object of a Type

Syntax:

```java
ClassName::instanceMethod
```

Example:

```java
Function<String, String> upper =
        String::toUpperCase;
```

Equivalent lambda:

```java
Function<String, String> upper =
        s -> s.toUpperCase();
```

The important distinction is:

```text
prefix::concat
        ↓
specific object

String::toUpperCase
        ↓
method is called on whichever String is supplied as the lambda argument
```

This form is especially common with Streams.

---

### 4. Constructor Reference

Syntax:

```java
ClassName::new
```

Example with a no-argument constructor:

```java
Supplier<User> creator =
        User::new;
```

Equivalent lambda:

```java
Supplier<User> creator =
        () -> new User();
```

For a constructor that takes an argument:

```java
Function<String, User> creator =
        User::new;
```

Equivalent lambda:

```java
Function<String, User> creator =
        name -> new User(name);
```

---

## 11. Target Typing Also Applies to Method References

The functional interface tells Java what kind of method reference is required.

For example:

```java
Function<String, Integer> parser =
        Integer::parseInt;
```

The target type says:

```text
Input  → String
Output → Integer
```

So Java can select the compatible `parseInt` method.

Similarly:

```java
Consumer<String> printer =
        System.out::println;
```

provides the context:

```text
Input → String
Output → void
```

The functional interface provides the context needed to interpret the method reference.

---

## 12. Lambda vs Method Reference

If a lambda simply delegates to an existing method, a method reference may make it shorter and clearer.

```java
users.forEach(
    user -> System.out.println(user)
);
```

becomes:

```java
users.forEach(
    System.out::println
);
```

Another example:

```java
names.stream()
     .map(name -> name.toUpperCase());
```

becomes:

```java
names.stream()
     .map(String::toUpperCase);
```

A method reference is not fundamentally more powerful than the equivalent lambda. It is primarily a concise expression of the same behavior when an existing method already matches the functional operation.

---

## 13. Complete Example

```java
@FunctionalInterface
interface Transformer {
    String transform(String value);
}
```

A lambda implementation:

```java
Transformer upper =
        name -> name.toUpperCase();
```

The same implementation as a method reference:

```java
Transformer upper =
        String::toUpperCase;
```

Using it:

```java
List<String> names = List.of("Tanay", "rahul", "Amit");

for (String name : names) {
    System.out.println(upper.transform(name));
}
```

The conceptual chain is:

```text
Functional Interface
        │
        │ defines
        ▼
transform(String)
        │
        │ implemented by
        ▼
Lambda
name -> name.toUpperCase()
        │
        │ can be shortened to
        ▼
String::toUpperCase
```

---

## 14. Functional Interface + Default/Static Methods

A functional interface can contain default and static methods without losing its functional-interface status:

```java
@FunctionalInterface
interface Validator {

    boolean validate(User user);

    default void log() {
        System.out.println("Validating...");
    }

    static Validator alwaysValid() {
        return user -> true;
    }
}
```

Only the abstract method counts:

```text
validate() → abstract  ← counts
log()      → default   ← doesn't count
alwaysValid() → static ← doesn't count
```

Default methods are a related Java 8 interface feature and are covered separately from the lambda mechanics.

---

## Core Mental Model

```text
Functional Interface
        ↓
One abstract method
        ↓
Provides target type
        ↓
Lambda
        ↓
Implementation of that operation
        ↓
Method Reference
        ↓
Concise form when an existing method already matches
```

### Functional Interface

> An interface with exactly one abstract method.

```java
@FunctionalInterface
interface Foo {
    void doSomething();
}
```

### Lambda

> A concise implementation of a functional interface's abstract method.

```java
Foo foo = () -> System.out.println("Hello");
```

### Method Reference

> A concise way to use an existing method when it matches the required functional operation.

```java
Foo foo = this::doSomething;
```

---

## Interview-Level Takeaways

- A functional interface has exactly **one abstract method**.
- `@FunctionalInterface` asks the compiler to enforce the intended functional-interface contract.
- Default and static methods do not count toward the one-abstract-method requirement.
- Lambdas provide concise implementations of functional interfaces.
- Lambda parameter types can usually be inferred from the target functional-interface type.
- A lambda needs a target functional-interface type to give it meaning in Java.
- Know the common `java.util.function` interfaces: `Predicate`, `Function`, `Consumer`, and `Supplier`.
- Method references are concise alternatives to certain lambdas when an existing method already matches the required operation.
- Know the four common method-reference forms:
  - `ClassName::staticMethod`
  - `object::instanceMethod`
  - `ClassName::instanceMethod`
  - `ClassName::new`
- Method references also rely on the target functional-interface type to determine the required method signature.

> **Core idea:** Functional interfaces define a single behavior contract, lambdas provide that behavior concisely, and method references provide an even shorter form when an existing method already fits.
