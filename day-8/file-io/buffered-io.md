# Buffered I/O

Buffered I/O improves file I/O performance by reducing the number of expensive interactions with the underlying I/O system.

## The Basic Idea

Without buffering, application-level reads and writes can result in many interactions with the underlying stream:

```text
Application
    ↓
   I/O
    ↓
  File
```

A buffered stream introduces an in-memory buffer:

```text
Application
    ↓
  Buffer
    ↓
   File
```

The buffer holds a **chunk of data**, not necessarily a complete line or the entire file. The underlying stream can read a larger chunk into the buffer, and the application can then consume data from memory until the buffer needs to be refilled.

## Buffered Input

For byte-oriented data:

```java
InputStream in =
    new BufferedInputStream(
        new FileInputStream("data.bin")
    );
```

For character-oriented data:

```java
BufferedReader reader =
    new BufferedReader(
        new FileReader("data.txt")
    );
```

`BufferedReader` is particularly useful for text because it provides convenient line-oriented operations such as:

```java
String line = reader.readLine();
```

### Important: Buffer vs Line

The buffer itself does **not** hold one line at a time. It contains a chunk of characters from the underlying stream.

For example, the buffer could conceptually contain:

```text
Tanay,Software Engineer\nRahul,Backend Engineer\nPriya...
```

When `readLine()` is called, `BufferedReader` searches the available buffered data for a line terminator and returns the line:

```text
"Tanay,Software Engineer"
```

The remaining buffered characters stay available for subsequent reads.

Therefore:

> **Buffer ≠ lines. `readLine()` extracts lines from the buffered character data.**

## Buffered Output

For byte-oriented output:

```java
OutputStream out =
    new BufferedOutputStream(
        new FileOutputStream("output.bin")
    );
```

For character-oriented output:

```java
BufferedWriter writer =
    new BufferedWriter(
        new FileWriter("output.txt")
    );
```

Data written through a buffered writer can accumulate in the buffer before being sent to the underlying file.

## `flush()` vs `close()`

### `flush()`

```java
writer.flush();
```

Pushes currently buffered output to the underlying destination but keeps the resource open and usable.

```java
writer.write("Hello");
writer.flush();
writer.write("World"); // still valid
```

### `close()`

```java
writer.close();
```

Finishes using the resource. Closing a buffered output stream/writer also flushes pending output before closing the underlying resource.

After closing, the resource should no longer be used.

Conceptually:

```text
flush()
  ↓
buffer → underlying stream
  ↓
resource remains open
```

```text
close()
  ↓
flush pending data
  ↓
close underlying resource
  ↓
resource is no longer usable
```

## Complete Example

```java
import java.io.*;

public class FileCopyExample {

    public static void main(String[] args) {

        try (
            BufferedReader reader =
                new BufferedReader(
                    new FileReader("employees.txt")
                );

            BufferedWriter writer =
                new BufferedWriter(
                    new FileWriter("employees-copy.txt")
                )
        ) {

            String line;

            while ((line = reader.readLine()) != null) {
                writer.write(line);
                writer.newLine();
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

The flow is:

```text
employees.txt
      ↓
 FileReader
      ↓
BufferedReader
      ↓
  readLine()
      ↓
 application logic
      ↓
BufferedWriter
      ↓
 FileWriter
      ↓
employees-copy.txt
```

The buffer is managed internally; application code does not need to manually manage its contents.

## Why Buffering Improves Performance

The key point is:

> **Buffering does not make the disk itself faster. It reduces the number of expensive underlying I/O operations.**

Instead of repeatedly interacting with the underlying file for small amounts of data, a buffered stream can transfer larger chunks and satisfy many application-level operations from memory.

Buffering also does **not** mean loading the entire file into memory. A large file can still be processed incrementally using a relatively small buffer.

## Decorator Pattern

Java I/O commonly layers functionality by wrapping streams:

```text
FileInputStream
      ↓
BufferedInputStream
```

or:

```text
FileReader
      ↓
BufferedReader
```

The outer wrapper adds behavior without changing the underlying resource. This is an example of the **Decorator pattern**.

## Key Takeaways

- Buffering reduces expensive underlying I/O operations.
- A buffer stores a chunk of data in memory.
- A buffer is not the same thing as a line and does not necessarily contain complete lines.
- `BufferedReader.readLine()` extracts a line from buffered character data.
- `BufferedReader` is useful for line-oriented text processing.
- `flush()` pushes pending output while keeping the resource open.
- `close()` finishes the resource and flushes pending output first.
- Buffering does not mean loading the entire file into memory.
- `BufferedReader`/`BufferedWriter` and `BufferedInputStream`/`BufferedOutputStream` are wrappers around underlying I/O streams.
