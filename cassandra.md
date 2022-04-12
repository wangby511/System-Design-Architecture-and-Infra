# Cassandra

## Write in Cassandra

Partitioning Key — each table has a Partitioning Key. It helps with determining which node in the cluster the data should be stored.

Commit Log — the transactional log. It’s used for transactional recovery in case of system failures. It’s an append only file and provides durability.

Memtable — a memory cache to store the in memory copy of the data. Each node has a memtable for each CQL table. The memtable accumulates writes and provides read for data which are not yet stored to disk.

SSTable — Actual files on disk and immutable.

Compaction — the periodic process of merging multiple SSTables into a single SSTable. It’s primarily done to optimize the read operations.

## How is data written in Cassandra?

1 Cassandra appends writes to the commit log on disk. The commit log receives every write made to a Cassandra node and these durable writes survive permanently even if power fails on a node.

2 Cassandra also stores the data in a memory structure called **memtable** and to provide configurable durability. The Memtable is a **write-back** cache of data partitions that Cassandra looks up by key.

3 The memtable stores writes in a sorted order until reaching a configurable limit and then it is flushed to a **sorted strings table** called **SSTable**. Writes are atomic at row level. Meaning that all columns are written or updated, or none are.

## Reads in Cassandra

Bloom Filters — helps to point if a key may exist in its corresponding SSTable.

## How to read data in Cassandra?

1 Cassandra first checks if the in-memory memtable cache contains the data. Memtable is an in memory read/write cache for each column family.

2 If not found, Cassandra will check Bloom Filters and then will read all SSTables.

3 SSTable returns the result of dataset and returns to the client.

## CAP Theorem

Consistency — all nodes see the same data at the same time.

Availability — a guarantee that every request receives a response about whether it was successful or failed.

Partition tolerance — the system continues to operate despite arbitrary message loss or failure of part of the system.

Cassandra is an **AP** system meaning it’s more important to be available and partition tolerant.

## Pros and Cons

Cassandra has quite a few benefits:

A) no single point of failure. B) read fast.

Potential issues:

A) update/delete a little slow as it needs to update/delete information from multiple nodes. B) low consistency.

## Reference

[1] <https://allaboutreact.medium.com/study-guide-cassandra-data-consistency-496e5bf9cadb>

[2] <https://1o24bbs.com/t/topic/13877>
