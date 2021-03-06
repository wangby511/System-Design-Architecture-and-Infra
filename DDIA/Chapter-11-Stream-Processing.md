# Chapter 11 - Stream Processing

CREATED 2022/01/23 BEGIN READ 2022/02/28

Unlike Chapter 10 (Batch Processing), in which the input is bounded with known/finite/**unbounded** size (the inputs and outputs of a job are files), a lot of data is unbounded.

## Transmitting Event Streams

In a stream processing context, a record is called **event**, generated by a producer and processed by one of multiple consumers. Related events are grouped together into a **topic or stream**.

### Messaging Systems

As with databases, **durability** requires some combination of writing to disk and/or replication.

A strong **reliability** guarantees that failed tasks are automatically retried and partial output from failed tasks is automatically discarded.

Message **broker**s - Producers write messages to the broker and consumers receive them by reading them from the broker.

Load balancing & Fan out - To one of the consumers & To all consumers.

**Acknowledgements** - A client must tell the broker that it has finished processing a message so that the broker can remove it from the queue.

### Partitioned Logs

A **topic** can be defined as a group of partitions that all carry messages of the same type. Each partition maintains an offset. Messages within the same partition are totally ordered and are delivered to the same node.

Guarantee the fixed order - make special key as the partitioned key like userID.

An **offset** in each partition, is like a log sequence number, tells where the consumers have processed at.

Log is divided into **segments**. The log implements a bounded-size buffer or ring buffer, which can be very large. Every message is written to disk.

## Databases and Streams

### Keeping Systems in Sync

dual writes - concurrency/race conditions may lead to inconsistent data.

To ensure both succeed or both fail, we have to use **Atomic Commit** or **Two-Phase Commit**. Two leaders in this case: the database and the search index. We could make one as leader and the other(s) as follower(s).

### Change Data Capture (CDC)

Process of observing all data changes written to a database and extracting them in a form in which they can be applied to other systems. E.g. Capture all the changes in the order written to one database(leader) and apply to search index(follower) in the same order. Therefore, the derived systems also have an accurate copy of the data.

**Log consumers** - derived data systems

**async** - change data capture is usually async. It has the downside that all the issues of replication **lag apply**.

**Initial snapshot** - Take a snapshot with **a known position or offset** in the log.

Also for log, we can do **log compaction** by merging the same key and only keeping the most recent version for each key, discarding overwritten versions. This compaction and merging process can run in the background.

### Event Sourcing

Unlike the change data capture, where the log of changes is extracted from the database from a low level, the event sourcing is modeled at a higher **application level**: the intent of a user action, not the mechanics of the state update. It is more meaningful to record the user's actions as immutable events and the event store is **append-only**.

Only after the validation is successful and the **command** is accepted, it becomes an **event** which is durable and immutable. At the point when the event is generated, it becomes a **fact**.

### States, Streams and Immutability

The **mutable state** and an **append-only log of immutable events** do not contradict with each other. The log of all changes, the **changelog**, represents the evolution of state over time.

Advantage - Immutable events can capture more information than just the current state. With an **append-only log of immutable events**, it is mush easier to diagnose.

## Processing Streams

Write to somewhere else, push to some users or produce one or more output streams.

A stream never ends so sort-merge joins can not be used. Fault-tolerance mechanisms must also change.

### Uses of Stream Processing

Monitoring purposes like Fraud detection, Trading system, Manufacture system and Intelligence system.

* Complex event processing (CEP) - allows to specify rules to search for certain patterns of events in a stream.

* Stream analytics - the probabilistic algorithms are merely an optimization.

* Maintaining materialized views

* Search on streams - a need to search for individual events based on complex criteria. E.g. ElasticSearch.

* Message passing and RPC - Distributed RPC allows user queries to be farmed out to a set of nodes that also process event streams, then aggregated and sent back.

### Reasoning About Time

Event time is not equal to processing time. Moreover, message delays can also lead to unpredictable ordering of messages. Some events could be delayed due to a network interruption.

If we want to count the number of events in a fixed time window, we need to either do 1) Ignore the straggler events 2) Publish a correction.

The timestamp on the events should be the time at which the user interaction occurred (the mobile device's local clock instead of the time on the server side), but it can be trusted fully due to wrong time. We can add offset to calibrate it.

Types of window:

* Tumbling window - window of fixed length like 10:03:00 to 10:03:59

* Hopping window - window of fixed length but they can overlap. E.g. 5 minute window starting from 10:03:00 to 10:07:59 or 10:04:00 to 10:08:59

* Sliding window - keep a buffer of events sorted by time and remove those old expired events. E.g. 10:03:39 to 10:08:12

* Session window - a user is active during some period of time. No fixed duration.

### Stream Joins

* Stream-stream joins - A stream processor needs to maintain **state**, e.g. grouping the events indexed by session ID.

* Stream-table joins - E.g. The input is a stream of activity events containing a user ID. The output is a stream of activity events in which the user ID has been augmented with profile information about user. **enriching**. The local copy on the stream processor also needs to be updated with the latest change.

* Table-table joins - E.g. two tables like tweets and follows to generate Twitter timeline. Both input streams are database changelogs. In this case, every change on one side is joined with the latest state of the other side.

**slowly changing dimension (SCD)** - If the ordering of events across streams is undetermined, the join becomes nondeterministic. It can be fixed by using a **unique identifier for a particular version of the joined record**.

### Fault Tolerance

Since a stream processing is a long-running and produces output continuously, we can not simply discard all outputs.

* Micro-batching - break down stream into small blocks(tumbling window) and treat each block as a minimal batch.

* Checkpoint - generate rolling checkpoints of state and write them to durable storage.

* Atomic commit revisit - process several input messages within a single transaction

* Idempotence - an idempotent operation is one that you can perform multiple times, and it has the same effect as if you performed it only once. It is an effective way of achieving exactly-once semantics.

* Rebuild state after a failure - store the state in another machine for replication, keep state local to the stream processor and replicate it periodically, rebuilt from the input streams corresponding to the window. Trade-off among disk access latency, network delay and network bandwidth.
