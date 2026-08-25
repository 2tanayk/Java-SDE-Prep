# File Encoding

Files on disk ultimately contain **bytes**, while Java applications usually want to work with text as **characters**. A character encoding defines how characters/code points are represented as bytes and how those bytes are decoded back into characters.

```text
File bytes
   ↓
  UTF-8
   ↓
Characters
```

When writing, the reverse happens:

```text
Characters
   ↓
  UTF-8
   ↓
File bytes
```

## Unicode vs UTF-8

**Unicode** defines characters/code points.

Examples:

```text
₹  → U+20B9
😀 → U+1F600
```

**UTF-8** is an encoding that represents Unicode characters/code points as bytes.

```text
₹
 ↓
U+20B9
 ↓ UTF-8
E2 82 B9
```

UTF-8 is variable-width and uses 1–4 bytes per Unicode code point.

```text
A   → 1 byte
₹   → 3 bytes
你   → 3 bytes
😀  → 4 bytes
```

The important distinction is:

> **Unicode tells us what characters/code points exist; UTF-8 tells us how to encode them as bytes.**

## `StandardCharsets`

Java provides predefined charset constants:

```java
StandardCharsets.UTF_8
StandardCharsets.US_ASCII
StandardCharsets.ISO_8859_1
```

When the encoding matters, explicitly specifying the charset is a good practice.

For example:

```java
Path path = Path.of("data.txt");

String content =
    Files.readString(
        path,
        StandardCharsets.UTF_8
    );
```

And writing:

```java
Files.writeString(
    path,
    "Hello ₹",
    StandardCharsets.UTF_8
);
```

## `InputStreamReader`

`FileInputStream` is byte-oriented:

```text
FileInputStream
    ↓
bytes
```

`InputStreamReader` is a **bridge from the byte world to the character world**. It consumes bytes from an `InputStream` and decodes them using a specified charset.

```java
Reader reader =
    new InputStreamReader(
        new FileInputStream("data.txt"),
        StandardCharsets.UTF_8
    );
```

Conceptually:

```text
File
 ↓
FileInputStream
 ↓
raw bytes
 ↓
InputStreamReader
 ↓
UTF-8 decoding
 ↓
characters
```

`InputStreamReader` extends `Reader`, so it provides the character-oriented `Reader` API while using an `InputStream` internally as its byte source.

## Why Wrap `FileInputStream` in `InputStreamReader`?

Each layer has a separate responsibility:

```text
FileInputStream
     ↓
"Give me bytes from this file"

InputStreamReader
     ↓
"Decode those bytes as UTF-8 characters"
```

This is an adapter/bridge between two APIs:

```text
InputStream API
      ↓
  byte-oriented
      ↓
InputStreamReader
      ↓
character-oriented Reader API
```

## `FileReader` vs `InputStreamReader`

`FileReader` is a convenient `Reader` for reading a file as characters:

```java
FileReader reader = new FileReader("data.txt");
```

Conceptually, it hides the byte-stream-to-character conversion that is needed underneath.

`InputStreamReader` is more flexible because its source can be **any `InputStream`**, not just a file, and it allows the charset to be explicitly specified:

```java
new InputStreamReader(
    inputStream,
    StandardCharsets.UTF_8
);
```

For production code where encoding matters, prefer an API where the charset is explicit.

## `OutputStreamWriter`

`OutputStreamWriter` is the reverse bridge. It converts Java characters into bytes using a charset and writes those bytes to an `OutputStream`.

```java
OutputStreamWriter writer =
    new OutputStreamWriter(
        new FileOutputStream("data.txt"),
        StandardCharsets.UTF_8
    );
```

Conceptually:

```text
Java characters
      ↓
OutputStreamWriter
      ↓
UTF-8 encoding
      ↓
bytes
      ↓
FileOutputStream
      ↓
File
```

## Combining with Buffering

These layers can be composed:

```java
BufferedReader reader =
    new BufferedReader(
        new InputStreamReader(
            new FileInputStream("data.txt"),
            StandardCharsets.UTF_8
        )
    );
```

Each layer adds a different responsibility:

```text
BufferedReader
    ↓
buffering + readLine()
    ↓
InputStreamReader
    ↓
UTF-8 bytes → characters
    ↓
FileInputStream
    ↓
File → raw bytes
```

Similarly, for writing:

```java
BufferedWriter writer =
    new BufferedWriter(
        new OutputStreamWriter(
            new FileOutputStream("data.txt"),
            StandardCharsets.UTF_8
        )
    );
```

## Why Explicit Encoding Matters

Suppose one application writes UTF-8 bytes and another application interprets those bytes using an incompatible encoding. The same bytes can produce corrupted text.

```text
Same bytes
   │
   ├── interpreted as UTF-8 → correct character
   │
   └── interpreted incorrectly → corrupted text
```

The bytes themselves may be unchanged; the problem is that they are being **decoded using the wrong encoding**.

## Key Takeaways

- Files contain bytes; text processing requires converting between bytes and characters.
- Unicode defines characters/code points; UTF-8 is an encoding of those characters/code points into bytes.
- `StandardCharsets.UTF_8` makes the intended encoding explicit.
- `InputStreamReader` converts bytes from an `InputStream` into characters using a charset.
- `OutputStreamWriter` converts characters into bytes for an `OutputStream`.
- `FileReader` is a convenient file-oriented `Reader`; `InputStreamReader` is the more general byte-to-character bridge.
- `Reader`/`Writer` are character-oriented APIs; `InputStreamReader`/`OutputStreamWriter` bridge them to byte-oriented streams.
- `BufferedReader`/`BufferedWriter` can be layered on top for buffering and convenient operations such as `readLine()`.
