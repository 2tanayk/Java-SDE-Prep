# String, String Pool, StringBuilder & StringBuffer

## String

`String` is a Java class representing a sequence of characters.

Two properties are especially important:

- `String` is **immutable**.
- Java gives special treatment to String literals through the **String Pool**.

## String Immutability

Once a `String` object is created, its contents cannot be changed.

```java
String s = "hello";
s.concat(" world");

System.out.println(s); // hello
```

`concat()` does not modify the existing String. It creates another String.

To use the result:

```java
s = s.concat(" world");
```

The variable is reassigned; the original String object remains unchanged.

### Why Is String Immutable?

Important benefits include:

- **Safe sharing:** String objects can be safely shared because nobody can modify their contents.
- **String Pool:** Java can reuse identical String literals safely because pooled Strings are immutable.
- **Security:** Strings are frequently used for values such as paths, URLs, class names, and other security-sensitive data; immutability prevents their contents from changing unexpectedly.
- **Hashing:** Strings are excellent `HashMap` keys because their contents cannot change after insertion, so their `equals()`/`hashCode()`-relevant state remains stable.

## `==` vs `equals()` for Strings

For objects:

- `==` compares whether two references point to the **same object**.
- `equals()` compares **logical/content equality** for `String`.

Example:

```java
String a = new String("hello");
String b = new String("hello");

System.out.println(a == b);      // false
System.out.println(a.equals(b)); // true
```

The two references point to separate objects, but their contents are equal.

## String Pool

Java maintains a special pool of String objects for String literals.

```java
String a = "hello";
String b = "hello";
```

Identical literals can refer to the same pooled object:

```text
a ─────┐
       ↓
    "hello"
       ↑
b ─────┘
```

Therefore:

```java
a == b; // true
```

This does **not** mean `==` compares String contents. It is true here because both references can point to the same pooled object.

## String Literal vs `new String()`

Compare:

```java
String a = "hello";
String b = new String("hello");
```

Conceptually:

```text
String Pool
    │
    └── "hello" ← a

Heap
    │
    └── new String("hello") ← b
```

Therefore:

```java
a == b;      // false
a.equals(b); // true
```

### Important: `new String()` Does NOT Make String Mutable

This is a key distinction:

```java
String s = new String("hello");
```

`new` explicitly creates a new String object, but the object is still a **String**, and `String` is immutable.

```text
new String("hello")
        ↓
creates a NEW String object
        ↓
that String is still IMMUTABLE
```

Therefore:

> **Object identity and mutability are separate concepts.**

`new` controls object creation; the class determines the object's mutability.

## How Many Objects Does `new String("hello")` Create?

If `"hello"` is not already in the String Pool, evaluation can involve:

1. A pooled String object for the literal.
2. A separate String object explicitly created by `new String(...)`.

Conceptually:

```text
String Pool
    ↓
"hello"

Heap
    ↓
new String("hello")
```

If `"hello"` is already present in the pool, the literal does not create another pooled object at that point.

Therefore, do not blindly memorize that `new String("hello")` always creates exactly two objects.

The accurate rule is:

> **`new String(...)` explicitly creates a new String object, while the literal is resolved through the String Pool.**

## `intern()`

`intern()` returns the canonical pooled String for the same contents.

```java
String a = new String("hello");
String b = a.intern();
```

Conceptually:

```text
a
↓
Heap String "hello"

b
↓
String Pool "hello"
```

So:

```java
a == b; // false
```

If:

```java
String c = "hello";
```

then:

```java
b == c; // true
```

A useful mental model:

> **`s.intern()` means: "Give me the canonical pooled String for these contents."**

`intern()` does not mutate the original reference.

For example:

```java
String s = new String("hello");
s.intern();
```

` s ` still refers to the separate object. To make the variable refer to the pooled String:

```java
s = s.intern();
```

## String Concatenation

Because String is immutable, modifying text means producing another String.

```java
String s = "Hello";
s = s + " World";
```

The existing String cannot be modified.

For compile-time constant expressions such as:

```java
String s = "Hello" + " World";
```

Java can resolve the concatenation as a compile-time constant.

Do not use the simplistic rule that every concatenation always creates a new String; compile-time constants and runtime concatenation can behave differently.

