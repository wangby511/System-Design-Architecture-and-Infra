# Cache Consistency

## Look-aside

For look-aside cache, client will query cache first before querying the data store. If it's a HIT, it will return the value in cache. If it's a MISS, it will return the value from data store. That's it.

## Demand-fill

Demand-fill means in the case of MISS, client will not only uses the value from data store, but also puts that value into cache.

## Read-through

Read-through cache means for reads, client reads directly from cache. And if it's a MISS, cache is responsible of filling the data from data store and reply to client's query. It doesn't say anything about writes. Clients can do demand-fill writes to cache or write-through.

## Write-through

When data is updated, it is written to both the cache and the back-end storage. This mode is easy for operation but is slow in data writing because data has to be written to both the cache and the storage.

## Write-back

Write-back(or Write-behind): When data is updated, it is written only to the cache. The modified data is written to the back-end storage only when data is removed from the cache. This mode has fast data write speed but data will be lost if a power failure occurs before the updated data is written to the storage.

![Img](https://blog.the-pans.com/content/images/2018/08/Screen-Shot-2018-08-10-at-3.21.50-PM.png)

If there are many replicas of cache, it becomes a distributed system problem, which a few potential solutions might exist. The most straightforward solution to keep multiple replicas of cache consistent is to have a log of mutations/events and update cache based on that log. This log serves the purpose of single point of serialization. It can be Kafka or even MySQL binlog. As long as mutations are globally total ordered in a way that's easy to replay these events, eventual cache consistency can be maintained. Notice that the reasoning behind this is the same as synchronization in distributed system.

## Reference

[1] <https://blog.the-pans.com/different-ways-of-caching-in-distributed-system/>
