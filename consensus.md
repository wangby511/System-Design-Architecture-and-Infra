# Consensus

CREATED 2023-01-13

## Related Concepts

Weak consistency: There is no guarantee that all nodes have the same data at any time. Eventual Consistency is the weakest level of consistency.

Strong consistency: The data in all nodes is the same at any time. E.g. Raft & Paxos.

## Paxos

Paxos is a family of protocols for solving the problem of consensus in distributed networks.

Paxos解决的问题是在一个分布式系统中如何就某个值或决议达成一致。

### Function

The Paxos algorithm provides a mechanism that enables distributed systems to continue working in a predictable way in the event of network partitioning or server failures.

Paxos is an algorithm that enables a distributed set of computers (e.g. a cluster of distributed database nodes) to achieve consensus over an asynchronous network. To achieve agreement, one or more of the computers proposes a value to Paxos. Consensus is achieved when a majority of the computers running Paxos agrees on one of the proposed values.

![img](https://www.scylladb.com/wp-content/uploads/paxos-diagram.png)

### Three Roles

Paxos defines several different roles which must cooperate to achieve consensus; they interact to collectively agree on a proposed value. The three roles are:

**Proposers** - receive requests (values) from clients and try to convince acceptors to accept the value they propose.

**Acceptors** - accept certain proposed values from proposers.

**Learners** - announce the outcome to all participating nodes.

Consensus is achieved when a majority of the computers running Paxos agrees on one of the proposed values. In reality, Paxos usually coexists with the service that requires consensus, with each node taking on all three roles.

## Reference

[1] <https://www.scylladb.com/glossary/paxos-consensus-algorithm/>
