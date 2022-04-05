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

### Arvo

Another binary encoding format. We must use the compatible schemas in the code (between the writer's schema and the reader's schema).

- Large file with lots of records, include the writer's schema at the beginning of the file.

- Database with individually written records, include the version number of the schema at the beginning of the file.

- Sending records over a network connection

### The Merits of Schemas

- Binary encodings based on schemas can be more compact than the various "binary JSON" variants.

- The schema is a valuable form of documentation.

- Keeping a database of schemas allows us to check forward and backward compatibility.

- The ability to generate code from the schema is useful.

## Models of Dataflow

Compatibility is a relationship between one process that encodes the data and another process that decodes it.

Via databases.

Via service calls.

Via async message passing.

### Dataflow Through Database

One process writing to the database encodes the data and another process reading from the database decodes it.

Notice: When an older version of the application updates data previously written by a newer version of the application, data may be lost if you are not careful. Thus, forward compatibility is also often required for databases.

data outlives code - The old data is still in the database unless the new version of application rewrites it.

Schema evolution thus allows the entire database to appear as if it was encoded with a single schema.

### Dataflow Through Services: REST and RPC

two roles - clients and servers.

One process sends a request over the network to another process and expects a response as quickly as possible.

service oriented architecture (SOA) - later rebranded as **micro-services architecture**.

**remote procedure call (RPC)** - The RPC model tries to make a request to a remote network service look the same as calling a function or method in your programming language, within the same process.

But unlike the local function call, a network request is unpredictable, having no way of knowing whether the request got through or not, much slower and it needs to consider de-duplication/idempotence, deal with large objects in parameters. Not all languages have the same types and it needs translation.

gRPC is an RPC implementation using Protocol Buffers and it supports streams.

A RESTful API has significant advantages - it is good for experimentation and debugging through a web browser or the command-line tool curl.

Service compatibility needs to be maintained for a long time.

### Message-Passing Dataflow

The message is sent via **message broker** - a message queue or message-oriented middleware.

advantages: act as a buffer, redeliver messages if failed, no need to know the IP address and port number of the recipients, send to several recipients, decouple senders from recipients.

one-way direction and asynchronous

## Summary

During rolling upgrade, it is important that all data flowing around the system is encoded in a way that provides backward compatibility(new code can read old data) and forward compatibility(old code can read new data).
