# `java.io.File` API Basics

`File` is Java's legacy API for representing a file or directory path. A `File` object represents a pathname; creating the object does not create anything on the filesystem.

## Creating a `File`

```java
File file = new File("data.txt");
```

This only creates a Java object representing the path. The actual file can be created with:

```java
file.createNewFile();
```

`createNewFile()` returns `true` if a new file was created and `false` if it already existed, and it can throw `IOException`.

## File vs Directory

A `File` object can represent either a file or a directory. The filesystem determines what actually exists at that path.

```java
file.exists();
file.isFile();
file.isDirectory();
```

## File Information

Useful methods include:

```java
file.getName();
file.getPath();
file.getAbsolutePath();
file.length();
file.lastModified();
```

- `getName()` returns the final path component.
- `getPath()` returns the path used to construct the `File`.
- `getAbsolutePath()` resolves the path to an absolute path.
- `length()` returns the file size in bytes for a regular file.
- `lastModified()` returns the last-modified timestamp.

## Listing a Directory

```java
File directory = new File("/tmp");
String[] names = directory.list();
File[] files = directory.listFiles();
```

`list()` returns entry names, while `listFiles()` returns `File` objects.

```java
for (File file : directory.listFiles()) {
    if (file.isFile()) {
        System.out.println("FILE: " + file.getName());
    } else if (file.isDirectory()) {
        System.out.println("DIR: " + file.getName());
    }
}
```

## Creating Directories

```java
new File("documents").mkdir();
new File("a/b/c").mkdirs();
```

- `mkdir()` creates a single directory and requires its parent to already exist.
- `mkdirs()` creates the directory and any missing parent directories.

## Key Takeaway

> `File` is primarily a path abstraction, not a file-content abstraction. Reading and writing file contents is handled by stream/reader/writer APIs and, in modern Java, by NIO.2's `Path` and `Files` APIs.
