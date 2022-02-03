# Chapter 7 - Transaction

CREATED 2022/01/23

## ACID

### Atomicity

Something that can not be broken down into smaller parts. If aborted, discard all or undo. Just make sure no partial failure. If a transaction is aborted, it doesn't change anything and the application can safely retry it.

### Consistent

The data(invariants) must always be true.

### Isolation

Concurrently executing transactions are isolated from each other. Run serially.

### Durability

Save the data to a safe place where the data can be stored without fear of losing it. Any data will never be forgotten.

## Weak Isolation levels

Dirty Read - another transactions see the uncommitted data, which is in a partially updated state ,causing confusion to customers.

Dirty Write - previous transaction has not committed and the later write overwrites the uncommitted value.

脏读：某个事务只完成部分数据的写入，但事务尚未提交，此时另外一个事务可以看到未提交数据。

脏写：两个事务同时尝试更新相同的对象，如果先前的写入只是尚未提交的事务的一部分，且后面事务的写入会覆盖较早的写入。

### Read committed

It can prevent both dirty reads and dirty writes. But it still does not prevent the race condition like between two counter increments. This situation is not dirty write but it is incorrect.

Implementation: Acquire a lock when attempting to write to a object, and release it after committed or aborted. Only one transaction can hold the lock for any given object.

读提交是最基本的事务级别，提供两个保证：

读数据库时，只能读到被提交成功的数据（不会读到脏数据）。

写数据库时，只会覆盖已被提交成功的数据（不会脏写）。

实现防脏写：当事务想修改某个对象时，必须先获得该对象的锁，并一直持有直到事务结束或者终止。因此，在某个时刻，只有一个事务可以拿到该对象的锁。

实现防脏读：对于每个待更新的对象，数据库都会维护其旧值和当前持有锁事务将要设置的新值两个版本。在事务提交之前返回的是旧值；仅当事务提交之后，才会切换到新的值。

### Snapshot Isolation and Repeatable Read

Non-repeatable read or read skew, is not acceptable in the following scenarios: Backups & Analytic queries and integrity checks.

Implementation: Isolation snapshot. Each transaction reads from a consistent snapshot of the database. Repeatable read isolation level guarantees that each transaction will read from the consistent snapshot of the database. In other words, a row is retrieved twice within the same transaction always has the same values.

MVCC(multi-version concurrency control) Add a created_by field and a deleted_by field containing the ID of the transaction.

不可重复读指一个事务读取到了另一事务已提交的数据，造成 select 前后数据不一致。 比如事务A修改了一些数据并且提交了，此时事务B却读取了，这时事务B就形成了不可重复读。

快照隔离级别：每个事物都从数据库的一致性快照中读取，事务一开始看到的是最近所提交的数据，即使数据随后可能被另一个事务修改，但保证事务都只能看到该特定时间点的旧数据。

考虑到多个正在进行的事务可能会在不同的时间点查看数据库状态，所以数据库保留了对象的多个不同的提交版本，称为MVCC。

### Preventing Lost Updates

Lost Updates - two clients concurrently perform a read-modify-write cycle. One overwrites the other's write without incorporating its changes so the data is lost.

更新丢失的典型场景：应用从数据库读取某些值，根据逻辑进行修改，然后写入新值（read-modify-write）。当有两个事务在同样的数据对象上执行类似操作时，由于隔离性，第二个写操作并不包括第一个事务修改后的值，最终会导致第一个事务修改后写入的值丢失。

1 Atomic write operations - 原子写操作

Some databases provide atomic write operations.

2 Explicit locking - 显示加锁

The application explicitly locks the objects that are going to be updated. So it can perform a read-modify-write cycle. E.g. SELECT FOR UPDATE.

3 Automatically detecting lost updates - 自动检测更新丢失

Execute in parallel and if a lost updating is detected, abort the transaction and force it to retry its read-modify-write cycle.

4 Compare and set - 原子比较和设置

只有在上次读取的数据没有发生变化时才允许更新，如果发生了变化，则回退到“读-修改-写回方式”。

### Write Skew and Phantoms

Write Skew - A transaction reads something, makes a decision and writes the decision to the database. By the time the write is made, the premise of the decision is no longer true.

Phantom reads - A write in one transaction changes the result of a search query in another transaction.

写倾斜：既不是脏写，也没有导致数据丢失。两次事务更新的是不同的对象（alice和bob的值班记录），写冲突并不那么直接。

即如果两个事务读取相同的一组对象，然后更新其中一部分：不同的事务可能更新不同的对象，则可能发生写倾斜；而如果不同的事务更新的是同一个对象，则可能发生脏写或更新丢失（具体取决于事件窗口）。

## Serializability - 串行化

### Actual Serial Execution - 串行执行

Remove the concurrency entirely. Execute one transaction at a time in serial order on a single thread.

Limited to some constraints:

Every transaction must be small and fast.

Active dataset can fit in memory.

The write throughput must be low enough to be handled on a single core CPU. So it doesn't scale well.

Cross-partition transactions are possible with a hard limit to extend.

### Two-Phase Locking - 两阶段锁

1st phase - While the transaction is executing - the locks are required.

2nd phase - At the end of transaction - the locks are released.

Exclusive access is required: Writers block other writers and readers, and vice versa. This is the main difference between it and snapshot isolation(Readers never blocks writers and writers never block readers). - 读写锁

It can prevent lost updates and write skew.

The lock can be either shared mode or exclusive mode.

It could cause quite unstable latencies and deadlock problems.

Predictive lock - Belong to all objects that match some search conditions, rather than a particular object. - 谓词锁

Index-range lock - 间隙锁

### Serializable Snapshot Isolation (SSI) - 可串行化的快照隔离

An optimistic concurrency control technique, compared to pessimistic for serial execution and two-phase locking.

SSI is based on snapshot isolation - All reads within a transaction are made from a consistent snapshot of the database. It also adds an algorithm for detecting serialization conflicts among writes and determining which transactions are aborted.

* Detecting stale MVCC reads

* Detecting writes that affects priori reads

Pros:

It doesn't need to block waiting for locks held by another transactions. The latency is more predictable.

It is not limited to the throughput of a single CPU core.

两阶段加锁是典型的悲观并发控制机制。基于这样的设计原则：如果某些操作可能出错，那么直接放弃。串行执行是极端悲观的选择：事务执行期间，等价于事务对整个数据库持有互斥锁。

相比之下，可串行化的快照隔离则是一种乐观并发控制。在这种情况下，如果可能发生潜在冲突，事务会继续执行而不是中止，寄希望于一切相安无事；而当事务提交时，数据库会检查是否确实发生了冲突，如果是的话中止事务再进行重试。

## Reference & Sources

[1] <https://www.codedump.info/post/20190403-ddia-chapter07-transaction>

[2] <https://stackoverflow.com/questions/48417632/why-write-skew-can-happen-in-repeatable-reads>
