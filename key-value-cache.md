# Key Value Cache

In large services we have a read-to-write ratio of 100:1 to 1000:1, so we usually optimize for read by adding cache.

## Inline Cache

### Read-through

The clients read data from the database via the cache layer. The cache returns when the read hits the cache; otherwise, it fetches data from the database, fills the data from data store into cache and replies to client's query.

[**Application**] <--> [**Cache**] <--> [**DB**]

### Write-through

When data is updated, it is written to both the cache and the database. This mode is easy for operation but is slow in data writing because data has to be written to both the cache and database. The cache returns when it finishes the database write.

### Write-back

Write-back (or Write-behind): When data is updated, it is written only to the cache and the cache returns immediately. The modified data is written to the back-end storage only when data is removed from the cache. This mode has fast data write speed but data will be lost if a power failure occurs before the updated data is written to the storage.

Summary: Cache will synchronously (write through) or asynchronously (write behind) write data to the backing database.

## Cache-Aside Pattern

### Read-aside

Application requests data from cache. If the data is not available, application then gets data from backing store and also writes it to the cache for future requests.

### Write

Write to the database first and then invalidate the cache entry.

## Evict Policy

LRU(Least Recently Used) - check time, and evict the most recently used entries and keep the most recently used ones.

LFU(Least Frequently Used) - check frequency, and evict the most frequently used entries and keep the most frequently used ones.

ARC(Adaptive replacement cache) - it has a better performance than LRU. It is achieved by keeping both the most frequently and frequently used entries, as well as a history for eviction (keeping MRU+MFU+eviction history).

![image](https://media-exp1.licdn.com/dms/image/C5622AQF5TD73OvsLOw/feedshare-shrink_800/0/1646325427703?e=2147483647&v=beta&t=yyRtGO7RbciAsL-o1pUnohWTRaEiN4xnJ2HBRzay-ho)

## Reference

[1] <https://blog.the-pans.com/different-ways-of-caching-in-distributed-system/>

[2] <https://1o24bbs.com/t/topic/17552>

[3] <https://codeahoy.com/2017/08/11/caching-strategies-and-how-to-choose-the-right-one/>

[4] <https://tianpan.co/notes/122-key-value-cache>

[5] <https://tanzu.vmware.com/content/blog/an-introduction-to-look-aside-vs-inline-caching-patterns>
