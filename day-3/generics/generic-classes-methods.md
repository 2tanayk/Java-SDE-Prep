# Generic Classes & Methods

## Why Do We Need Generics?

Generics allow us to parameterize code with **types** while retaining compile-time type safety.

Without generics, a reusable container might use `Object`:

```java
class Box {
    private Object value;

    public void set(Object value) {
        this.value = value;
    }

    public Object get() {
        return value;
    }
}
```

Usage requires a cast:

```java
Box box = new Box();
box.set("Tanay");

String name = (String) box.get();
```

And an incorrect type can result in a runtime failure:

```java
box.set(123);
String name = (String) box.get(); // ClassCastException
```

Generics move this type checking to compile time.

---

## Generic Class

A generic class declares one or more **type parameters**.

```java
class Box<T> {
    private T value;

    public void set(T value) {
        this.value = value;
    }

    public T get() {
        return value;
    }
}
```

Here, `T` is a type parameter. It is a placeholder for the type that will be supplied when the class is used.

```java
Box<String> stringBox = new Box<>();
Box<Integer> integerBox = new Box<>();
Box<User> userBox = new Box<>();
```

Conceptually:

```text
Box<String>  → T = String
Box<Integer> → T = Integer
Box<User>    → T = User
```

The same class can therefore be reused with different types while remaining type-safe.

### Why This Is Better Than `Object`

With `Object`:

```text
Object → cast → possible runtime failure
```

With generics:

```text
specific type → compiler checks → type-safe code
```

Benefits include:

- Compile-time type safety
- Fewer explicit casts
- Fewer runtime `ClassCastException`s caused by incorrect types
- Reusable classes and APIs
- Clearer APIs and intent

> **Generics move many type-related errors from runtime to compile time.**

---

## What Is `T`?

`T` is not a special Java class. It is simply a **type parameter**.

```java
class Box<T>
```

means:

> When this class is used, the caller will specify what type `T` represents.

For example:

```java
Box<String> box = new Box<>();
```

means that, for this particular use:

```text
T = String
```

---

## Generic Method

Methods can have their own type parameters.

```java
public static <T> void print(T value) {
    System.out.println(value);
}
```

Usage:

```java
print("Tanay");
print(123);
print(true);
```

Java can infer the type for each invocation:

```text
print("Tanay") → T = String
print(123)     → T = Integer
print(true)    → T = Boolean
```

### Why Does `<T>` Appear Before the Return Type?

In:

```java
public static <T> void print(T value)
```

`<T>` declares the **method-level type parameter**.

Compare:

```java
class Box<T>
```

where `T` belongs to the class, with:

```java
<T> void print(T value)
```

where `T` belongs only to that method.

---

## Generic Method With a Return Value

A generic method can use its type parameter for the return type as well:

```java
public static <T> T identity(T value) {
    return value;
}
```

Usage:

```java
String name = identity("Tanay");
Integer number = identity(42);
```

Conceptually:

```text
identity("Tanay")
        ↓
      T = String
        ↓
     returns String

identity(42)
        ↓
      T = Integer
        ↓
     returns Integer
```

The compiler can infer the appropriate type from the invocation context and arguments.

---

## Generic Class vs Generic Method

### Generic class

```java
class Box<T> {
    private T value;

    public T get() {
        return value;
    }
}
```

`T` belongs to the class and is available throughout the class.

```java
Box<String> box = new Box<>();
```

The type is associated with the particular object/type usage.

### Generic method

```java
public <T> T identity(T value) {
    return value;
}
```

`T` exists for that method's invocation.

Different calls can use different inferred types:

```java
identity("Tanay"); // T = String
identity(42);       // T = Integer
```

Mental model:

```text
                    Generics
                       │
          ┌────────────┴────────────┐
          │                         │
          ▼                         ▼
    Generic Class              Generic Method
          │                         │
     class Box<T>             <T> T foo(T x)
          │                         │
          ▼                         ▼
    T belongs to class        T belongs to method
```

---

## Generic Method Inside a Generic Class

A class and a method can each have their own type parameters:

```java
class Box<T> {

    private T value;

    public T get() {
        return value;
    }

    public <U> void print(U other) {
        System.out.println(other);
    }
}
```

Here:

```text
T → belongs to the class
U → belongs to the method
```

They are independent.

For example:

```java
Box<String> box = new Box<>();
box.print(123);
```

Conceptually:

```text
T = String
U = Integer
```

---

## Multiple Type Parameters

A generic class can have multiple type parameters.

```java
class Pair<K, V> {

    private K key;
    private V value;

    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public K getKey() {
        return key;
    }

    public V getValue() {
        return value;
    }
}
```

Usage:

```java
Pair<Long, String> user =
        new Pair<>(101L, "Tanay");
```

Conceptually:

```text
K = Long
V = String
```

This is the same basic idea used by:

```java
Map<K, V>
```

For example:

```java
Map<Long, User>
```

means:

```text
K = Long
V = User
```

---

## Common Generic Naming Conventions

Common conventions include:

```text
T → Type
E → Element
K → Key
V → Value
N → Number
R → Return type
```

Examples:

```java
List<E>
Map<K, V>
```

These are only naming conventions. `T` is not special syntax with a fixed meaning.

Technically, a programmer could use another valid identifier as a type parameter, but conventional names make generic APIs easier to understand.

---

## Generics in the Collections Framework

Generics are used constantly in Java's Collections Framework.

For example:

```java
List<User> users;
Set<String> roles;
Map<Long, User> usersById;
```

Conceptually:

```text
List<E>
   ↓
E = User

Set<E>
   ↓
E = String

Map<K, V>
   ↓
K = Long
V = User
```

So generics are not a niche feature. They are fundamental to everyday Java backend development and the Collections Framework.

---

## Primitive Types Cannot Be Used Directly

Generic type arguments must be **reference types**, not primitive types.

This is invalid:

```java
List<int> numbers; // ❌
```

Use the wrapper type:

```java
List<Integer> numbers; // ✅
```

Java's autoboxing makes ordinary usage convenient:

```java
numbers.add(10);
```

The primitive `int` is automatically boxed into an `Integer`.

Similarly:

```java
Map<Long, User> usersById; // ✅
Map<long, User> usersById; // ❌
```

---

## Core Mental Model

Don't think of generics as complicated `<T>` syntax.

Think:

> **Generics allow us to parameterize code with types while retaining compile-time type safety.**

Generic class:

```java
class Box<T>
```

means:

> Build a `Box` whose contained type will be specified when it is used.

Generic method:

```java
<T> T identity(T value)
```

means:

> This method works with some type `T`, and its input and output use that same type.

---

## Interview-Level Takeaways

- Generics provide **compile-time type safety**.
- They reduce the need for explicit casts and help prevent incorrect-type runtime failures.
- `T` is a type parameter, not a special class.
- A generic class declares its type parameter at the class level: `class Box<T>`.
- A generic method declares its own type parameter before its return type: `<T> T method(T value)`.
- A class type parameter belongs to the class; a method type parameter belongs to that method.
- A generic class and generic method can have independent type parameters, such as `T` and `U`.
- Generic classes and methods can have multiple type parameters, such as `<K, V>`.
- `T`, `E`, `K`, `V`, etc. are conventional names, not special meanings enforced by Java.
- The Collections Framework heavily relies on generics: `List<E>`, `Set<E>`, `Map<K,V>`.
- Generic type arguments cannot be primitives; use wrapper types such as `Integer` instead of `int`.

> **Core idea:** Generics let Java express "this code works with a particular type" while allowing the compiler to enforce that type throughout the API.
