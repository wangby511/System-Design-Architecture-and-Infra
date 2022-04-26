# Chapter 8 - The Trouble with Distributed Systems

CREATED 2022/02/05

Consensus - get all of the nodes to agree on something.

## Consistency Guarantees

Convergence - eventually consistency. If you write a value to a database and wait for some time, then all the nodes will return the same value when the read request comes afterwards.

最终一致性（eventual consistency）：如果停止更新数据，等待一段时间（时间长度未知），则最终所有读请求将返回相同的内容。
然而最终一致性是一种非常弱的一致性保证，因为无法知道何时（when）系统会收敛。而在收敛之前，读请求都可能返回任何值。

## Linearizability - 可线性化

**Linearizability** - also called atomic consistency, strong consistency. It makes a system look like it has one copy of the data even through there are maybe multiple replicas behind it in reality.

可线性化(Linearizability)，也被称为原子一致性（atomic consistency）、强一致性（strong consistency），可线性化的基本思想是让一个系统看起来好像只有一个数据副本，而且所有的操作都是原子的，有了这个保证，应用程序就不需要关心系统内部的多个副本。

在一个可线性化的系统中，一旦某个客户端成功提交写请求，所有的客户端读请求一定能看到刚刚写入的值。而不是过期的缓存。但并不要求将操作组合到事务中，因此无法避免写倾斜等问题。

There must be some point in time (between the start and end of the write operation) at which the value of x automatically flips from 0 to 1 for example. If one client's read returns 1 (the new value), then all subsequent reads must also return the new value. We should not see a value flip back and forth several times.

Image a vertical line at the time when the operation(read, write or cas) is executed. E.g. CAS(compare-and-set) operation.

CAS(x, old, new)：表示一次原子的比较-设置操作（compare-and-set，简称CAS），如果此时x的值为old，则原子设置这个值为new；否则保留原有值不变

