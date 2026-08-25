# Directory Traversal

Java NIO.2 provides several ways to inspect directory contents. For this preparation, the important APIs are `Files.list()` and `Files.walk()`. `DirectoryStream` is useful to recognize but does not need deep coverage.

## `Files.list()`

Use `Files.list()` when you want the **immediate contents** of a directory.

```java
Path dir = Path.of("data");

try (Stream<Path> paths = Files.list(dir)) {
    paths.forEach(System.out::println);
}
```

Given:

```text
data/
├── a.txt
├── b.txt
└── subdir/
    └── c.txt
```

The stream contains:

```text
data/a.txt
data/b.txt
data/subdir
```

It does **not** recursively enter `subdir`.

## `Files.walk()`

Use `Files.walk()` when you want **recursive traversal**.

```java
try (Stream<Path> paths = Files.walk(Path.of("data"))) {
    paths.forEach(System.out::println);
}
```

This can produce:

```text
data
data/a.txt
data/b.txt
data/subdir
data/subdir/c.txt
```

You can filter the result, for example to get only regular files:

```java
try (Stream<Path> paths = Files.walk(Path.of("data"))) {
    paths
        .filter(Files::isRegularFile)
        .forEach(System.out::println);
}
```

### Important: Close the Stream

`Files.walk()` returns a `Stream<Path>` backed by filesystem resources, so it should be closed. `try-with-resources` is the appropriate pattern.

## `DirectoryStream`

`DirectoryStream` is an older, lower-level, iterator-like API for iterating over directory entries.

```java
try (DirectoryStream<Path> stream =
         Files.newDirectoryStream(Path.of("data"))) {

    for (Path path : stream) {
        System.out.println(path);
    }
}
```

It can also filter entries using a glob:

```java
try (DirectoryStream<Path> stream =
         Files.newDirectoryStream(Path.of("data"), "*.txt")) {

    for (Path path : stream) {
        System.out.println(path);
    }
}
```

For our preparation, know what `DirectoryStream` is for, but prioritize `Files.list()` and `Files.walk()`.

## Interview-Level Comparison

| API | Recursive? | Returns |
|---|---|---|
| `Files.list()` | No | `Stream<Path>` |
| `Files.walk()` | Yes | `Stream<Path>` |
| `DirectoryStream` | No | `DirectoryStream<Path>` |

### Mental Model

```text
Files.list()
    ↓
"What's directly inside this directory?"
```

```text
Files.walk()
    ↓
"Show me everything underneath this directory."
```

```text
DirectoryStream
    ↓
"Give me an iterator-like way to inspect directory entries."
```

## Key Takeaways

- `Files.list()` inspects only the immediate children of a directory.
- `Files.walk()` recursively traverses a directory tree.
- Both `Files.list()` and `Files.walk()` return streams and should be used with appropriate resource management.
- `Files.walk()` can be combined with stream operations such as `filter()` to select particular files.
- `DirectoryStream` provides a lower-level iterator-like approach and supports filtering.
- For modern Java development, prioritize `Files.list()` and `Files.walk()`.
