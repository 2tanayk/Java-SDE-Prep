# Byte vs Character Streams

The fundamental distinction in Java I/O is:

> **Byte streams deal with raw binary data. Character streams deal with text.**

## Byte Streams

The base classes are:

```java
InputStream
OutputStream
```

Common file implementations are:

```java
FileInputStream
FileOutputStream
```

Byte streams provide access to raw bytes.

```java
InputStream in = new FileInputStream("data.bin");
int value = in.read();
```

`read()` returns an `int`; for byte streams, the value represents a byte in the range `0–255`, while `-1` indicates end-of-file.

Byte streams are appropriate for binary data such as:

- Images
- PDFs
- ZIP files
- Audio/video
- Arbitrary binary files

## Character Streams

The base classes are:

```java
Reader
Writer
```

Common file implementations include:

```java
FileReader
FileWriter
```

Character streams provide a text-oriented abstraction, handling conversion between encoded bytes and characters.

Typical use cases include:

- Text files
- CSV
- JSON
- XML
- Source code
- Logs

## Why Character Streams Exist

Files on disk ultimately contain bytes. Applications, however, usually want to work with text as characters.

For example, the character `₹` cannot be represented by a single byte in UTF-8. Its UTF-8 representation is three bytes:

```text
₹
↓
U+20B9
↓
UTF-8
↓
E2 82 B9
```

A byte stream exposes those raw bytes. A character-oriented API performs the decoding step so the application can work with the text:

```text
File
 ↓
bytes
 ↓
charset decoding
 ↓
characters
 ↓
application
```

When writing text, the reverse happens:

```text
application characters
 ↓
charset encoding
 ↓
bytes
 ↓
File
```

So character streams are not needed because files literally contain characters. They exist because **files contain bytes while applications often want to work with text as characters**.

## Why 0–255 Is Not a Character Limit

A byte contains 8 bits, so it has `2^8 = 256` possible values:

```text
0 → 255
```

This does not mean Java can represent only 256 characters. A byte is simply a unit of raw storage.

ASCII uses one byte for basic characters, but Unicode contains vastly more characters. Encodings such as UTF-8 represent Unicode using a variable number of bytes:

```text
A   → 1 byte
₹   → 3 bytes
你   → 3 bytes
😀  → 4 bytes
```

Multiple bytes can therefore represent one character/code point.

## Byte Stream vs Character Stream

```text
InputStream / OutputStream
        ↓
       bytes
        ↓
binary data / raw data
```

```text
Reader / Writer
        ↓
    characters
        ↓
text + encoding/decoding
```

## Important Mental Model

> **Byte streams provide raw byte access; character streams provide text-oriented access with character encoding/decoding.**

Character streams are therefore an abstraction over the byte-to-character and character-to-byte conversion required when processing encoded text.

## Interview Note

Do not reduce the rule to "text always means `Reader` and binary always means `InputStream`". The deeper distinction is that byte streams expose raw bytes, while character streams provide text-oriented processing with encoding and decoding.

`FileReader` and `FileWriter` are convenient legacy APIs, but when explicit charset control is important, APIs such as `InputStreamReader` and `OutputStreamWriter` allow a charset such as `StandardCharsets.UTF_8` to be specified explicitly. Charset handling is covered separately in the File Encoding topic.
