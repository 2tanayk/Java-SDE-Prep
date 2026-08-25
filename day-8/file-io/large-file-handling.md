# Large File Handling

The key principle for large files is:

> **Do not load the entire file into memory when you can process it incrementally.**

For small files, reading everything into memory can be simple and perfectly reasonable. For large or potentially unbounded files, streaming is safer and more scalable.

## The Problem with Reading Everything

For example:

```java
String content = Files.readString(path);
```

`readString()` loads the entire file into a `String`.

Similarly:

```java
List<String> lines = Files.readAllLines(path);
```

loads all lines into memory, and:

```java
byte[] data = Files.readAllBytes(path);
```

loads the entire file into a byte array.

These APIs are convenient for small files but can cause excessive memory usage for large files.

```text
10 GB file
   ↓
readString()
   ↓
10 GB+ memory usage
   ↓
problem
```

## Streaming / Incremental Processing

For large files, process the data progressively:

```text
File
 ↓
small chunk
 ↓
process
 ↓
small chunk
 ↓
process
 ↓
...
```

This keeps the amount of file data held by the application relatively small.

## `BufferedReader`

For large text files, a `BufferedReader` can process the file line-by-line:

```java
try (BufferedReader reader = Files.newBufferedReader(path)) {

    String line;

    while ((line = reader.readLine()) != null) {
        process(line);
    }
}
```

The application only needs to hold the current line rather than the entire file.

```text
File
 ↓
line
 ↓
process
 ↓
next line
 ↓
process
 ↓
...
```

This is a good approach when the processing requires explicit procedural control.

## `Files.lines()`

NIO also provides a lazy stream of lines:

```java
try (Stream<String> lines = Files.lines(path)) {

    lines.forEach(line -> {
        process(line);
    });
}
```

The important property is that the lines are processed lazily rather than first constructing a `List<String>` containing the entire file.

This works particularly well with Stream operations:

```java
try (Stream<String> lines = Files.lines(path)) {

    lines
        .filter(line -> line.contains("ERROR"))
        .forEach(System.out::println);
}
```

## `readAllLines()` vs `Files.lines()`

This is an important interview comparison.

### `readAllLines()`

```java
List<String> lines = Files.readAllLines(path);
```

Conceptually:

```text
File
 ↓
ALL lines
 ↓
List<String>
 ↓
memory
```

Suitable for small files, but potentially dangerous for huge files.

### `Files.lines()`

```java
try (Stream<String> lines = Files.lines(path)) {
    // process progressively
}
```

Conceptually:

```text
File
 ↓
line
 ↓
process
 ↓
next line
 ↓
process
 ↓
...
```

Better suited to large text files when the processing itself does not accumulate everything into memory.

## Important Trap: A Lazy Stream Can Still Become Large

`Files.lines()` does not guarantee low memory usage if the application deliberately accumulates all results.

For example:

```java
List<String> errors;

try (Stream<String> lines = Files.lines(path)) {
    errors = lines
        .filter(line -> line.contains("ERROR"))
        .toList();
}
```

The stream is lazy, but `toList()` eventually stores all matching lines in memory.

The important principle is:

> **Use streaming APIs and avoid accumulating the entire dataset when processing large files.**

## Large Binary Files

For large binary files, use a byte-oriented stream and process chunks:

```java
byte[] buffer = new byte[8192];

int bytesRead;

while ((bytesRead = input.read(buffer)) != -1) {
    process(buffer, bytesRead);
}
```

The buffer might be only a few kilobytes while the file could be many gigabytes.

```text
buffer = 8 KB
file   = 10 GB
```

The entire file does not need to be loaded into memory.

## `BufferedReader` vs `Files.lines()`

| | `BufferedReader` | `Files.lines()` |
|---|---|---|
| Reads text line-by-line | Yes | Yes |
| Processes incrementally | Yes | Yes |
| Stream operations | No | Yes |
| Explicit loop/control | Yes | No |
| Suitable for large files | Yes | Yes |

Use `BufferedReader` when explicit procedural control is useful. Use `Files.lines()` when the processing naturally fits a Stream pipeline.

## Backend Example

Suppose a Spring Boot service receives a 500 MB CSV upload. Instead of loading the entire upload into memory, the application can process it incrementally:

```text
HTTP request
     ↓
InputStream
     ↓
BufferedReader
     ↓
one line
     ↓
parse
     ↓
persist/process
     ↓
next line
```

This same principle applies to log processing, batch jobs, CSV imports, and other large-file workflows.

## Key Comparisons

```text
Files.readString()
        ↓
entire file → String
        ↓
small text files
```

```text
Files.readAllLines()
        ↓
entire file → List<String>
        ↓
small text files
```

```text
Files.readAllBytes()
        ↓
entire file → byte[]
        ↓
small binary files
```

Versus:

```text
BufferedReader
        ↓
process incrementally
        ↓
large text files
```

```text
Files.lines()
        ↓
lazy Stream<String>
        ↓
large text files
```

```text
InputStream
        ↓
read chunks
        ↓
process incrementally
        ↓
large binary files
```

## Key Takeaway

> **File size should influence your I/O strategy.** If a file is small, loading it entirely can be simpler and reasonable. If it can be large, stream it and process incrementally rather than accumulating the entire file in memory.
