# Exception Handling & Resource Management

File I/O operations can fail because of filesystem and operating-system conditions, so exception handling and proper resource management are essential.

## `IOException`

`IOException` is the general checked exception for many I/O failures.

```java
Path path = Path.of("data.txt");

String content = Files.readString(path);
```

Operations like this can fail because the file does not exist, permissions are insufficient, the path is invalid, or another I/O failure occurs.

## `FileNotFoundException`

`FileNotFoundException` is a more specific exception used by legacy `java.io` APIs.

```java
FileInputStream in =
    new FileInputStream("missing.txt");
```

It is a subclass of `IOException`:

```text
FileNotFoundException
        ↓
    IOException
```

## Why Resources Must Be Closed

Readers, writers, streams, and similar objects hold underlying resources that should be released when finished.

A manual approach such as:

```java
BufferedReader reader =
    new BufferedReader(
        new FileReader("data.txt")
    );

String line = reader.readLine();
reader.close();
```

has a problem: if `readLine()` throws an exception, `close()` may never execute.

That can result in a resource leak.

## Try-with-Resources ⭐⭐⭐

Use try-with-resources for resources that implement `AutoCloseable`:

```java
try (
    BufferedReader reader =
        new BufferedReader(
            new FileReader("data.txt")
        )
) {
    String line = reader.readLine();
}
```

When execution leaves the `try` block, Java automatically closes the resource, including when the operation inside the block throws an exception.

The relevant hierarchy is:

```text
AutoCloseable
     ↑
  Closeable
     ↑
BufferedReader
```

The important idea is:

> If a resource implements `AutoCloseable`, try-with-resources can manage its cleanup automatically.

## Multiple Resources

Multiple resources can be declared in the same try-with-resources statement:

```java
try (
    BufferedReader reader =
        new BufferedReader(
            new FileReader("input.txt")
        );

    BufferedWriter writer =
        new BufferedWriter(
            new FileWriter("output.txt")
        )
) {

    String line;

    while ((line = reader.readLine()) != null) {
        writer.write(line);
        writer.newLine();
    }
}
```

Resources are closed in **reverse order of declaration**:

```text
reader created
writer created

      ↓

writer closed first
reader closed second
```

## Suppressed Exceptions ⭐⭐

A subtle but important behavior occurs when both the main operation and resource cleanup throw exceptions.

For example:

```text
main operation
     │
     └── Exception A  ← primary exception

close()
     │
     └── Exception B  ← suppressed exception
```

Java keeps the exception from the main operation as the primary exception and records the exception from `close()` as a suppressed exception.

Suppressed exceptions can be inspected with:

```java
catch (Exception e) {
    for (Throwable suppressed : e.getSuppressed()) {
        System.out.println(suppressed);
    }
}
```

This is one reason try-with-resources is preferable to manually managing `close()` calls.

## Complete Example

```java
import java.io.*;

public class FileExample {

    public static void main(String[] args) {

        try (
            BufferedReader reader =
                new BufferedReader(
                    new FileReader("employees.txt")
                )
        ) {

            String line;

            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }

        } catch (IOException e) {
            System.out.println(
                "Failed to read file: " + e.getMessage()
            );
        }
    }
}
```

The flow is:

```text
open resource
     ↓
try block
     ↓
success OR exception
     ↓
automatic close()
     ↓
catch exception if applicable
```

## Key Takeaways

- `IOException` represents general I/O failures.
- `FileNotFoundException` is a specific `IOException` used by legacy file APIs.
- File streams/readers/writers must be closed to release underlying resources.
- Prefer **try-with-resources** over manual `close()` calls.
- `AutoCloseable` enables try-with-resources.
- Multiple resources close in **reverse declaration order**.
- If both the main operation and `close()` fail, the main exception is primary and the cleanup exception is suppressed.
