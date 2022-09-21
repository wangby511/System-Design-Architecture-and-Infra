# Common Knowledge for System Design Interview

CREATED 2022/03/22

## Interview Signals

Workable solution

Analysis and communication

Tradeoff/Pros and Cons

Knowledge base

## Common Non-Functional Requirements

### Reliability

Reliability is the probability a system will fail in a given period. A distributed system is considered reliable if it keeps delivering its services even when one or several of its software or hardware components fail.

### Scalability

According to Bondi et al., "Scalability is the capability of a system, network, or process to handle a growing amount of work, or its potential to be enlarged to accommodate that growth.". It measures the capability of a system to handle or manage growing and increasing demands or requests.

Solution: **Vertical scaling** - adding resources (memory, CPU, disk, etc.) to a single node and **Horizontal scaling** - adding more nodes to the system.

### Availability

Availability is the percentage of time in a given period that a system is available to perform its task and function under normal conditions.

Availability is the probability/percentage of time of a system to work as required, when required, during a mission. (If a system is reliable, it is available. However, if it is available, it is not necessarily reliable. E.g. if a system was launched without any information security testing.)

Solution: **Redundancy** is one of the widely used mechanisms to achieve higher availability. It refers to storing data into multiple, redundant computers. E.g. Primary server ->(failover) Secondary server. Active data -> mirrored data (Data replication).

### Consistency

Trade off between latency and consistency - After the data is written into primary node and a certain number of secondary/replica nodes(may be not all), the write request is considered successfully. Eventual consistency is implemented like this.

## Common Concepts

**SLA (Service-level Agreement)** - a contract between a service provider and a client. E.g. 99.9% availability.

**AuthN and AuthZ** - Authentication verifies who you are. Authorization validates what operations you could perform based on who you are.

**Write Ahead Log (WAL)** (预写式日志) - write-ahead logging (WAL) is a family of techniques for providing **atomicity and durability** (two of the ACID properties) in database systems. The changes are first recorded in the log, which must be written to stable storage, before the changes are written to the database. It can be used for preventing node failure (checkpoint restarting).

**Rocks DB** - The basic constructs of RocksDB are memtable, an in-memory data-structure to ensure very low latency on reads. sorted static table files (SST files): where data is finally persisted on disk.

**MD5** - MD5 hashes are 128 bits in length and generally represented by 32 hex digits.

**Merkle Tree** - Merkle Trees are used in Dynamo to identify any inconsistencies between nodes/replicas. A Merkle tree is a binary tree of hashes, where each internal node is the hash of its two children, and each leaf node is a hash of a portion of the original data. Individual nodes maintain separate Merkle Trees for the key ranges that they store and they can be used to quickly work out if data is consistent and if not, identify which pieces of information need to be synchronized.

Pros: It can do calculation without requiring nodes to download the entire tree or the whole data set.

**Last-write-wins (LWW)** - Based on the wall-clock timestamp, the later writes win and overwrite the data written by the previous client. Big cons: LWW can easily end up losing data.

**Gossip protocol** is a peer-to-peer communication mechanism in which nodes periodically exchange state information about themselves and other nodes they know about. Each node initiates a gossip round every second to exchange state information about itself and other nodes with one other random node.

Seed nodes are fully functional nodes and can be obtained either from a static configuration or a configuration service. This concept is introduced to avoid **logical partitions** (E.g. Nodes A and B joins the ring together but they are not immediately aware of each other).

**Partition-ing** - is a technique to break a big database (DB) into many smaller parts to improve the availability and load balancing of an application. It is more feasible to scale horizontally after doing that.

a. Horizontal Partitioning: In this scheme, we put different rows into different tables.

b. Vertical Partitioning: In this scheme, we divide our data to store tables related to a specific feature in their own server. Columns-based partitioning.

c. Directory-Based Partitioning: Create a lookup service that records the current partitioning scheme. To find out where a particular data entity resides, we query the directory server that holds the mapping between each tuple key to its DB server. However, it is at the cost of increasing the complexity of the system and creating a new single point of failure (i.e. the lookup service/database).

We should avoid those two problems: 1) unbalanced data - The data distribution is not uniform. 2) hot key issue - There is a lot of load on a partition. In such cases, either we have to create more DB partitions or have to re-balance existing partitions.

**Replication** - Replication means sharing information to ensure consistency between redundant resources to improve reliability, fault-tolerance, or accessibility.

**Microservice** - Break down the application into logical components such that these components become services of their own. So each team would work on the services that handle their features. These services will now be communicating with each other via a set of API calls like REST APIs or Remote Procedure Calls.

Benefits: Each team/system does one thing and does it well, without worrying about breaking another set of features. It is easier for developing independently, testing and scaling.

Others need to be considered: Latency(Function calls are faster than API calls through network), Backward Compatibility, Centralized logging like building a Log Aggregation System.