## Why StringBuilder Exists

Repeated String modification can result in many intermediate String objects because String is immutable.

For example:

```java
String result = "";

for (int i = 0; i < 10000; i++) {
    result += i;
}
```

Conceptually, the result evolves through many intermediate Strings:

```text
""
 ↓
"0"
 ↓
"01"
 ↓
"012"
 ↓
...
```

The old Strings cannot be modified.

For repeated/dynamic text construction, `StringBuilder` is usually a better fit.

## StringBuilder

`StringBuilder` is **mutable**.

```java
StringBuilder sb = new StringBuilder();

sb.append("Hello");
sb.append(" ");
sb.append("World");

String result = sb.toString();
```

`append()` modifies the builder's internal state instead of requiring a new String for every intermediate modification.

A common pattern is:

```java
StringBuilder result = new StringBuilder();

for (...) {
    result.append(something);
}

String finalResult = result.toString();
```

The key reason to use it is:

> **String is immutable; StringBuilder is mutable, so repeated modifications can happen without creating a new String for every intermediate state.**

## StringBuffer

`StringBuffer` is also a mutable character sequence.

The major difference is synchronization:

```text
StringBuilder
→ mutable
→ not synchronized

StringBuffer
→ mutable
→ synchronized methods
```

Therefore:

- Prefer `StringBuilder` by default for ordinary single-threaded text construction.
- Use `StringBuffer` when you specifically need its synchronization semantics for shared mutable access.

Do not assume `StringBuffer` is always better because it is synchronized. Synchronization can add unnecessary overhead when it is not needed.

## String vs StringBuilder vs StringBuffer

| | `String` | `StringBuilder` | `StringBuffer` |
|---|---|---|---|
| Mutable? | ❌ | ✅ | ✅ |
| Synchronized? | Not applicable to mutation because immutable | ❌ | ✅ |
| Repeated modification | Poor fit | Excellent fit | Good fit |
| Typical use | Immutable text values | Building/modifying text | Shared mutable buffer requiring synchronization |
| `append()` mutates object? | N/A | ✅ | ✅ |

A more precise statement than "String is thread-safe" is:

> **String is immutable, so its state cannot be changed concurrently.**

## Compile-Time vs Runtime Concatenation

Consider:

```java
String a = "hello";
String b = "hel" + "lo";

System.out.println(a == b); // true
```

The concatenation can be resolved as a compile-time constant, so the resulting literal can use the String Pool.

But:

```java
String part = "lo";
String b = "hel" + part;
```

now involves a runtime value, so the expression is not treated in the same way as the compile-time constant expression.

The key point is:

> **Do not assume all String concatenation has identical runtime behavior. Distinguish compile-time constant concatenation from runtime concatenation.**

## Mental Model

```text
                         Java Strings
                              |
              +---------------+---------------+
              |                               |
          String                           Builders
              |                               |
        immutable                     +-------+-------+
                                      |               |
                                StringBuilder   StringBuffer
                                      |               |
                                  mutable          mutable
                                  unsync.          synchronized
```

And:

```text
String literal
     ↓
String Pool
     ↓
shared safely because String is immutable
```

For repeated text construction:

```text
Repeated text construction
          ↓
    StringBuilder
          ↓
      append()
          ↓
     toString()
          ↓
       String
```

## Interview-Level Takeaways

- `String` is immutable.
- `==` compares references; `String.equals()` compares contents.
- String literals use the String Pool.
- `new String(...)` explicitly creates a separate String object but **does not make it mutable**.
- Object identity and mutability are separate concepts.
- `intern()` returns the canonical pooled String.
- Do not blindly memorize that `new String("hello")` always creates exactly two objects.
- Repeated String modification can create unnecessary intermediate objects.
- `StringBuilder` is mutable and generally preferred for repeated String construction.
- `StringBuffer` is mutable and synchronized.
- Know the difference between compile-time constant concatenation and runtime concatenation.
- Do not blindly memorize that every String concatenation always creates a new String.

> **Key sentence:** `String` is immutable regardless of whether it was created as a literal or with `new String()`; `new` changes object creation/identity, not the mutability of the `String` class.
