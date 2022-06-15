# Chapter 10 - Batch Processing

CREATED 2022/02/17

Services(online systems) - response time and availability

Batch processing systems(offline systems) - run jobs periodically with some throughput

Stream processing systems(near real-time systems) - operate on events shortly after they happen

## Batch Processing with Unix Tools

### Simple Log Analysis

Unix commands like awk, sed, grep, sort, uniq, ...

The sort utility in GNU Coreutils automatically handles larger-than-memory datasets by spilling to disk, and automatically parallelizes sorting across multiple CPU cores.

### The Unix Philosophy

The same interface of all programs - file : an ordered sequence of bytes.

stdin - standard input

stdout - standard output

limitation of Unix tools - They can only run on a single machine.

## MapReduce and Distributed File systems

It takes one or more inputs and produces one or more outputs, also reads and write files on a distributed system. Normally it does not modify the inputs.

HDFS - Hadoop Distributed File System

GSS - Google File System

Every general-purpose machine in a data center has some disks attached to it. A central server called **the NameNode** keeps track of which file blocks are stored on which machine. It also has **Replication**, which means simply several copies of the same data on multiple machines.

### MapReduce Job Execution

1. Read a set of input files and break it up into records.

2. (Map) Call mapper function to extract a key and value from each input record. Each record is handled independently. Each of the partitions is written to a sorted file on the mapper's local disk.

3. Sort all the key-value pairs by key.

4. (Reduce) Call reducer function to iterate over the sorted key-value pairs. Combine them based on the same key.

MayReduce can **parallelize** a computation across many machines without writing code to handle the parallelism. It also ensures that all key-value pairs with the same key end up at the same reducer by using a hash function. The number of map tasks is determined by the number of input file blocks and the number of reduce tasks is configured by the job author.

**Shuffle** - The process of partitioning by reducer, sorting and copying data partitions from mappers to reducers. Not shuffling a deck of cards.

**Chained together into workflows** - MapReduce jobs can be chained together into workflows with designated HDFS directory. A batch job's output is only considered valid when the job has completed successfully (MapReduce discards the partial output of a failed job.). Tool support is important for managing such complex data-flows with large collection of batch jobs in some dependency order.

### Reduce-Side Joins and Grouping

**1 Sort-merge joins** - mapper output is sorted by key, and the reducers then merge together the sorted lists of records from both sides of the join. E.g.

User activity mapper        Reducer partition 1
                       ->
User database mapper        Reducer partition 2...

...

All data from a particular user ID is brought together in the same partition. All key-value pairs with the same key will be delivered to the same destination (a call to the reducer).

排序合并join - 在mapper端经过分区、排序，然后reducer将来自join两个输入的已排序记录列表合并，结果是具有相同关键字的所有记录都最终会进入对reducer的同一个调用。

**GROUP BY** - Bring the related data to the same place. Sessionization - e.g. collect all the activity events for a particular user session.

**Handling skew** - Because of **linchpin objects or hot keys**, a reducer can produce more records than others. Solution - Spread the work of handling the hot key over several reducers, then combine the values from all the first-stage reducers into a single value key-pair.

在单个Reducer中收集与某个celebrity相关的所有活动（例如他们发布内容的回复）可能导致严重的数据倾斜，也称为热点hot spot。

### Map-side Joins (Page 408)

Extract the key and value from each input record, assign the key-value pairs to a reducer partition, and sort by key.

**2 Broadcast hash joins** - One of the two join inputs is small that it can be loaded into a hash table. Each mapper for a partition of the large input reads the entire data of the small input. We can load the small join input into an in-memory hash table OR put them into a read-only index on the local disk.

广播哈希join

**3 Partitioned hash joins** - If the two join inputs to the map-side join are partitioned in the same way. It is also called "bucket map joins". The prerequisite is that both of the join's inputs have the same number of partitions, with the records assigned by the same key and hash function.

分区哈希join - 如果两个join的输入以相同的方式分区（使用相同的分区数量，相同的关键字，相同的哈希函数），则哈希表方法可以独立用于每个分区。

**Map-side merge joins** - The datasets are not only partitioned in the same way, but also sorted based on the same key.

The output of a reduce-side join is partitioned and sorted by the join key.

The output of a map-side join is partitioned and sorted in the same way as the large input.

### The Output of Batch Workflows

Building search indexes - Documents in and indexes out.

Key-value stores as batch process output - Build machine learning system like classifiers and recommendation systems. The files can only be written once by a batch job and are then immutable.

Philosophy of batch process outputs - Treat inputs as immutable and avoid side effects.

### Comparing Hadoop to Distributed Databases

PP(Massively parallel processing) databases focus on parallel execution of analytic SQL queries on a cluster of machines.

Diversity of storage

Diversity of processing models

Designing for frequent faults - the freedom to arbitrarily terminate processes to enable better resource utilization in a computing cluster.

## Beyond MapReduce

### Materialization of Intermediate State

Every MapReduce job is independent from every other job by only maintaining its own input and output directories on the distributed system.

The files on the distributed system are simply **intermediate state** - a means of passing data from one job to the next. The output of one job is used as the input of another job.

Downsides:

A MapReduce job can only start when all tasks in the preceding jobs(that generate its inputs) have completed.

Mappers are often redundant.

The files are replicated across several nodes, which are overkill for such temporary data.

Fault tolerance - make operators deterministic.

### Graph and Iterative Processing

E.g. PageRank.

The Pregel-Processing model - One vertex can "send a message" to another vertex along the edges in a graph. In each iteration, a function is called for each vertex, passing the function all the messages that were sent to that vertex.

Fault tolerance - the algorithm is deterministic and the messages are logged.

Parallel execution - the framework partitions the graph by vertex ID and graph algorithms often have a lot of cross-machine communication overhead.

### High-Level APIs and Languages

Use relational-style building blocks to express a computation: joining datasets on the value of some field; grouping tuples by key; filtering by some condition; aggregating tuples by counting or summing.

## Summary

### Partitioning

In MapReduce, mappers are partitioned according to input file blocks. The output of mappers is re-partitioned, sorted and merged into a configurable number of reducer partitions. The purpose is to bring all the related data with the same hash key together.

### Fault Tolerance

If a task in a MapReduce job fails, it can simply be started again on another machine and the output of the failed task is discarded.

## Reference

[1] <https://blog.csdn.net/weixin_43902592/article/details/105701289>