**Optimistic & Pessimistic Locking** - Optimistic locking, also referred to as optimistic concurrency control, allows multiple concurrent users to attempt to update the same resource. When we read a record, take note of the version number and plus one (better than timestamp because the server clock can be inaccurate over time) and check that the version number hasn't changed before we write back.

Pessimistic Locking is when you lock the record for your exclusive use until you have finished with it. It has much better integrity than optimistic locking but we need to avoid deadlocks problem.

**Modes of communication** - Synchronous and Asynchronous.

Synchronous: When a service waits for a downstream system to respond before responding back to the client with a success or failure response.

Asynchronous: This is a more of a fire and forget approach. A service will fire a call to the downstream system and won’t track it further.

Usually we can follow a hybrid approach - use a synchronous approach for the mandatory steps and an asynchronous approach for the rest. E.g. put Inventory and Payment Services into sync part and put Warehouse and Notification Services into async part.

## Common Component

### Proxy Server

A proxy is a piece of software or hardware that sits between a client and a server to facilitate traffic.

**Forward Proxy** - forward proxy hides the identity of the client.

**Reverse Proxy** - reverse proxy hides the identity of the servers. A reverse proxy server is a type of proxy server that typically sits behind the firewall in a private network and directs client requests to the appropriate backend server.

They can both be used for caching, load balancing, or routing requests to the appropriate servers.

### Load Balancer

**Be Aware of this during Interview! - Only add load balancer when there are multi servers (One server is not enough for traffic)**.

Load balancer helps to spread the traffic of requests across a cluster of servers to improve responsiveness and availability of applications, websites or databases.

Typically we can try to balance the load at each layer of the system. We can add LBs at three places: 1) Between users and web servers. 2) Between web servers and an internal platform layer, like application servers or cache servers. 3) Between internal platform layer and database.

Health Checks - "health check" regularly attempts to connect to backend servers to ensure that servers are listening before a load balancer forwards traffic to that server. Also called "heart beat check".

The load balancer can be **A single point of failure**. To overcome this, we should have **a second, standby load balancer** which can take over the first failure one in the event the main load balancer fails. This is called [mirrored pair](https://www.loadbalancer.org/blog/easiest-way-to-reduce-downtime-avoiding-a-single-point-of-failure/) which gives us an extra layer of **redundancy** and protection against downtime. Or we call `active and passive`.

Difference with API Gateway: An API gateway connects micro-services or different API calls, while load balancers redirect multiple instances of the same microservice components or the same API call as they scale out to different machines.

### Message Queues

Use a message queue - de-coupling the services and act as a buffer in case throttling.

A common solution is to use/adopt a message queue (Kafka) to decouple producers/publishers and consumers/subscribers. This makes the whole process asynchronous and producers/consumers can be scaled independently. Message Queues are highly fault-tolerant and persist messages for some time. It has some publishers adding messages to it, and some subscribers listening to it and picking up the events meant for them at their own pace.

The overall architecture could be microservice-based with heavy usage of the publisher-subscriber pattern, involving a queuing technology like **Kafka, RabbitMQ, ActiveMQ**. Each microservice can talk with another one by using this model of publishing a message and subscribing to channels or topics. This mechanism **de-couples** services from each other in the best possible way.

Benefits: Avoid data loss, scaling independently, reduce complexity, decouple services.

## Normal Questions

### Scaling

* database - vertical vs horizontal

* server machines - The stateless servers can be horizontally scaled.

Leader-follower replica

### Database Sharding - hotkey issue and handling mechanisms

Original purpose - the data becomes very large and it can not be stored on a single machine. So we need to split it into distributed disks for storing and querying.

A shard/partition or service that receives much more data than the others is called **a hotspot or a hotkey**.

This problem can be mitigated by 1) Allocate more nodes/resources to process popular item/key. 2) At the beginning we should choose the right/proper partition key.

Steps of allocating more resources: 1) Apply for extra resources. 2) Extra resources allocated. 3) Split items/events. 4) Merged/Reduced results.

## Delivery Protocols

**Long Polling** - Long polling is technique where the server elects to hold a client connection open for as long as possible, delivering a response only after data becomes available or timeout threshold has been reached. Therefore, Long Polling is **Half-Duplex** meaning that a new request-response cycle is required each time the client wants to communicate something to the server. However, it is more resource intensive and can come with a latency overhead. Also reliable message ordering can be an issue.

**WebSocket** - A WebSocket connection is a thin transport layer built on top of a device’s TCP/IP stack. Suitable for bi-directional real-time communication. **Full-duplex** asynchronous messaging is supported so that both the client and the server can stream messages to each other independently. However, WebSockets don’t automatically recover when connections are terminated.

How many WebSocket connection can a server handle? - By default, a single server can handle 65,536 socket connections just because it's the max number of TCP ports available. But the actual limit is often more like 20k.

Use case: If the communication from the client is at a higher throughput, WebSocket may be a better option. If the communication can be driven by both client and server, WebSocket is the way to go. If the communication is always client-driven, WebSocket is not needed. Although here comes the tradeoff between cost and performance. E.g. It is optimized for high-frequency communication like chat App.

