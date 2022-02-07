# Chapter 8 - The Trouble with Distributed Systems

CREATED 2022/02/01

## Faults And Partial Failures

The partial failures(Some parts of the system are broken in some unpredictable ways even though others are working fine) are non-deterministic.

### Cloud Computing and Super-computing

Computation can restart from checkpoint.

Build fault-tolerance system. Build a reliable system from unreliable components.

## Unreliable Network

Timeout: after some time we give up waiting and assume that the response is not going to arrive.

### Network Faults in Practice

Chaos Monkey - deliberately trigger (network in this case) problems and test the system's response.

### Detecting Faults

Automatically detect faulty nodes like load balancer and single-leader replication.

### Timeout and Unbounded Delays

Every packet is either delivered within time d, or it is lost.

A non-failed node always handles a request within some time r.

A reasonable timeout is 2d + r.

Async networks have unbounded delays.

### Network congestion and queueing

* Network congestion in a network switch

* CPU cores are busy

* virtualized environments

* TCP flow control (congestion avoidance or back-pressure)

Make a trade-off between failure detection delay and risk of premature(过早的) timeouts. Some latencies applications use UDP rather than TCP.

### Sync VS Async Network

Ethernet and IP are packet-switched protocols. The Internet shares the network bandwidth dynamically.

## Unreliable Clocks

Each machine has its own notion of time. NTP(Network Time Protocol).

### Monotonic VS Time-Of-Day Clocks

Time-Of-Day clocks are synced with NTP. It return the seconds or milliseconds since the epoch. - 墙上时钟

A monotonic clock is suitable for measuring a duration (an interval). It can only be speeded up or slowed down, but it can not jump forward or backward. - 单调时钟

### Clock Synchronization and Accuracy

Precision Time Protocol (PTP) - 精准时间协议

### Relying on Synchronized Clocks

Last Write Wins (LWW) - A node with a lagging clock is unable to overwrite the values previously written by a node with a fast clock. - 最后写入获胜

Logical clocks - are based on incrementing counters rather than an oscillating quartz. They care only about the relative ordering of events. - 递增计数器解决依赖排序的事件

Google's TrueTime API - Returns two values [earliest, latest], which are the earliest possible time and the latest possible time. The actual current time is somewhere within this interval. - 时钟的置信区间 [不早于，不晚于]

A = [A1, A2], B = [B1, B2]. We want to make sure that A1 < A2 < B1 < B2, which means no overlap. A happens before B.

### Process Pauses

In the distributed system, a node's execution can be paused for a significant length of time and then continues running. E.g. a stop-the-world garbage collection.

Response time guarantees - specify a deadline time that the software must respond.

Limit the impact of garbage collection - like an rolling upgrade or a brief planned outage of a node.

进程或线程暂停，导致实际的超时时间与预想的不一样。场景：垃圾收集器导致线程暂停，虚拟机环境中会暂停虚拟机，操作系统上下文切换...

## Knowledge, Truth and Lies

Distributed system - there is no shared memory, only message passing via an unreliable network.

### The Truth Is Defined by the Majority

Quorum - voting number among nodes. Decision to be made requires a minimum number of votes from several nodes.

The leader and the lock -

* Only one node is allowed to be the leader for a database partition.

* Only one transaction or client is allowed to hold the lock for a specified object.

* User name must be unique identifier.

Problem - If a client holds the lock too long and its lease expires after paused process, its write can corrupt the file.

Fencing token - a number that increases every time a lock is granted. Every time a client sends a write request to the storage service, it must include its current token for validation check.

Zookeeper - the transaction ID `zxid` and the node version `cversion` are both monotonically increasing. - 当使用zookeeper作为锁服务时，可以用事务标识 zxid 或节点版本 cversion 来充当fencing token，这两个都满足单调递增的要求。

Byzantine Faults - A node is inadvertently acting in error like making fake fencing token. We assume that there is no cheating issue in this scope. - 拜占庭故障, 节点存在撒谎的情况。

Solutions for Byzantine Faults:

* Add checksum in the application-level protocol for network packets.

* NTP clients can be configured with multiple server addresses.

## System Model and Reality

System Models

* Synchronous model - assumes bounded network delay, process pauses and clock error.

* Partially synchronous model

* Asynchronous model

Faults Modes

* Crash-stop faults

* Crash-recovery faults

* Byzantine (arbitrary) faults

### Correctness of an algorithm

Safety: Uniqueness, Monotonic sequence

Liveness: Availability

有必要区分两种不同的属性：安全性和活性。在上面的例子中，唯一性和单调递增属于安全性，可用性属于活性。

两种性质有何区别？活性的定义中通常包含暗示“最终”一词（最终一致性就是一种活性）。安全性可以理解为“没有发生意外”，活性类似“预期的事情最终一定会发生”。

Although the unreliability of **networks, clocks and processes** is not an inevitable law of nature, most non-safety-critical systems choose cheap and unreliable over expensive and reliable.

## Reference

[1] <https://blog.csdn.net/JackComeOn/article/details/112718776>

[2] <https://www.codedump.info/post/20190405-ddia-chapter08-the-trouble-with-distributed-system/>
