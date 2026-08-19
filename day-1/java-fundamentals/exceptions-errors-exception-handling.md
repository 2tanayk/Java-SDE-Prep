# Exceptions, Errors & Exception Handling

## Throwable Hierarchy

```text
Throwable
├── Exception
│   ├── RuntimeException       → unchecked
│   └── Other Exceptions       → checked
└── Error                      → unchecked, but normally not handled
    ├── OutOfMemoryError
    └── StackOverflowError
```

An **Exception** generally represents an abnormal condition application code may reasonably handle. An **Error** generally represents a serious JVM/system-level problem that application code normally should not try to recover from.

Examples:

- Exceptions: `IOException`, `SQLException`, `IllegalArgumentException`, `NullPointerException`
- Errors: `OutOfMemoryError`, `StackOverflowError`

## `throw` vs `throws`

`throw` actually throws an exception:

```java
if (age < 18) {
    throw new IllegalArgumentException("Age must be >= 18");
}
```

`throws` declares that a method may propagate an exception to its caller:

```java
void readFile() throws IOException {
    // ...
}
```

Mental model: **`throw` = do it now; `throws` = declare the possibility.**

## Exception Handling

Basic structure:

```java
try {
    // risky code
} catch (Exception e) {
    // handle exception
} finally {
    // code that should run when leaving the construct
}
```

When an exception occurs, the remaining statements in the `try` block are skipped and Java looks for a matching `catch`.

Multiple catches are allowed, and more specific types must come before broader types:

```java
try {
    // ...
} catch (IOException e) {
    // ...
} catch (Exception e) {
    // ...
}
```

The reverse order would make the `IOException` catch unreachable.

## Checked vs Unchecked Exceptions

### Checked

Checked exceptions are subclasses of `Exception` that are **not** subclasses of `RuntimeException`.

The compiler requires them to be either handled or declared:

```java
void readFile() throws IOException {
    Files.readString(Path.of("data.txt"));
}
```

or:

```java
try {
    Files.readString(Path.of("data.txt"));
} catch (IOException e) {
    // handle
}
```

Typical examples: `IOException`, `SQLException`.

### Unchecked

Unchecked exceptions are `RuntimeException` and its subclasses. The compiler does not require them to be caught or declared.

Examples:

- `NullPointerException`
- `IllegalArgumentException`
- `IllegalStateException`
- `IndexOutOfBoundsException`

A useful rule:

```text
Exception
├── RuntimeException       → unchecked
└── Other Exception types  → checked
```

`Error` types are also unchecked in the Java language sense, although they are not exceptions and normally should not be caught for application recovery.

## Exception Propagation

If a method does not handle an exception, it propagates up the call stack until a matching handler is found.

```java
void methodA() { methodB(); }
void methodB() { methodC(); }
void methodC() {
    throw new RuntimeException("Something went wrong");
}
```

Conceptually:

```text
methodC → methodB → methodA → caller → matching catch
```

If no matching handler exists, the exception remains uncaught and the thread terminates.

For checked exceptions, each method in the propagation chain must either handle the exception or declare it with `throws`.

## Custom Exceptions

Use domain-specific exceptions instead of generic exceptions when the failure has meaningful application semantics.

```java
class InsufficientBalanceException extends RuntimeException {
    public InsufficientBalanceException(String message) {
        super(message);
    }
}
```

Then:

```java
if (amount > balance) {
    throw new InsufficientBalanceException("Insufficient balance");
}
```

A custom exception can also be checked by extending `Exception` instead of `RuntimeException`.

## Exception Chaining

Exception chaining wraps a lower-level exception in a higher-level exception while preserving the original cause:

```java
try {
    // database operation
} catch (SQLException e) {
    throw new UserRepositoryException(
        "Failed to load user",
        e
    );
}
```

Conceptually:

```text
UserRepositoryException
        ↓ cause
   SQLException
```

Retrieve the original cause with:

```java
exception.getCause();
```

This lets a layer translate an implementation-specific exception while retaining the original diagnostic information.

## `finally`

`finally` is useful for code that should execute when leaving the `try`/`catch` construct, including when the local `catch` does not match an exception or when execution exits through `return`.

```java
try {
    riskyOperation();
} catch (IOException e) {
    handle(e);
} finally {
    cleanup();
}
```

For example, code after the `catch` might never execute if an uncaught exception leaves the method, whereas `finally` still executes as the stack unwinds.

It also executes before a return completes:

```java
int process() {
    try {
        return 10;
    } finally {
        cleanup();
    }
}
```

For `AutoCloseable` resources, prefer **try-with-resources** instead of manually closing resources in `finally`.

## Try-With-Resources

Try-with-resources automatically closes resources implementing `AutoCloseable`:

```java
try (BufferedReader reader =
         new BufferedReader(new FileReader("data.txt"))) {
    System.out.println(reader.readLine());
}
```

It can still have `catch` and `finally` blocks:

```java
try (BufferedReader reader =
         new BufferedReader(new FileReader("data.txt"))) {
    System.out.println(reader.readLine());
} catch (IOException e) {
    System.out.println("Could not read file");
} finally {
    System.out.println("Done");
}
```

The resource-management responsibility is automated; `catch` remains available for exception handling and `finally` for additional unconditional actions.

### Multiple Resources

```java
try (
    Connection connection = getConnection();
    PreparedStatement statement = connection.prepareStatement(sql)
) {
    statement.execute();
}
```

Resources close in **reverse order of initialization**:

```text
Created: connection → statement
Closed:  statement  → connection
```

### Suppressed Exceptions

If the try body throws and `close()` also throws, the try-body exception is the **primary exception** and the close exception becomes **suppressed**.

```java
Throwable[] suppressed = e.getSuppressed();
```

Conceptually:

```text
Primary exception
      |
      └── suppressed exception from close()
```

This is one reason try-with-resources is safer than manual cleanup in `finally`.

## Production Principle

Do not catch exceptions merely to catch them:

```java
try {
    service.process();
} catch (Exception e) {
    e.printStackTrace();
}
```

Catch an exception where you can meaningfully **handle it, recover from it, translate it, or add useful context**.

For example:

```java
try {
    repository.save(user);
} catch (SQLException e) {
    throw new UserRepositoryException(
        "Failed to save user",
        e
    );
}
```

The low-level exception is translated while its original cause is preserved.

## Interview-Level Takeaways

- `Throwable` is the root of Java's exception/error hierarchy.
- `Exception` generally represents application-level abnormal conditions; `Error` generally represents serious JVM/system problems.
- `throw` actually throws; `throws` declares possible propagation.
- Checked exceptions must be handled or declared.
- Unchecked exceptions are `RuntimeException` and its subclasses.
- Exceptions propagate up the call stack until a matching handler is found.
- Custom exceptions communicate domain-specific failures.
- Exception chaining preserves the original cause while adding higher-level context.
- `finally` handles cleanup/actions across early exits and uncaught-exception paths.
- Try-with-resources automatically closes `AutoCloseable` resources.
- Try-with-resources can have `catch` and `finally` blocks.
- Multiple resources close in reverse initialization order.
- A close exception can become a suppressed exception.
- Prefer try-with-resources for resource management.