**Server-Sent Events(SSE)** - Server-Sent Events are a one-way communication channel where events flow from server to client only. Server-Sent Events allows browser clients to receive a stream of events from a server over an HTTP connection without polling.

## Process vs Thread

* Processes are independent and contain their own state information. Threads run within a process and share the same state and memory space

* Processes have to communicate with each other using inter-process communication mechanisms. Threads communicate directly because they share the same variables.

* Context switching between threads is faster than between processes.

## How to choose database? - SQL or NoSQL

* What does data look like? Is the data relational? Is it a document or a blob?

* Is the workflow read heavy or write heavy or both heavy?

* Is transaction needed?

* Do the queries rely on many online analytical processing (OLAP) functions?

## Reconciliation

Reconciliation means comparing different sets of data in order to ensure data integrity.

## Replication Related

Data might be replicated among different database replicas to increase reliability.

If data is replicated, replication lag could cause inconsistent data between the primary database and the replicas.

Make sure all replicas are always in-sync. We could use consensus algorithms such as Paxos and Raft.

## Error Handing

### Retry-able and Non-retry-able errors

In general, we believe unexpected runtime exceptions due to network and infrastructure issues (5XX HTTP statuses) are retry-able. We expect these errors to be transient, and we expect that a later retry of the same request may eventually be successful.

We categorize validation errors, such as invalid input and states (for example, record not found or operation not supported), as non-retry-able (4XX HTTP statuses) — we expect all subsequent retries of the same request to fail in the same manner.

## Consistent Hashing

Consistent hashing is a special kind of hashing algorithm such that when a hash table is re-sized and consistent hashing is used,only k/n keys need to be remapped on average, where k is the number of keys, and n is the number of slots.

Using the same hash function like SHA-1, we map the keys and servers into a circular ring. To determine which server a key is stored on, we first calculate its hash and determine its position on the ring. Then we go clockwise from the key position on the ring until a server is found. Based on which node’s range it falls in, we will map the data to that node. Also, when a server is removed or added, only a small fraction of keys require re-distribution.

Benefits: 1) We do not need to move around all the data while adding or removing nodes. Only a minimal movement of data is needed. 2) We can have a nearly even distribution of data across all machines.

### Virtual Nodes (Vnode)

To further achieve more uniform/balanced distribution, we introduce virtual nodes. A real server can have multiple virtual nodes. With virtual nodes, each server is responsible for multiple partitions.

The hash range is divided into multiple smaller ranges, and each physical node is assigned multiple of these smaller ranges. Each of these sub-ranges is called a **Vnode**.

Practically, Vnodes are randomly distributed across the cluster and are generally non-contiguous so that no two neighboring Vnodes are assigned to the same physical node. Furthermore, nodes do carry replicas of other nodes for fault-tolerance.

Advantages of Vnode: 1) spread the load more evenly. 2) make it easier to maintain a cluster containing heterogeneous machines. 3) probability of hot-spots is much less (than that we use big range per node).

Cons: 1) Minimized keys are re-distributed when servers are added or removed. 2) It is easy to scale horizontally because data are distributed more evenly.

## Content Delivery Network (CDN)

A CDN is a system of distributed servers that deliver web content to a user based on the user’s geographic locations, the origin of the web page, and a content delivery server.

We can put more popular items like videos into CDN and leave those less popular items just into various data centers.

## Lambda Architecture

Lambda architecture is a way of processing massive quantities of data that provides access to batch-processing and stream-processing methods with a hybrid approach [3].

## Other Knowledge

### Guice VS Dagger

The key difference is that Guice is dependency injection at runtime, while Dagger is at compile time.

**Guice** - Guice uses reflection at runtime. When we call the method like `Guice.createInjector(new AppModule())`, Guice reflects back onto our AppModule, creates the dependency graph, and uses its annotations to determine how to construct all of the dependencies needed to give our application the classes which we inject into it.

**Dagger** - Dagger is generating extra code during compile-time. Specifically, it runs an annotation processor which looks for @Inject, @Provides, @Module, and @Component annotations to construct the dependency graph. The annotation processor then generates Java code which will be used to create the necessary objects. Also, Dagger is often used in Android development.

![img](https://databricks.com/wp-content/uploads/2018/12/hadoop-architecture.jpg)

## Reference

[1] <https://medium.com/geekculture/cracking-the-system-design-interview-theory-basics-c57f5326181b>

[2] <https://medium.com/system-design-blog/long-polling-vs-websockets-vs-server-sent-events-c43ba96df7c1>

[3] <https://blog.csdn.net/zhxdick/article/details/108780884>

[4] <https://medium.com/airbnb-engineering/avoiding-double-payments-in-a-distributed-payments-system-2981f6b070bb>

[5] <https://ably.com/blog/websockets-vs-long-polling>

[6] <https://dev.to/kevburnsjr/websockets-vs-long-polling-3a0o>
