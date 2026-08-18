# ☕ Java Interview Prep: Type Erasure

## 📌 Executive Summary
**Type erasure** is a core mechanism in the Java compiler designed to enforce **backward compatibility** with pre-Java 5 legacy code. Generics in Java are strictly a **compile-time feature** used to guarantee type safety. Once the Java compiler compiles the source code into bytecode, all generic type information is permanently stripped away.

---

## ⚙️ How Type Erasure Works (The 3 Actions)
During compilation, the compiler transforms generic code by performing three distinct steps:

* **Type Parameter Replacement**: Replaces all type parameters with their first bound (e.g., `Number`) or `Object` if unbounded.
* **Implicit Cast Insertion**: Injects explicit, type-safe type casts into the bytecode wherever a value is retrieved.
* **Bridge Method Generation**: Synthesizes hidden "bridge methods" in extended classes to preserve polymorphism and method overriding.

---

## 💻 Code Transformation Cheat Sheet

### 1. Unbounded Generics
Unbounded type parameters default entirely to `Object`.

| Before Compilation (Your Source Code) | After Type Erasure (The Bytecode Equivalent) |
| :--- | :--- |
| `public class Node<T> {` | `public class Node {` |
| `    private T data;` | `    private Object data;` |
| `    public T getData() { return data; }` | `    public Object getData() { return data; }` |
| `}` | `}` |

### 2. Bounded Generics
Bounded type parameters are replaced with their upper bound limit.

| Before Compilation (Your Source Code) | After Type Erasure (The Bytecode Equivalent) |
| :--- | :--- |
| `public class NumericNode<T extends Number> {` | `public class NumericNode {` |
| `    private T data;` | `    private Number data;` |
| `    public T getData() { return data; }` | `    public Number getData() { return data; }` |
| `}` | `}` |

### 3. Automatic Type Casting
The compiler automatically inserts downstream casts to ensure safety.

| Before Compilation (Your Source Code) | After Type Erasure (The Bytecode Equivalent) |
| :--- | :--- |
| `Node<String> node = new Node<>();` | `Node node = new Node();` |
| `String s = node.getData();` | `String s = (String) node.getData();` |

---

## 🚫 Interview Gotchas & Runtime Restrictions
Expect interviewers to probe your understanding using these exact edge cases:

* **Runtime Class Equality**: At runtime, `ArrayList<String>` and `ArrayList<Integer>` share the exact same runtime `Class` object (`ArrayList.class`).
* **No `instanceof` Checks**: You cannot evaluate generic arguments at runtime. `if (list instanceof List<String>)` will trigger a compilation error.
* **No Generic Instance Creation**: Instantiations like `new T()` or `new T[10]` are strictly illegal because the underlying type is not known at runtime.
* **Method Overloading Conflicts**: You cannot overload methods that resolve to identical raw signatures after erasure.
  * *Example:* `void process(List<String> list)` and `void process(List<Integer> list)` will result in a `"has the same erasure"` compiler clash.

---

## 🧠 Advanced Topic: Bridge Methods
Consider a subclass extending a parameterized class:

```java
public class MyNode extends Node<Integer> {
    @Override
    public void setData(Integer data) { super.setData(data); }
}
```

Because `Node.setData` resolves to `setData(Object data)` after erasure, the signatures would no longer match, destroying polymorphism. To fix this, the compiler generates a hidden **bridge method**:

```java
// Synthesized by the compiler behind the scenes
public void setData(Object data) {
    setData((Integer) data);
}
```

---

## 🔥 Recommended Cheat Sheet Keywords
Ensure you mention these high-impact phrases in your answer:
* **Backward compatibility** (Java 1.4 vs Java 5)
* **Compile-time type safety** vs **Runtime type ignorance**
* **Raw types** and **Bridge methods**
* **Bytecode transformation**
