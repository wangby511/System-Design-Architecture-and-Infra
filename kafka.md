# Kafka

Created in 2022/01/21 lecture - https://donggeitnote.com/2022/01/23/kafka-introduction

## Concept

An open source project for stream event processing.

## Functions

Async communications between two components

Reduce peak traffic

Decoupling each components

## Components

Producers produce messages and write messages into topics. The consumers read and consume the messages. They are independent of each other.

Kafka's data is consisted of topics. Each topic can have many partitions in order to increase scalability. The order of messages in each partition is guaranteed, but the order among partitions are not guaranteed. The producers can have own defined hash function to decide which partition to send the message for one topic. The message key is used to determine the partition of the message but it is not mandatory (The key is not defined and we choose a random partition).

![img](https://donggeitnote.com/wp-content/uploads/2021/11/image-17.png)

Each partition contain many replicas in case we have single point failure and increase high availability. The replicas are saved in brokers. Every broker saves hundreds of thousands of replicas and we should keep replicas of one partition into different brokers as possible. If we have a higher level of rack, we should keep replicas into different racks as well(A rack can have many different brokers).

![img](https://donggeitnote.com/wp-content/uploads/2022/01/image-5.png)

There are two modes of replica:

Leader Replica: Every partition can only have one leader replica in one broker. Every requests from producers are going to be sent to this leader replica.

Follower Replica(s): The main job of followers is to keep data sync with their leader by pulling new messages. When there is an outage or problem in the leader, it can be quickly promoted to leader to ensure the availability.

All the replicas of a partition should not be in the same node/broker.

**(ISR) In-sync replicas** refer to replicas that are "in-sync" with the leader. ```lag. max. messages``` is n, it means that as long as the follower is behind the leader by not more than n messages, it won't be removed from the in-sync replica list. Producers can choose to receive ack until k number of ISRs has received the message, where k is configurable.

The leader and followers are separated in different brokers. The producers must send the messages to leaders but the consumers can read messages from leaders or followers.

![img](https://donggeitnote.com/wp-content/uploads/2021/11/image-19.png)

Many consumers can be defined as a consumer group, They can subscribe to the messages from one topic. But each partition can be consumed by only one consumer in the same consumer group. When one of the consumers in the consumer group is down, another consumer in the same group can take over its original consumer partition.

The consumers from different consumer groups can consume the same partition for one topic.

![img](https://donggeitnote.com/wp-content/uploads/2021/11/image-45.png)

![img](https://donggeitnote.com/wp-content/uploads/2021/11/image-48.png)

## Flows

### Producer APIs

The producer can only push/write messages to partition leader. Each messaged is appended into its specific partition.

The consumer read/pull messages from partition leader in the brokers.

## Partition

### How to sync data in follower from leader?

For replication, each partition has several replicas across different brokers. The follower replicas keep pulling new messages from leader by a ReplicaFetcherThread fetch request.

### Acknowledge Setting

The ack-value is a producer configuration parameter and it has the following three modes:

**ACK=0** - No ACK returned. The producer keeps sending messages to the leader without waiting for any ack, and it never retries.

**ACK=1** - Return ACK once the leader persists the message. The latency is improved but not waiting for data synchronization. The data can still be lost if the followers fail to sync the data before the leader crashes, after the ack.

**ACK=all** - Return ACK when all of the in-sync replicas (all ISRs) receive the record. No need to worry about the data loss but the latency could be high.

The number of ISRs specifies how many replicas the producer must receive before a message is considered to be successfully committed. That number k (k number of ISRs) is configurable.

## Data Delivery semantics

**At-most once** - A message will be delivered not more than once. Messages may be lost. No waiting for an acknowledgement(ACK = 0). Suitable for cases like monitoring metrics where a small amount of data loss is acceptable.

**At-least once** - It is acceptable to deliver a message more than once, but no messages should be lost. ACK = 1 or all. If a consumer process the message but fails to commit the offset to the broker, the message will be re-consumed when the consumer restarts, resulting in duplicating.

**Exactly-once** - Financial-related use cases. Duplication is not acceptable and the downstream service does not support idempotency.

## Commit

Different ways of commit messages:

### Auto commit

Set ```enable.auto.commit=true``` and the consumer commits every 5 seconds. Also we can set ```auto.commit.interval.ms``` by our own. We could re-consume the messages in this situation.

### Commit current offset

The API is ```commitSync()```. We should do a trade-off whether we should put this function before the logic of dealing with those messages (could lead to data loss) or after that (could lead to consuming twice).

### Commit Async

We don't have to wait for the response of commit() before doing the next loop. The API is ```commitAsync()```. It also could lead to the duplicate dealing messages when the commit failed behind.

### Commit Any Offset

Need the parameters of both partition number and offset number.

## File & Disk

A log can contain many segments. Each segment can have two files, .index file and .log file.

In order writing in disk.

## Watermark

## Zero Copy

Send memory data bytes to NIC(Network Interface Card) directly from kernel space's Page Cache, without going through JVM.

## Zookeeper & Kafka Controller

**Zookeeper** is commonly used to provide a distributed configuration service, synchronization service and naming registry. It contains metadata storage, state storage and coordination service. Zookeeper is a good choice for storing metadata.

**Metadata** stores the configuration and properties of topics including a number of partitions. It does not change frequently and the data volume is very small. Also it helps with the leader election of the broker cluster.

## Scalability

### Change of number of partitions

When the number of partitions decreases, the decommissioned partition **can not be removed immediately** because data might be currently consumed by consumers for a certain amount of time.

## Follow Up

**Historical data archive** - use HDFS or object storage to store them. We can even replay some historical messages.

**Delayed messages & Scheduled messages** - The producer sends delayed messages to a temporary storage. And when the deliver time is up, it then sends messages to topics. E.g. you want to delay the delivery of messages to a consumer for a specified period of time. For example, finishing the payment within 15 minutes for an order.

**Message filtering** - filter by adding tags to the metadata of a message. With a message tag, a broker can filter messages in that dimension.

**Pull Model** - Advantage of consumers' pulling data from brokers: Consumers can control the consumption speed/rate. We can scale out the consumers. More suitable for batch processing. But keep pulling when there is no data is some wasting resources. Also we should know the pros/cons for push model (like low latency vs. could fall behind & overwhelmed.)

**Retry consumption** - Send failed messages to a dedicated retry topic, so that they can be consumed later.

## Reference

[1] <https://jack-vanlightly.com/blog/2018/9/2/rabbitmq-vs-kafka-part-6-fault-tolerance-and-high-availability-with-kafka>

[2] <https://cloud.tencent.com/developer/article/1698552>

[3] <https://www.cnblogs.com/lixinjie/p/a-post-about-io-clearly.html>

[4] <https://www.cnblogs.com/huxi2b/p/7453543.html>

[5] <https://www.cnblogs.com/wangby511/p/15193797.html>

[6] <https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/>

[7] <https://donggeitnote.com/2021/12/03/kafka-commit/>

[8] <https://juejin.cn/post/6995519558475841550>

[9] <https://donggeitnote.com/category/kafka/>

[10] System Design Interview Book Volume II. Alex Xu. Chapter 4. Distributed Message Queue.
