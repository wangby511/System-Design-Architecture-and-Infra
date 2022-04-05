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
