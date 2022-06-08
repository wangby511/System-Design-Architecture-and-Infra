# Common Knowledge for System Design Interview

CREATED 2022/03/22

## Interview Signals

Workable solution

Analysis and communication

Tradeoff/Pros and Cons

Knowledge base

## Common Non-Functional Requirements

Reliability - probability a system will fail in a given period. A distributed system is considered reliable if it keeps delivering its services even when one or several of its software or hardware components fail.

Scalability - capability of a system to handle or manage growing and increasing demands or requests.

Availability - simple measure of the percentage of time that a system, service, or a machine remains operational under normal conditions.

## Common Concepts

**SLA (Service-level Agreement)** - a contract between a service provider and a client. E.g. 99.9% availability.

## Common Component

### Decouple

Use a queue - de-coupling the services and act as a buffer in case throttling.

A common solution is to use/adopt a message queue (Kafka) to decouple producers and consumers. This makes the whole process asynchronous and producers/consumers can be scaled independently.

## Normal Questions

Scaling database - vertical vs horizontal

Leader-follower replica

### Database Sharding - hotkey issue and handling mechanisms

Original purpose - the data becomes very large and it can not be stored on a single machine. So we need to split it into distributed disks for storing and querying.

A shard or service that receives much more data than the others is called **a hotspot**.

This problem can be mitigated by 1) Allocate more nodes/resources to process popular item/key. 2) At the beginning we should choose the right/proper partition key.

Steps of allocating more resources: 1) Apply for extra resources. 2) Extra resources allocated. 3) Split items/events. 4) Merged/Reduced results.

## Delivery Protocols

**Long Polling** - Long polling is technique where the server elects to hold a client connection open for as long as possible, delivering a response only after data becomes available or timeout threshold has been reached.

**WebSocket** - WebSocket is a computer communication protocol which provides full-duplex communication channels. Suitable for bi-directional real-time communication.

**Server-Sent Events(SSE)** - Unlike WebSockets, Server-Sent Events are a one-way communication channel where events flow from server to client only. Server-Sent Events allows browser clients to receive a stream of events from a server over an HTTP connection without polling.

## Process vs Thread

- Processes are independent and contain their own state information. Threads run within a process and share the same state and memory space

- Processes have to communicate with each other using inter-process communication mechanisms. Threads communicate directly because they share the same variables.

- Context switching between threads is faster than between processes.

## How to choose database? - SQL or NoSQL

- What does data look like? Is the data relational? Is it a document or a blob?

- Is the workflow read heavy or write heavy or both heavy?

- Is transaction needed?

- Do the queries rely on many online analytical processing (OLAP) functions?

## Reconciliation

Reconciliation means comparing different sets of data in order to ensure data integrity.

## Consistent Hashing

Consistent hashing is a special kind of hashing algorithm such that when a hash table is re-sized and consistent hashing is used,only k/n keys need to be remapped on average, where k is the number of keys, and n is the number of slots.

Using the same hash function like SHA-1, we map the keys and servers into a circular ring. To determine which server a key is stored on, we go clockwise from the key position on the ring until a server is found. When a server is removed or added, only a small fraction of keys require re-distribution with consistent hashing.

### Virtual Nodes

To further achieve more uniform/balanced distribution, we introduce virtual nodes. A real server can have multiple virtual nodes. With virtual nodes, each server is responsible for multiple partitions.

### Cons

Minimized keys are re-distributed when servers are added or removed.

It is easy to scale horizontally because data are distributed more evenly.

## Lambda Architecture

Lambda architecture is a way of processing massive quantities of data that provides access to batch-processing and stream-processing methods with a hybrid approach [3].

![img](https://databricks.com/wp-content/uploads/2018/12/hadoop-architecture.jpg)

## Reference

[1] <https://medium.com/geekculture/cracking-the-system-design-interview-theory-basics-c57f5326181b>

[2] <https://medium.com/system-design-blog/long-polling-vs-websockets-vs-server-sent-events-c43ba96df7c1>
