# AWS Related Domain Knowledge

CREATED 2022/08/13

UPDATED 2023/03/11

## [1] SNS And SQS

### Key Differences

Entity Type

SQS : Queue (similar to JMS, MSMQ).

SNS : Topic-Subscriber (Pub/Sub system).

### Message consumption

SQS : Pull Mechanism — Consumers poll messages from SQS.

SNS : Push Mechanism — SNS pushes messages to consumers.

### Persistence

SQS : Messages are persisted for some duration is no consumer available.

SNS : No persistence. Whichever consumer is present at the time of message arrival, get the message and the message is deleted. If no consumers available then the message is lost.

### Use Cases

Choose SNS if:

* You would like to be able to publish and consume batches of messages.

* You would like to allow same message to be processed in multiple ways.

* Multiple subscribers are needed. (When multiple consumers need notifying, and a push message is required. When the message needs publishing to a variety of consumers.)

Choose SQS if:

* You need a simple queue with no particular additional requirements.

* Decoupling two applications and allowing parallel asynchronous processing.

* Only one subscriber is needed. (When a message only needs to be processed by a known consumer.)

We can design fan-out pattern by using both SNS and SQS. In this pattern, a message published to an SNS topic is distributed to multiple SQS queues in parallel. SQS is mainly used to decouple applications. SNS distributes several copies of message to several subscribers.

### Details About SQS

AWS Simple Queue Service (SQS) offers a robust, highly scalable serverless hosted queue for storing messages and delivering data between application components.

There are two types of queues - Standard(Best effort ordering, At least once delivery) and FIFO(First in First Out ordering, Exactly once processing).

In general, three things can happen to a "message" in the queue:

* Received from a producer and ready to be sent to a consumer: these are "Messages Available".
* Sent to a consumer and waiting for a response(*): these are "Messages in Flight".
* Processed by a consumer (successfully) and deleted from the queue (successfully): these are deleted messages.

The messages not processed by the consumer(s) will end up in the Dead Letter Queue (DLQ) after some retries.

"Retention Period" of a message defines the time at which messages can live in the queue (if not deleted by a consumer or moved to DLQ). It should be between 1 minute and 14 days.

"Visibility Timeout" is the maximum time a message may not be visible to the consumers. It is the period in which the message cannot be processed by any other consumer.

"Delivery Delay" is another parameter you can set to delay the processing of messages appearing in your queue. 0 means no delay (default) and 15 mins is the max value we can configure.

## [2] Kinesis

AWS Kinesis Streams offers large-scale data input and real-time streaming data processing. It is a **highly scalable** service to collect and process large streams of data records in real time. It is the best option for sending a **high volume** of messages, like website or application click stream, transactions log, tracking events – everything requiring a high throughput. And it is also the best option for **stream data analytics**.

A Kinesis data stream is a set of shards. Each shard has a sequence of data records. In each individual shard, the ordering of data records is maintained by sequence number assigned to each record. However, ordering is not maintained across shards. So it is required to have a well-thought-out partition key to avoid hot & cold shards.

## [3] Internet Gateway and NAT Gateway

### Internet Gateway (IGW)

Internet Gateway allows instances with public IPs to access the internet. Only one per VPC.

Internet Gateway enables resources (like EC2 instances) in public subnets to connect to the internet. Similarly, resources on the internet can initiate a connection to resources in your subnet using the public. So it is **two-way direction**.

### NAT Gateway (NGW)

NAT Gateway allows instances with no public IPs to access the internet, but only **one-way direction**. In order to maintain an Availability Zone-independent architecture, it is better to create a NAT gateway in each Availability Zone.

* Default - public NAT Gateway - Instances in **private subnets** can connect to the internet through a public NAT gateway, but cannot receive unsolicited inbound connections from the internet. We must associate an elastic IP address with the NAT gateway when creating a public NAT gateway in a public subnet.

* Private NAT Gateway – Instances in private subnets can connect to other VPCs or your on-premises network through a private NAT gateway.

## [4] AWS Lambda Sandbox

### Components

**Load Balancer** — Distributes invocation requests to multiple frontend invokers in different availability zones. When AZ failures are detected, requests are routed to alternate AZ’s automatically.

**Frontend Worker** — This receives function invocation requests, validates them and dispatches the requests to the Worker Manager.

**Worker Manager** — It tracks the usage of resources and execution environments and handles the assignments of requests.

**Worker** — A secure environment for customers to execute their code in. Workers are typically EC2 instances that have multiple functions being executed on them and these functions are owned by multiple users. **Sandbox** is totally isolated from other sandboxes using things such as cgroups, namespaces, iptables, etc. Sandboxes can be **re-used** for another invocation of the same function (what we’d call “warm”) but a sandbox will never be shared between different Lambda functions. These sandboxes have a lifespan of the function execution and are then destroyed.

## [5] AWS ECS

Amazon Elastic Container Service (ECS) is a highly scalable and fast container management／orchestration service. You can use it to run, stop, and manage containers on a cluster. With Amazon ECS, your containers are defined in a task definition.

An Amazon ECS container instance is an Amazon EC2 instance that is running the Amazon ECS container agent and has been registered into an Amazon ECS cluster.

### Fargate

AWS Fargate is a serverless option. With AWS Fargate, you don't need to manage servers, handle capacity planning, or isolate container workloads for security.

Fargate handles the infrastructure management aspects of your workload for you.

## [6] AWS EC2

### Availability Zones

AWS services have been using Availability Zones for years to build highly available, fault tolerant, and scalable applications. Most services have spread multiple independent replicas of their services across multiple Availability Zones.

Each replica is called a zonal replica. The system is designed so that any of the replicas can fail at any time. When a replica fails, it can be temporarily removed from the system until everything works as expected again.

## Reference

[1] <https://medium.com/awesome-cloud/aws-difference-between-sqs-and-sns-61a397bf76c5>

[2] <https://medium.com/awesome-cloud/aws-vpc-difference-between-internet-gateway-and-nat-gateway-c9177e710af6>

[3] <https://matthewleak.medium.com/aws-lambda-under-the-hood-how-lambda-works-43efba14d899>

[4] <https://aws.amazon.com/blogs/compute/aws-lambda-resilience-under-the-hood/>

[5] <https://betterdev.blog/decision-tree-sqs-sns-kinesis-eventbridge/>

[6] <https://medium.com/lego-engineering/resolving-bottlenecks-of-lambda-triggered-by-kinesis-part-1-data-stream-capacity-41979a64cf49>

[7] <https://medium.com/@knoldus/data-streaming-with-aws-kinesis-8183ca2e600d>
