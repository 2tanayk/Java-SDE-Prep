# Nested & Inner Classes

A **nested class** is a class declared inside another class. Java has four forms:

- Static nested class
- Member inner class
- Local inner class
- Anonymous inner class

## Static Nested Class

A static nested class belongs logically to the enclosing class but does **not** require an instance of the enclosing class.

```java
class User {
    static class Builder {
        // ...
    }
}
```

It can be instantiated without a `User` object:

```java
User.Builder builder = new User.Builder();
```

A static nested class can directly access static members of the outer class, but cannot directly access its instance members because it has no implicit outer-instance reference.

### Typical Use Case

Use a static nested class when the type is strongly related to the enclosing class but does not depend on a particular enclosing object.

A common example is the Builder pattern. The builder is conceptually part of `User`, but does not belong to an existing `User` instance.

## Member Inner Class

A non-static class declared directly inside another class is a **member inner class**.

```java
class Outer {
    int value = 10;

    class Inner {
        void print() {
            System.out.println(value);
        }
    }
}
```

A member inner class is associated with a particular outer-class instance:

```java
Outer outer = new Outer();
Outer.Inner inner = outer.new Inner();
```

Because it has an enclosing instance, it can directly access the outer object's instance members.

### Typical Use Case

Use an inner class when the inner object genuinely belongs to or operates on a specific outer object.

A classic example is an iterator implemented as an inner class: the iterator operates on the state of one particular collection instance.

Conceptually:

```text
list1
  └── iterator1

list2
  └── iterator2
```

The iterator for `list1` should operate on `list1`, not on some unrelated list.

## `Outer.this`

An inner class can explicitly refer to the enclosing object's `this` using `Outer.this`.

```java
class Outer {
    int x = 10;

    class Inner {
        int x = 20;

        void print() {
            System.out.println(x);             // 20
            System.out.println(this.x);        // 20
            System.out.println(Outer.this.x);  // 10
        }
    }
}
```

`Outer.this` means the `this` reference of the enclosing `Outer` object.

## Local Inner Class

A class declared inside a method or block is a **local class**.

```java
class Outer {
    void process() {
        class Helper {
            void help() {
                System.out.println("Helping...");
            }
        }

        new Helper().help();
    }
}
```

The class is scoped to that method/block and cannot be referred to as a member such as `Outer.Helper`.

Local classes can capture local variables when those variables are final or effectively final, similar to lambdas.

## Anonymous Inner Class

An anonymous class is an unnamed class created inline, commonly for a one-off implementation of an interface or subclass of a class.

```java
Runnable task = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running");
    }
};
```

### Typical Use Case

Use an anonymous class when a small custom implementation is needed only once.

Before Java 8, anonymous classes were commonly used for callbacks and functional interfaces. Lambdas replaced many of those use cases, but anonymous classes remain useful when the target is a class rather than a functional interface or when class-specific behavior/state is required.

## Nested Class vs Inner Class

**Nested class** is the umbrella term for any class declared inside another class.

An **inner class** specifically means a **non-static nested class**.

```java
class Outer {
    static class A {}  // static nested class
    class B {}         // inner class
}
```

Both `A` and `B` are nested classes, but only `B` is an inner class.

## Choosing Between Them

- **Static nested class:** "This type is logically related to the outer class, but does not need an outer instance."
- **Inner class:** "This object belongs to / operates on a particular outer object."
- **Anonymous class:** "I need a small one-off implementation here."

In modern backend Java, static nested classes are generally more common than non-static inner classes. Prefer a static nested class when the enclosing-instance relationship is unnecessary; use a non-static inner class when that relationship is genuinely part of the design.

## Interview-Level Takeaways

- A nested class is a class declared inside another class.
- A static nested class does not require an outer instance.
- A non-static inner class is associated with a particular outer instance.
- Static nested classes can directly access outer static members, but not outer instance members.
- Inner classes can directly access outer instance members.
- `Outer.this` refers to the enclosing outer object.
- Local classes are declared inside methods/blocks.
- Anonymous classes are unnamed one-off implementations.
- A common static nested class use case is `Builder`.
- A common inner-class use case is an iterator or helper that must operate on one particular outer object.
- In modern Java, many anonymous-class use cases for functional interfaces have been replaced by lambdas.
