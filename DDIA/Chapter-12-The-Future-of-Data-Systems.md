# Chapter 12 - The Future of Data Systems

CREATED 2022/03/07 - Conclusion chapter of the whole previous chapters

READ 2022/04/10

## Data Integration

### Combing Specialized Tools BY Deriving Data

e.g. integrate an OLTP database with a full-text search index in order to handle queries for arbitrary keywords

Change Data Capture (CDC) (page 454) - Replicating changes from a database to some derived data systems.

Total Order Broadcast (page 384) & Consensus - We want a mechanism for multiple nodes to share the work of ordering the events.

**Atomic Commit and Two-Phase Commit 2PC** (page 354) - provides atomic commit in a distributed database. To ensure that either all nodes commit or all nodes abort. It includes write data, prepare and commit or abort those three phases.

Two-Phase Locking 2PL (page 257) - provides serializable isolation.

Linearizability (page 324)

Read your own writes (page 162)

Ordering events to capture causality:

Sequence Number Ordering (page 343) - logical timestamps can provide total ordering without coordination.

Reference the event identifier to record the casual dependency

Atomic Conflict Resolution (page 174)

### Batch and Stream Processing

Spark performs stream processing on top of a batch processing engine by breaking the stream into **micro batches**.

Apache Flink performs batch processing on top of a stream processing engine.

**gradual evolution** - We can maintain the old schema and the new schema side by side as two independently derived views onto the same underlying data. Then we can start shift a small number of users to the new view.

## Unbundling Databases

Unix and relational databases

### Composing Data Storage Technologies

* Secondary Indexes

* Materialized views - a kind of pre-computed cache of query results (e.g. Data Cubes and Materialized views page 101)

* Replication Logs - keep copies of the data on other nodes up to date (page 158)

* Full-text search indexes - allow keyword search in text

CREATE INDEX is similar to setting up a new follower replica. Scan over the snapshot of the database and process the backlog of writes.

Atomic commit revisited (page 487)

The advantage of log-based integration is loose coupling between the various component:

* async event streams make the system as a whole more robust to outages

* unbundling data systems allow different software components and services to be developed, improved and maintained individually.

### Designing Applications Around Dataflow
