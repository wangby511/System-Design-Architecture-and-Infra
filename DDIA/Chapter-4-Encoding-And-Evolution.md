# Chapter 4 - Encoding and Evolution

CREATED 2022/03/24

Compatibility

**Backward compatibility** - Newer code can read data that was written by older code.

**Forward compatibility** - Older code can read data that was written by newer code.

## Formats for Encoding Data

**encoding(serialization or marshalling)** - translation from the in-memory representation to a byte sequence. The reverse is called **decoding**.

### Language-Specific Formats

Different languages have such method.

### JSON, XML, and Binary Variants

JSON, XML, and CSV are textual formats.

But there are some subtle problems:

- Ambiguity around the encoding of numbers.

- Dealing with large numbers.

- JSON and XML don't support binary strings.

- Applications that don't use XML/JSON schemas need to potentially hardcode the appropriate encoding/decoding logic.

- CSV doesn't have any schema.

### Thrift and Protocol Buffers

Thrift has two different binary encoding formats - BinaryProtocol and CompactProtocol.

No field names inside.
