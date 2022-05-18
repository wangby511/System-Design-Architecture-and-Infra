# TOP-K

CREATED 2022/05/15

## Functional Requirement

topK(k, startTime, endTime)

## Non-Functional Requirement

Scalable

Highly Available

Highly Performant/Low latency

Accurate? as accurate as we can

## Solution

### Single host

hash table. A B C A A D C A B C -> A: 4 B: 2 C: 3 D: 1. Use min-heap to find top k. Time complexity O(nLogK).

### Parallel

Load balancer -> (Data Partition-er) to several processor hosts. Each processor host summarize its top k elements. And merge sorted lists in the final storage host.

Split into chucks for bounded data. After finding top k heavy hitters for each chunk, we just merge them together.

We need the whole 1-minute data sets for a particular time period, e.g. 1 day. But on the other hand, we can not accumulate data in memory for the whole day.

### Count-min sketch

Calculate several hash functions for a value. When retrieving a specific data frequency, we take the minimum of all the counters for the element.

### Infra Graph Flow

User -> API Gateway including log processing (factors: memory, CPU, network and disk IO) -> Distributed Messaging System (like Kafka, AWS Kinesis, etc)

1 fast path - fast processor (count-min sketch for a short period of time. It does not grow over time for its size). Also flushes in memory data into storage.

2 slow path - This will have accurate results a few hours later.) Dump all the data into distributed file system like HDFS or object storage.

-> Data Partition-er -> Distributed Messaging System (one queue for each partition) -> Partition Processor (one for each partition) -> Distributed File System -> Frequency Count Map Reduce -> Top K Map Reduce.

### Map Reduce

Frequency Count Map Reduce: Input -> Split -> Map (intermediate key value pairs) -> Shuffle and Sort(group together. All the values of a single key go to the same reducer) -> Reducer Task -> Output: File system.

Top K Map Reduce: Input -> Local Top K -> Global Top K -> Final Top K.

### Data Retrieve

1:00 Top k list, 1:01 Top k list, 1:02 Top k list, ..., 1:59 Top k list

1:00 hour Top k list, 2:00 hour Top k list, ...

## Follow Up

Lambda architecture - is a way of processing massive quantities of data (i.e. “Big Data”) that provides access to batch-processing and stream-processing methods with a hybrid approach. We send events to batch system and a stream system processing system in parallel, and we stitch together the results from both systems at query time.

Spark internally partitions the data and calculates a top k list for each partition using a heap data structure. And all these lists are merged together in the reduce operation.

## Reference

[1] [System Design Interview - Top K Problem (Heavy Hitters)](https://www.youtube.com/watch?v=kx-XDoPjoHw)
