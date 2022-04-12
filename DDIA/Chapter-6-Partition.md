# Chapter 6 - Partition

CREATED 2022/04/07

break data into partitions, also known as sharding.

## Partitioning and Replication

They are usually combined so that copies of each partition are stored on multiple nodes.

Each partition's leader is assigned to one node, and its followers are assigned to other nodes.

## Partitioning of Key-Value Data

**skewed or hot spot** - unfair data distribution or queries.

### Partitioning by Key Range

Keys are sorted. Assign a continuous range of keys to each partition, from some minimum to some maximum.

### Partitioning by Hash Range

E.g. a 32-bit hash function to make the data uniformly distributed. We can assign each partition a range of hashes where a hash function is applied to each key.

Famous example - Consistent Hashing

Cassandra's SSTables - specify a fixed value for the first column and perform an efficient range scan over the other columns of the key.

**Hybrid approach** - use one key to identify the partition and another part for the sort order.

### Skewed Workloads and Relieving Hot Spots

In the future, data systems may be able to automatically detect and compensate for skewed workloads.

## Partitioning and Secondary Indexes

Two main approaches to partitioning a database with secondary indexes - document-based partitioning and term-based partitioning.

### Partitioning Secondary Indexes by Document

Each partition maintains its own secondary index - a local index

pros - only a single partition needs to be updated on write

cons - **scatter or gather** - send the read query to all partitions and combine all the results.

### Partitioning Secondary Indexes by Term

construct a global index that covers data in all partitions.

pros - make read request to the only partition containing the term it wants

cons - a write may now affect multiple partitions

## Re-balancing Partitions

All these changes call for data and requests to be removed from one node to another.

### Strategies for Re-balancing

Not: hash mod N

Fixed number of partitions - with different nodes assigned to. e.g. 1000 partitions assigned to 10 nodes.

Dynamic partitioning - The large partition can be split and small partitions can be merged. **pre-setting** - HBase and MongoDB allow an initial set of partitions to be configured on an empty database. The number of partitions is proportional to the size of the dataset.

To have the fixed number of partitions per node - e.g. in Cassandra 256 partitions per node by default.

### Operations: Automatic or Manual Re-balancing

both

## Request Routing

**service discovery** - to find which node to connect to.

1 Allow the clients to contact any node 2. Send all the requests to a routing tire first 3. Require the clients to be aware

e.g. Zookeeper keeps track of assignment of partitions to nodes. It keeps track of this cluster metadata.

LinkedIn's Espresso uses Helix.

Cassandra uses gossip protocol.

### Parallel Query Execution

**massively parallel processing(MPP)** - breaks one complex query into a number of execution stages and partitions.
