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

### Synchronous Versus Asynchronous Replication

**semi-synchronous** - It can guarantee that we can have up-to-date copy of the data on at least 2 nodes - the leader and one sync follower.

### Setting Up New Followers

Take a snapshot of the leader's database and copy that to the new follower node.

The follower connects to the leader and requests all the data changes.

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

If we take synchronous-replication mode, a single node failure or network outage would make the entire system unavailable for writing. But if we take asynchronous-replication mode, we may see outdated information if the follower has fallen behind (called **replication lag**).

### Reading Your Own Writes

We should use **read-after-write** consistency, also known as **read-your-writes consistency**.

E.g. by performing certain kind of reads on the leader.

### Monotonic Reads

A user first reads from a fresh replica, then from a stale replica. Time appears to go backward.

Monotonic read means that if one user makes several reads in sequence, they will not see time go backward. We can do that by making sure that each user always reads from the same replica.

### Consistent Prefix Reads

Any writes that are casually related to each other are written to the same pattern.

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

* Operational transformation

### Multi-Leader Replication Topologies

all-to-all topology, star topology, circular topology.

Figure 5-9: With multi-leader replication, writes may arrive in the wrong order at some replicas.

## Leaderless Replication

The client directly sends its writes to several replicas, while in others, a coordinator node handles that.

### Write to the Database When a Node Is Down

The replication system needs to make sure that eventually all the data is copied to every replica.

Two ways of approaches:

* Read repair - writes the newer value back to the replica when a value is read by application

* Anti-entropy process - a background process

Quorums for reading and writing - w, r are the minimum number of votes required for the read or write to be valid. The requests don't have to wait for all n nodes to respond - they can return when w or r nodes have responded.

If w + r > n, at least one node of the r replicas you read from must have seen the most recent successful write. **at least one overlap**

Some nodes are unavailable due to: the node is down, disk error or network interruption.

### Limitations of Quorum Consistency

There is no direct way of measuring the staleness measurements.

### Sloppy Quorums and Hinted Handoff

**sloppy quorum** - writes and reads still require w and r successful responses, but those may include nodes that are not among the designated n "home" nodes for a value. It is only an assurance of durability and it does not guarantee that we can read the latest value.

n can describes the number of replicas within one data center. Cross data center replication between database clusters happens asynchronously.

### Detecting Concurrent Writes

Concurrent writes in a Dynamo-style data store - there is no well-defined ordering.

**last write wins(LWW)** - attach a timestamp to each write, pick the biggest timestamp as the most "recent" and discard any writes with an earlier timestamp. If data loss is not acceptable, it is a poor choice.

A deletion marker is known as **a tombstone**.

## Summary

Single-leader replication, Multi-leader replication and Leaderless replication.

Replication can be synchronous or asynchronous.

Read-after-write consistency - Users should always see data that they submitted themselves.

Monotonic reads - After users have seen the data at one point in time, they shouldn't later see the older data.

Consistent prefix reads - Users should see the data in a state that makes casual sense.
