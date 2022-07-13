# System Design - Real-time Gaming Rank board

FIRST CREATED 2022/06/29 UPDATED 2022/07/05

## Functional Requirements

Display top 10 players and their scores

Fetch the rank of a specific user

## Non-Functional Requirements

Real-time update on scores

General scalability, availability and reliability

## APIs

POST /v1/scores - add specific scores to one user (This API can only be called by the game servers.)

GET /v1/scores - fetch the top 10 users from the rank board

GET /v1/scores/{userId} - get the specific score of a user

## Components Relationships

[**User**] ->(win a game)-> [**Game Service**] -> (update score)-> [**Rank Score Service**] <-> [**Score DB**]

[**User**] ->(fetch his or her rank / get top 10 users)-> [**Rank Score Service**]

## Store Type Choice

### Relational Database

Not performant: attempting to do a rank operation over millions of rows is not efficient. Also a relational database is not designed to handle the high load of read queries.

However, we can store the rest of details about the users such as their names, profile images in MySQL databases.

User table: userId, display name, profile image url, etc.

Point table: userId, score, timestamp, etc.

### Redis

We can use a specific data type called **sorted sets**. The members of a set must be unique but scores may repeat. Internally, a sorted set is implemented by two data structures: a hash table and a skip list. The hash table maps users to scores and the skip list maps scores to users.

A skip list is a list structure that allows for fast search. It consists of a base sorted linked list and multi-level indexes. We can reduce the time complexity from O(n) to O(logn).

Commands:

```
ZINCRBY leaderboard_feb_2021 1 'mary1934' // a user scores a point

ZREVRANGE leaderboard_feb_2021 0 9 WITHSCORES // fetch the top 10 

ZREVRANK leaderboard_feb_2021 'mary1934' // fetch the user position in the board

ZREVRANGE leaderboard_feb_2021 357 365 // fetch the relative positions of range of users in the board
```

Data:

24 bytes of id + 2 bytes of number (16-bit integer) = 26 bytes

26 bytes * 25 million = 650 MB which is enough for one modern Redis server to hold the whole data.

QPS peak is 2500 updates/second, this is well within the performance of a single Redis server.

### Cache

Also we can store user profiles for top 10 in Redis.

## Scaling

For scaling Redis, we use fixed partition instead of hash partition.

### Fixed Partition

Based on the range of points on the board, for example, 1 ~ 100, 101 ~ 200, ..., 901 ~ 1000. Like bucket sorting algorithm.

### Hash Partition

Redis cluster provides a way to shard data automatically across multiple Redis nodes. It uses CRC16(key) % 16384 instead of consistent hashing function. Each shards has its primary and replicas with a certain number of slots. But the scores are not continuous, therefore it does not provide a straightforward solution for determining the rank of a specific user.

## Other Approach - NoSQL

As the same, it can not provide a straightforward solution for determining the relative rank of a user. But we can do scanning everyday and return a user's relative ranking in percentile.

Cons: we have to query all partitions(shards) to return each top k results and then do merge operation together.

## Fault Tolerance

Redis is configured with a read replica, and when the main instance fails, that replica is promoted.

## Reference

[1] System Design Interview Book Volume II. Alex Xu. Chapter 10. Real-time Gaming Leaderboard.
