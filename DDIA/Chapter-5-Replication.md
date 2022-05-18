# Chapter 5 - Replication

CREATED 2022/04/03

Scalability - If a single machine can not handle the read load and write load, we should potentially spread the load across multiple machines.

Fault tolerance/high availability - If one machine fails, another one can take over to maintain redundancy.

Latency - have servers at various locations worldwide close to each user.

Replication - keep a copy of the same data on several different nodes, potentially in different locations.

Reasons - keep data geographically close to users, allow the system to continue working even if some fails, scale out the number of machines to increase read throughput.

## Leaders And Followers

One of the replicas is designated the leader and the other are known as followers. The change is via **replication log or change stream**.

Querying/Reading can be from both leader and followers but writes are only directed to leader.

### Synchronous Versus Asynchronous Replication - 同步复制与异步复制

Synchronous - the follower has to wait for the response of its follower before reporting to the user.

Asynchronous - the follower does not wait for a response.

**semi-synchronous** - It can guarantee that we can have up-to-date copy of the data on at least 2 nodes - the leader and one sync follower.

### Setting Up New Followers

Take a snapshot of the leader's database and copy that to the new follower node.

The follower connects to the leader and requests all the data changes. If the leader fails, one of the followers needs to be promoted to be the new leader.

### Handling Node Outages

1 Follower failure - Catch-up recovery

2 Leader failure - Failover

* Determining that the leader has failed.

* Choosing a new leader. (Getting all the nodes to agree on a new leader is a **consensus** problem. The situation that both two nodes believe that they are the leader is called **split brain**.)

* Reconfiguring the system to use the new leader.

### Implementation of Replication Logs

* Write-ahead log(WAL) shipping - closely coupled to the storage engine.

* Logical (row-based) log replication - allow the replication log to be decoupled from the storage engine internals. And it also allows the leader and followers to run different versions of the database software, or even different storage engines.

* Trigger-based replication

## Problems with Replication Log

Leader-based replication requires all writes to go through a single node, but read-only queries can go to any replica.

If we take synchronous-replication mode, a single node failure or network outage would make the entire system unavailable for writing.

But if we take asynchronous-replication mode, we may see outdated information if the follower has fallen behind (called **replication lag**). This effect is known as **eventual consistency**.

### Reading Your Own Writes

Let the user submit some data and then view what they have submitted by refreshing the page. But the user can read stale data from a stale replica. To prevent this anomaly, we should use **read-after-write** consistency, also known as **read-your-writes consistency**.

Methods:

* Performing certain kind of reads on the leader. Always read the user's own profile from the leader and any other users' profiles from a follower.

* Monitor the timestamps/replication lag on followers and prevent queries on any follower that is more than one minute behind the leader.

### Monotonic Reads - 单调读

A user first reads from a fresh replica, then from a stale replica. Time appears to go backward.

Monotonic read means that if one user makes several reads in sequence, they will not see time go backward. We can do that by making sure that each user always reads from the same replica.

由于存在Master和各Follower之间的复制延迟不同，因此当用户从不同Follower读取数据时，可能会出现“时光倒流”的问题。即，如果先访问延迟小的Follower，再访问延迟大的Follower，那么后面读到的反而是旧数据。

保证用户如果先读取到较新的数据，后续不会读取到更旧的数据。通常做法包括：确保同一用户总是从固定的副本读取。

### Consistent Prefix Reads - 前缀一致读

Any writes that are **casually related** to each other are written to **the same partition**.

对于一系列按照某个顺序发生的写请求，读取这些内容时也会按照当时写入的顺序。这是分片数据库出现的一个特殊问题。

解决方案是确保任何具有因果关系顺序的写入都交给一个分区来完成。

### Solutions for Replication Lag

transactions - providing stronger guarantees.

## Multi-Leader Replication

Allow more than one node to accept writes - called **multi-leader** configuration.

### Use Cases for Multi-Leader Configuration

E.g. we have a leader in each data center, and each data center's leader replicates its changes to the leaders in other data centers.

Performance - Every write can be processed in the local data center and is replicated synchronously to other data centers.

Tolerance of outages - each data center can operate independently of the others, and replication catches up when the failed data center comes back online.

Collaborative writing - allow several people to edit a document simultaneously.

