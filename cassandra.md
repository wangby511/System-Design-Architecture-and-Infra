# Cassandra

## Write in Cassandra

Partitioning Key — each table has a Partitioning Key. It helps with determining which node in the cluster the data should be stored.

Commit Log —the transactional log. It’s used for transactional recovery in case of system failures. It’s an append only file and provides durability.

Memtable — a memory cache to store the in memory copy of the data. Each node has a memtable for each CQL table. The memtable accumulates writes and provides read for data which are not yet stored to disk.

SSTable —the final destination of data in C*. They are actual files on disk and are immutable.

## How is data written in Cassandra?

1 Cassandra appends writes to the commit log on disk. The commit log receives every write made to a Cassandra node and these durable writes survive permanently even if power fails on a node.

2 Cassandra also stores the data in a memory structure called Memtable and to provide configurable durability. The Memtable is a write-back cache of data partitions that Cassandra looks up by key.

3 The Memtable stores writes in a sorted order until reaching a configurable limit and then it is flushed to a a sorted strings table called SSTable. Writes are atomic at row level. Meaning that all columns are written or updated, or none are.

## Reads in Cassandra

Row Cache — a memory cache which stores recently read rows (records). It’s an optional component in C*.

Bloom Filters — helps to point if a partition key may exist in its corresponding SSTable.

## How to read data in Cassandra?

1 Cassandra first checks if the in-memory memtable cache contains the data. Memtable is an in memory read/write cache for each column family.

2 If not found, Cassandra will check Bloom Filters.

3 SSTable returns the result of dataset and returns to the client.
