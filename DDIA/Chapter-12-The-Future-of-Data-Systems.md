# Chapter 12 - The Future of Data Systems

CREATED 2022/03/07 - Conclusion chapter of the whole previous chapters

READ 2022/04/10

## Data Integration

### Combing Specialized Tools BY Deriving Data

e.g. integrate an OLTP database with a full-text search index in order to handle queries for arbitrary keywords

Change Data Capture (CDC) (page 454) - Replicating changes from a database to some derived data systems.

Total Order Broadcast (page 384) & Consensus - We want a mechanism for multiple nodes to share the work of ordering the events.

**Atomic Commit and Two-Phase Commit 2PC** (page 354) - provides **atomic commit in a distributed database**. Multiple nodes are involved in a transaction. To ensure that either all nodes commit or all nodes abort. It includes three phases: write data, prepare and commit or abort.

**Two-Phase Locking 2PL** (page 257) - provides serializable isolation. The lock can be either in shared mode (for read) or exclusive mode (for write). After a transaction has acquired the lock, it must continue to hold the lock until the end of the transaction(commit or abort).

**Write Skew** (page 246) - Two transactions read the same objects, and then update some of those objects. Different transactions may update different objects. The **serializable isolation** can prevent this.

Linearizability (page 324) - It makes the database behave like a variable in a single-threaded program.

Read your own writes (page 162) - Let the user submit some data and then view what they have submitted by refreshing the page.

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

Derived data versus distributed transactions (page 492)

Atomic commit revisited (page 487)

The advantage of log-based integration is loose coupling between the various component:

* async event streams make the system as a whole more robust to outages

* unbundling data systems allow different software components and services to be developed, improved and maintained individually.

### Designing Applications Around Dataflow

In a spreadsheet, you can put a formula in one cell and whenever any input to the formula changes, the result of the formula is automatically recalculated.

API support for change streams (page 456)

Message-passing systems such as actors (Message Passing Dataflow page 136)

Logs compared to traditional messaging (page 448)

* When maintaining derived data, the order of state changes is often important. We need stable message ordering.

* Fault-tolerance is essential for derived data, losing just a single message causes the derived dataset out of sync.

Stream processor example: The code that processes purchases would subscribe to a stream of exchange rate updates ahead of time, and record the current rate in a local database whenever it changes. So we can have a stream join instead of RPC. Note: this join is time-dependent.

### Observing Derived State

write path - eager evaluation, precomputing results and save to a cache of common queries.

read path - lazy evaluation only happens when someone asks for it.

No index means less work on the write path, but a lot more work on read path. On the other hand, we can pre-compute the search results for all possible results. Therefore, the boundary between write path and read path can be shifted.

Clients with offline operation (page 170) - mobile apps can work off-line and do not have to wait for synchronous network requests.

**on-device state** - a cache of state on the server

Pushing state changes to clients - like using Websocket, keeping an open TCP connection toa server.

Consumer offsets (page 449) - a consumer of log-based message broker can reconnect after failing or becoming disconnected.

## Aiming for Correctness

### The End-to-End Argument for Databases

Exactly-once execution of an operation - give up or try again. But we need **Idempotent** - to ensure it has the same effect no matter whether it is executed once or multiple times.

Uniquely identifying requests - e.g. generating a unique identifier for a request. A transaction identifier is passed all the way from the end-user client to the database. Also, end-to-end checksums to catch data corruption. End-to-end encryption and authentication to against network attacks.

### Enforcing Constraints

Constraints and uniqueness guarantees (page 330). E.g. a user name or email address must uniquely identify a user.

Make a single node the leader OR we have to deal with the consensus problem.

Uniqueness in log-based messaging - Any writes that may conflict are routed to the same partition and processes sequentially.

Total Order Broadcast (page 348)

Logs Compared to traditional messaging (page 448)

### Timeliness and Integrity

Timeliness means ensuring that users observe the system in an up-to-date state.

Integrity means absence of corruption. Such violations (perpetual inconsistency) can be catastrophic. Thus, fault-tolerance message delivery and duplicate suppression (e.g. idempotent operation) are important for maintaining the integrity of a data system.

Advantages of immutable events (page 461)

Need a trade off between performance/availability and number of apologies.

### Trust, but Verify

Random bit-flips are still very rare but needs attention.

Checking the integrity of data is called **auditing**, not only for financial applications but also for the need to detect and fix problems.

For example, large-scale storage systems such as HDFS and S3 do not fully trust disks - they run background processes that continually read back files, compare them to other replicas, and move files from one disk to another in order to mitigate the risk of silent corruption.

A culture of verification - The disk works correctly most of time doesn't mean it always works correctly. Not many systems have such kind of "trust, but verify" approach of continuously auditing themselves mechanism. We may need more self-auditing or self-validating systems in the future.

Designing for auditability - If a transaction mutates several objects in a database, it is difficult to tell. But event-based systems can provide better auditability. A deterministic and well-defined dataflow also makes it easier to debug and trace the execution of a system - a kind of time-travel debugging capability.

Having continuous end-to-end integrity checks can help finding corruptions.

Merkle tree - trees of hashes that can be used to efficiently prove that a record appears in some dataset.

## Doing the Right Thing

### Predictive Analytics

The output of a prediction system is probabilistic and may well be wrong in personal/individual cases.

### Privacy and Tracking

A world that treats people with humanity and respect.
