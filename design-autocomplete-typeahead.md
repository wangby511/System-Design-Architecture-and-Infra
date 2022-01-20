# System Design - Autocomplete / Type ahead suggestion

## Functional requirements

Recommend K suggestion terms to users as they are entering text in the searching console in real time.

Those terms matching the given prefix are determined by popularity and historical query frequency.

It guides the users and helps them in constructing their search query.

Need to confirm : Whether the real-time data should be reflected in auto complete suggestions.

## Non-Functional requirements

Real-time. Low latency < 200ms.

Highly available.

Scalable.

## Other requirements

English word only. Case-insensitive.

## Estimations

### DAU

10 million users

10 searches per user each day

Each query 4 words * 5 chars/word = 20 characters

10million *10* 20 / 86400 => 24000 QPS

Peak QPS => 50000

## Basic System Design

### Trie Tree

A trie is a tree-like data structure used to store phrases where each node stores a character of the phrase in a sequential manner.

In addition to the character, we also need to store the frequency info with each reference in each ending node(not the same as leaf node).

E.g. "tr" -> [tree, 10], [true, 35], [try, 29].

Time complexity: O(p) + O(c) + O(c * log c). p = length of a prefix. c = number of children of a given child.

However, it is very slow because it has to traverse all children nodes under one node (subtree).

### Trading space for time

By doing the pre-calculation, we can store/cache top queries/suggestions at each trie node so that fetching top k queries can be in O(1) time. But this will require **a lot of extra storage**.

### Frequency for updating Trie Tree

**Need to confirm whether** we should reflect the most recent queries pattern **or** we can update the trie at a fixed/certain interval of time like one day.

**Whether the real-time data is important?** It is relevant to how frequently we should update the trie tree storage.

### How to update the trie tree

In Trie tree:

Update the trie tree bottom-up by changing the frequencies. Each parent node will merge results from all of its children to figure out its top suggestions.

Add a filtering layer to protect from some illegal words.

In servers:

We can make a copy of the new trie on each server. Or we can deploy the new trie tree one by one to the servers. (Stop traffic, update and then serve traffic again.)

## Service

### Data Gathering Service

Gather users' input query terms and aggregate them together in backend.

[**Analytic Logs**] -> [**Aggregators**] -> [**Aggregated Data**] -> [**Workers**] -> [**Trie DB**] & [**Trie Cache**] & [**Daily Trie Snapshot**]

Trie DB: It is the persistent storage. There are two options:

1. Document Store. It stores the serialized data of the snapshot in the database like MongoDB.

2. Key-Value Store. Each node in the trie is mapped to a key-value pair in the hash table. E.g. b -> ["be":15, "bee":20, "beer":10, "best":35]

Build the trie tree bottom-up.

### Query Service

Return top K search terms matching user's current input as prefix.

## Storage

### Storage Calculation

100 million unique terms and we will need 30 bytes to store an average query.

100 million * 30 bytes => 3 GB.

Assuming 2% growth every day.

3GB + (0.02 *3 GB* 365 days) => 25 GB

### Storage Type

Document DB like MongoDB to store serialized data.

Key-value store : Each node in the trie tree is mapped to a key in a hash table.

### Storage Partition

#### Range Based Partitioning

A range could contain too many words that they can’t fit into one partition.

#### Adding a shard map manager

Partition based on the maximum capacity of the server dynamically. We also need to maintain another range index mapping. E.g.

Server 1, A-AABC
Server 2, AABD-BXA
Server 3, BXB-CDA
...

When we do querying, it will aggregate results from one or multiple trie servers and return the top results to the client.

#### Partition based on the hash of the term

Make our term distribution random and minimize hot terms.

In order to do querying we have to ask all the shards and then aggregate the results.

## Cache

We can have separate cache servers in front of the trie servers holding the most frequently searched terms and their suggestions.

## Fault Tolerance

Use primary-secondary configuration.

Use the latest Trie tree snapshot as backup plan.

## Follow up

Non-English languages? Use Unicode characters.

Different countries? Store different tries in CDNs.

Personalization? We can store the personal history of each user separately on the server.

## Reference

[1] System Design Interview – An insider's guide, Second Edition. by Alex Xu.

[2] <https://www.educative.io/courses/grokking-the-system-design-interview>

[3] <https://tianpan.co/notes/179-designing-typeahead-search-or-autocomplete>
