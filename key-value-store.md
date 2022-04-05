# System Design - Key Value Store

FIRST CREATED 2022/01/12 BEGIN 2022/04/02

## Functional Requirements

A non-relational database. Keys can be plain text or hashed values. Values can be strings, lists, objects, etc.

The size of key-value pair is less than 10KB.

## Non-Functional Requirements

Need to confirm which to choose from? availability or consistency.

High and automatic scalability.

Low latency.

## Supported APIs

* get(key)

* put(key, value, expiration)

## CAP Theory

CP systems - supports consistency and partition tolerance while sacrificing availability.

AP systems - supports availability and partition tolerance while sacrificing consistency.

## Data Partitioning

Using consistent hashing to partition data. Both servers and keys are mapped to a **hash ring**. After a key is mapped to a hash position on a hash ring, walk clockwise from the position and choose the first server to store that key-value pair.

Automatic scaling - servers could be added or removed automatically depending on the load.

Heterogeneity - the number of virtual nodes for a server is proportional to the server capacity.

## Data Replication

After a key is mapped to a hash position on a hash ring, walk clockwise from the position and choose the first **N servers** to store that key-value pair **copies**. Replicas are placed in distinct data centers.

## Consistency

N = The number of replicas

W = A write quorum of size W. W = 1 means the coordinator must receive at least one acknowledgement before the write operation is considered as successful.

R = A read quorum of size R.

If W + R > N, strong consistency is guaranteed.

**Eventual Consistency** - Given enough time, all updates are propagated and all replicas are consistent.

Use **vector clock [server, version]** to detect and resolve the conflicts.

## Handling Failures

Use de-centralized failure detection methods like **gossip protocol** to find the node which has failure or is down/offline/unavailable.

## Cassandra

### Write Path

1. The request is persisted on a commit log file.

2. Data is saved in the memory file.

3. Flushed into SSTable on disk when memory cache is full or reaches limit.

### Read Path

1. Check if the data exists in the memory.

2. If not, checks the bloom filter first.

3. Check the SSTable on disk.

## Reference

[1] System Design Interview. Alex Xu. Chapter 6. Design a key-value store.