### Handling Write Conflicts

We need synchronous conflict detection, which means we should use single-leader replication.

Ways of achieving convergent conflict resolution:

1 Give each write a unique ID. **last write wins (LWW)**.

2 Give each replica a unique ID.

3 Somehow merge the values together.

4 Record the conflict in an explicit data structure that preserves all the information.

Custom conflict resolution logic - **On write** (the handler is running in a background process) or **On read** (let the user decide in the next time).

Automatic Conflict Resolution:

* Conflict-free replicated data types (CRDTs)

* Mergeable persistent data structures

* Operational transformation is the conflict resolution algorithm behind collaborative editing applications

### Multi-Leader Replication Topologies

all-to-all topology, star topology, circular topology. - 全部至全部型拓扑，星型拓扑，环型拓扑。

With multi-leader replication, writes may arrive in the wrong order at some replicas.

## Leaderless Replication

The client directly sends its writes to several replicas, while in others, a coordinator node handles that.

### Write to the Database When a Node Is Down

The replication system needs to make sure that eventually all the data is copied to every replica.

Two ways of approaches:

* Read repair - writes the newer value back to the replica when a value is read by application

* Anti-entropy process - a background process

Quorums for reading and writing - r, w are the minimum number of votes required for the read or write to be valid. The requests don't have to wait for all n nodes to respond - they can return when w or r nodes have responded.

If w + r > n, at least one node of the r replicas you read from must have seen the most recent successful write. **at least one overlap**

n个副本，写入需要w个节点确认，读取需要至少查询r个节点，如果 w+r>n，则读取的节点中可以保证包含最新值。

Some nodes are unavailable due to: the node is down, disk error or network interruption.

### Limitations of Quorum Consistency

If a sloppy quorum is used, no guarantee of reading the latest value (overlapping between r nodes and w nodes).

If two writes occur concurrently, no clear which one is the first. The only safe solution is to merge the concurrent writes.

If a write happens concurrently with a read, it is undetermined whether the read returns the old or new value.

If a write is partially successful (fails on some nodes), the successful nodes/replicas can not do the roll back.

If a node carrying a new value fails, the data is restored from the replica carrying the old value. The number of replicas storing the new value may fall behind w, breaking the quorum condition.

Monitoring staleness - difficult to measure the amount of replication lag in leaderless replication.

### Sloppy Quorums and Hinted Handoff

**Sloppy Quorum** - Writes and reads still require w and r successful responses, but those may include nodes that are not among the designated n "home" nodes for a value. It is only an assurance of durability and it does not guarantee that we can read the latest value.

当出现网络问题时，无法满足w和r的数量。可以将写请求暂时放在不属于n集合的其他临时节点中，令写和读满足w和r。等网络问题解决，再将临时节点中的数据回传给原始节点。sloppy quorum是为了提高写入的可用性，但也意味着即使满足w+r>n，也不能保证读取到新值，因为新值可能存在于临时节点，还没被回传过来。

**hinted handoff** - Once the network interruption is fixed, any writes that one node temporarily accepted on behalf of another node are sent to the appropriate “home” nodes.

n can describes the number of replicas within one data center. Cross data center replication between database clusters happens asynchronously.

### Detecting Concurrent Writes

Concurrent writes in a Dynamo-style data store - there is no well-defined ordering.

**Last Write Wins(LWW)** - Attach a timestamp to each write, pick the biggest timestamp as the most "recent" and discard any writes with an earlier timestamp. LWW will discard concurrent writes. If data loss is not acceptable, LWW is a poor choice for conflict resolution.

最后写入胜利

A deletion marker is known as **a tombstone**.

## Summary

Single-leader replication, Multi-leader replication and Leaderless replication.

Replication can be synchronous or asynchronous.

Read-after-write consistency - Users should always see data that they submitted themselves.

Monotonic reads - After users have seen the data at one point in time, they shouldn't later see the older data.

Consistent prefix reads - Users should see the data in a state that makes casual sense.

## Reference

[1] <https://zhuanlan.zhihu.com/p/359976256>

[2] <https://www.1point3acres.com/bbs/thread-623896-1-1.html>

[3] <https://zhuanlan.zhihu.com/p/438670474>
