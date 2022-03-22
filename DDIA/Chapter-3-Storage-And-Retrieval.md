# Chapter 3 - Storage and Retrieval

CREATED 2022/03/19

select a storage engine - mainly two families of storage engines: log-structured storage engines and page-oriented storage engines.

## Data Structures That Power Your Database

log - an append-only sequence of records

index - additional structure that is derived from the primary data which affects the performance of queries, not affecting the contents of the database. But that is a trade-off: we need to have well-chosen indexes.

### Hash Indexes

avoid running our of disk space - breaking the log into segments of a certain size and closing a segment file when it reaches a certain size.

compaction & merge - perform **compaction** on those segment files by only keeping the most recent update for each key. We can also merge those several segments together at the same time of performing the compaction in the background thread.

* File format - CSV is not the best format for a log.

* Deleting records - append a special deletion record to the data file called **tombstone**.

* Crash Recovery - Bitcask speeds up recovery by storing a snapshot of each segment's hash map on disk, which can be loaded into memory more quickly.

* Partially written records - include checksums to detect corrupted parts of the log.

* Concurrency control - to have only one writer thread.

pros:

**sequential write operations** - much faster than random writes.

cons:

The hash table must fit in memory. Range queries are not efficient.

### SSTables and LSM-Trees

**Sorted String Table, or SSTable** - the sequence of key-value pairs is sorted by key.

1 Merging segments and producing a new merged segment file also sorted by key.

2 Still need an in-memory index to record the offsets for some of the keys.

3 Each entry of the sparse in-memory index points at the start of a compressed block.

Constructing and maintaining SSTables:

* **memtable** - in-memory balanced tree data structure.

* When the size of memtable reaches a threshold, write it out to disk as **an SSTable file**.

* When a read request comes, find from the memtable first and then the most recent on-disk segment, the second most recent ...

* Running a merging and compaction process in the background.

**Term Dictionary** - Given a word in a search query, find all the documents (web pages, product descriptions, etc) that mention the word. The key is a word(a term) and the value is the list of IDs of all the documents(the postings list).

**Bloom Filter** - is a memory-efficient data structure for approximating the contents of a set. It can tell if a key does not appear in the database, and thus saves many unnecessary disk reads for non-existent keys.

**size-tiered** - newer and smaller SSTables are successively merged into older and larger SSTables.

**leveled compaction** - the key range is split up into smaller SSTables and old data is moved into separate "levels".

The idea of LSM-Trees - keeps a cascade of SSTables merged in the background. We can efficiently perform range queries. And we can support remarkably high write throughput because of **sequential write order** to disk.

### B-Trees

B-Trees break the database down into fixed-size blocks or pages, traditionally 4KB in size on disk.

The number of references to child pages in one page of the B-tree is called **the branching factor**.

A B-tree with n keys always has the length of O(logn).

**Write-ahead log (WAL)** - a redo log of an append-only file to which every B tree modification must be written before it can be applied to the pages of the tree itself, to prepare for crash recovery.

optimization - leaf nodes can have references to its sibling pages to the left or right.

### Comparing B-Trees and LSM-Trees

LSM-Trees are typically faster for writes, whereas B-trees are thought to be faster for reads.

A B-tree index must write every piece of data at least twice - one to the write-ahead log and one to the tree page itself.

Advantages of LSM-trees:

* They are able to sustain higher write throughput than B-trees.

* It can be compressed better.

Downsides of LSM-trees:

* The compaction process can interfere with the performance of ongoing reads and writes.

* The compaction can not keep up with the rate of incoming writes and leaves a large number of unmerged segments.

## Other Indexing Structures

We can have **secondary indexes** on the same table.

**concatenated index** - the most common type of multi-column index.

**multi-dimensional index** - a more general way of querying several columns at once. E.g. geographic locations or color dimensions(RGB).

**full-text search and fuzzy indexes** - nearby word and typos with given edit distance.

memory database - The disk is used only as an append-only log for durability and reads are served entirely from memory. It needs to load its state from disk or replica when it is restarted. LRU(least recently used) approach may be used to keep enough memory.

Memcached, caching use only (in-memory key-value stores). Redis offers a database-like interface to various data structures such as priority queues and sets, which is easy to implement in memory.

## Transaction Processing or Analytics?