![img](https://cdn.jsdelivr.net/gh/lichuang/lichuang.github.io/media/imgs/20190406-ddia-chapter09-consistency-and-consensus/9-4.jpg)

上图中的每个操作都有一个竖线，表示可能的执行时间点。可线性化要求，连接这些标记的竖线，必须总是按时间（即从左到右）向前移动，而不能向后移动。因此，一旦新值被写入或读取，所有后续的值读到的都是新值，直到被覆盖。

### Difference between Linearizability and Serializability

Serializability - is an isolation property of transactions. It guarantees that even though transactions may execute in parallel, the end result is the same as if they had executed one at a time, serially, without any concurrency. Its implementation is based on: two-phase locking or actual serial execution. However, the serializable snapshot is not linearizable. It makes read from a consistent snapshot.

Linearizability - is a recency guarantee on reads and writes of a register. It does not prevent problems like write skew.

可串行化：可串行化是事务的隔离属性，其中每个事务可以读写多个对象。用来确保事务执行的结果与串行执行的结果完全相同，即使串行执行的顺序可能与事务实际执行顺序不同。

可线性化：可线性化是读写寄存器（单个对象）的最新值保证，并不要求将操作组合到事务中，因此无法避免写倾斜等问题。

### Usage

Locking and leader election - like single-leader replication. If a nodes owns the lock, it becomes the leader and all other nodes must agree.

Constraints and uniqueness guarantees - E.g. a username is unique. It requires a single up-to-date value that **all nodes agree on**.

Cross-channel timing dependencies - The violation problem arises because there are additional or multiple different communication channels.

### Implementing

1 Single-leader replication - potentially linearizable.

2 Consensus algorithms - contain measures to prevent split brain and stale replicas. E.g. Zookeeper.

3 Multi-leader replication - not linearizable.

3 Leaderless replication - probably not linearizable.

### The Cost

If your application needs linearizability but some replicas are disconnected, then they must wait there until fixed.

If your application does not require linearizability, then the request can be written into any replica independently even if they are disconnected.

### CAP Theorem

When a network fault occurs, we have to choose between either linearizability or total availability.

A better way of rephrasing: **Either choose Consistent or Available when Partitioned**.

CAP定理，表示一致性、可用性、分区容错性，三者之间只能同时满足两个特性。网络在发生故障以后，必须从一致性和可用性之间做出选择。因此，更准确的应该是“网络分区情况下，选择一致还是可用”。

### Trade-off

Every CPU core has its own cache and store buffer to increase performance and meanwhile, it drops linearizability.

Linearizability is slow and not suitable for latency-sensitive systems. Weaker consistency models can be much faster.

## Order Guarantees - 顺序保证

### Ordering and Causality

Causality - Causes come before effects.

* Casual dependency.

* A row must be created before it can be updated.

* No casual line between two concurrent things.

* Read skew means reading data in a state that violates causality.

* SSI detects write skew by tracking the casual dependencies between transactions.

* "Crossing-channel timing dependencies" also violates causality.

If a system obeys the ordering imposed by causality, then it is **causally consistent**, like snapshot isolation.

Linearizability implies causality.

### Sequence Number Ordering

It can come from a logical clock, an algorithm to generate a sequence of numbers to identify operations, instead of a time-of-day clock. So we can create sequence numbers in a total order that is consistent with causality, especially for the single-leader replication mode. However, multi-leader or leaderless database can not guarantee consistent causality.

In a database with single-leader replication, the replication log defines a total order of write operations that is consistent with causality. The leader can simply increment a counter for each operation and then assign a monotonically sequence number to each operation in the replication log. We can guarantee that the state of the followers is always consistent.

在主从复制数据库中，复制日志定义了与因果关系一致的写操作全序关系。主节点可以简单地为每个操作递增某个计数器，从而为复制日志中的每个操作赋值一个单调递增的序列号。从节点按照复制日志的顺序来写，结果一定满足因果一致性。

**Lamport timestamp** - (counter, nodeId) provides a total ordering consistent with causality.

Cons: We know the total order of operations **only after** we have collected all the operations so we can not make a right-now decision. (E.g. create a unique username.)

每个节点都有一个唯一的标识符，且每个节点都有一个计数器来记录各自已处理的请求总数。Lamport时间戳是一个值对（计数器，节点ID）。两个节点可能会有相同的计数器值，但时间戳中还包含节点ID信息，因此可以确保每个时间戳都是唯一的。

In order to implement something like a uniqueness constraint for user names, we have to both maintain a total ordering of operations and also know when that order is finalized.

### Total Order Broadcast - 全序关系广播

**total order broadcast** or atomic broadcast. It is usually described as a protocol for exchanging messages between nodes with two safety properties satisfied:

* Reliable delivery - no messages are lost.

* Totally ordered delivery - Messages are delivered to every node in the same order.

Also do retry logic after the network is repaired from interrupted.

Every replica processes the same writes in the same order with the leader.

1.Order is fixed. 2.Replication log, transaction log or write-ahead log. Appending to the log file.

全序关系广播指节点间交换消息的某种协议，要求满足以下两个基本安全属性：

可靠发送：没有消息丢失，如果消息发送到了一个节点，也必须要发送到其他节点。严格有序：消息总是以相同的顺序发送给每个节点。

一旦消息发出，此时顺序就确定下来了。节点不允许往已经存在的消息序列中插入消息，只能往后追加，因此可以将全序广播看作是一个记录日志的过程：所有节点都在异步记录一个全局顺序一致的日志。

在single-leader架构中，只有一个leader能接受写操作，从而能够保证所有写操作都是有序的，进而保证所有副本都是有序的。这意味着single-leader架构本身就具有全序传递的特性，只要在此基础上解决了可靠传输问题，就能实现全序广播。 常规的single-leader架构需要在启动时，人为指定一个leader节点，一旦这个节点失效，整个系统将会陷入一个不可用的状态，直到人工介入指定一个新的leader。这对系统的可用性无疑会造成严重的影响。

为了实现自动灾备auto-failover，系统本身需要支持leader选举功能：当leader 失效时，从健康的follower中选择一个新的leader，继续对外提供服务。在此类选举场景中，不可避免的会用到布式一致性算法distributed consensus algorithm。

## Distributed Transactions and Consensus - 共识

Consensus - get several nodes to agree on something.

Leader election - Two nodes both believe themselves to be the leader due to a network fault.

Atomic commit - All nodes agree on the outcome of the transaction, either all commit or all abort/roll back.

### Atomic Commit and Two-Phase Commit

Multiple nodes are involved in a transaction - A node must only commit once it is certain that all other nodes in the transaction are also going to commit.

**Two-Phase Commit (2PC)** - an algorithm for achieving atomic transaction commit across multiple nodes. To ensure that all nodes commit or all nodes abort.

Phase 1 - The coordinator (transaction manager) sends out a request request to each of nodes, asking them whether they are able to commit.

Phase 2 - The coordinator sends commit request or abort request. At the same time it must write the decision in its transaction log, called **commit point**.

![img](https://cdn.jsdelivr.net/gh/lichuang/lichuang.github.io/media/imgs/20190406-ddia-chapter09-consistency-and-consensus/9-9.jpg)

Two crucial "points of no return": 1. When a participant votes yes, it promises that it will definitely be able to commit later (although the coordinator may still choose to abort). 2. Once the coordinator decides, the decision is irrevocable.

**Coordinator Failure** - If the coordinator crashes before it sends each participant commit/abort request, that state is called **in doubt or uncertain**. They can do nothing but stuck waiting until the coordinator recovers and reads the transaction log. So the only way is waiting for the coordinator to recover. And the coordinator must write its commit or abort decision to a transaction log on disk before sending to participants. Any transactions that do not have a commit record are aborted.

The commit point of 2PC comes down to a regular single-node atomic commit on the coordinator.

### Distributed Transactions in Practice

**Database-internal distributed transactions** is better than **Heterogeneous distributed transactions**.

Holding locks while in doubt - The database can not release those locks until the transaction commits or aborts.

Recovering from coordinator failure - But some orphaned in-doubt transactions do occur and the coordinator can not decide after it restarts from crashing. Therefore it holds locks and blocks other transactions. At this time manual efforts are required.

Heuristic decision may be used to get out of catastrophic situations but it violates promise of two-phase commit.

### Fault-Tolerant Consensus

* Uniform agreement - No two nodes decide differently

* Integrity - No node decides twice

* Validity - If a node decides value v, then v was proposed by some nodes.

* Termination - Every node that does not crash eventually decides some value.

Even if some nodes fail, the other nodes must still reach a decision. (Termination is a liveness property. 2PC does not meet the requirements for termination.)

Total order broadcast is equivalent to repeated rounds of consensus:

* Agreement property - all nodes decide to deliver the same messages in the same order.

* Integrity property - messages are not duplicated.

* Validity property - messages are not corrupted and not fabricated out of thin air.

* Termination property - messages are not lost.

**Single-leader replication** - We still need to worry about consensus. If the leader node goes down, we need human intervention or automatic leader election which may bring another split brain problem. It does not guarantee that the leader is unique.

**Epoch numbering and quorums** - fault-tolerant consensus algorithm to elect a leader node. There are two rounds of voting, one to choose a leader, and a second time to vote on a leader's proposal. This only requires a quorum number of nodes.

**Limitations of Consensus** - Some data could be lost during async replication. A network failure cuts off some nodes. We can not add or remove nodes from cluster due to a fixed set of nodes which participate in voting. False positive timeout failure in a node. Edge case found in Raft.

所有支持容错的共识算法都有以下的性质：

* 协商一致性（Uniform agreement）：所有的节点都接受相同的决议。

* 诚实性（Integrity）：所有节点都不能反悔，即对一项决议不能有两次不同的结果。

* 合法性（Validity）：如果决定了v值，则v一定是某个节点所提议的。即：不能有一个凭空的决议产生。

* 可终止性（Termination）：节点如果不崩溃则最终一定可以达成协议。

可终止性引入了容错的思想。它强调一个共识算法不能原地空转，永远不做事情。即使某些节点出现了故障，其它节点也必须最终做出决定。因此，可终止性属于一种活性属性（liveness property），而其它三个性质属于安全性方面的属性。

任何共识性算法，都需要至少大部分节点正确运行才能保证终止性，这个”大多数节点“又被称为”quorum“。因此，可终止性的前提是，发生崩溃或者不可用的节点必须小于小半数节点。另外，共识算法也界定系统不存在拜占庭错误。

当主节点失效时，马上进行一轮新的投票来选举出新的主节点。选举会赋予一个单调递增的epoch号，如果出现不同的主节点，那么就看谁的epoch号更大的胜出。

在主节点做出任何决定之前，必须首先检查是否存在比它更高的epoch号。基于前面做分布式系统的一个准则“真理由多数决定“，节点不能依靠自己掌握的信息来决策，而应该从quorum节点中收集投票。节点只有当没有发现更高epoch的主节点存在的情况下，才会对当前的提议进行投票。

因此实际上这里是两轮不同的投票：首先投票决定谁是主节点，然后是对主节点的提议进行投票。投票过程看起来像2PC，区别在于：2PC的协调者不是依靠选举产生；另外共识算法只需要收到quorum节点的应答就可以通过决议，而2PC需要所有参与者都通过才能通过决议。

### Membership and Coordination Services

* Linearizable atomic operations

* Total order of operations - zxid or cversion

* Failure detection - Open an active session by checking active heartbeats. The locks are automatically released when the session is expired.

* Change notifications

Allocating work to nodes:

* single-leader database

* partitioned resources are shifted

Service discovery - find out which IP address to connect in order to reach a particular service.

Membership service - A membership determines which nodes are currently active and live members of a cluster.

ZooKeeper不仅实现了全序广播(因此实现了共识)，还提供了很多非常有用的功能: 线性化的原子操作,操作全序,故障检测,更改通知。

## Reference

[1] <https://www.codedump.info/post/20190406-ddia-chapter09-consistency-and-consensus/>

[2] <https://zhuanlan.zhihu.com/p/447220637>
