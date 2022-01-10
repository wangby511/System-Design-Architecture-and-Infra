# Rate Limiter

## Concept

A rate limiter limits the number of client requests allowed to be processed over a specific period of time. It then blocks requests once the cap is reached.

## Benefits

1. Prevent resource starvation caused by Denial of Service(DoS).

2. Reduce cost.

3. Prevent servers being overloaded.

## Non-Functional Requirements

1. Highly available.

2. It should not introduce substantial latencies.

## Clarifications

service side? based on what rule? single or distributed environment? a middleware or within gateway or integrated with service code? HTTP code 429 (Too many requests) or other exception handing?

## Algorithms

### Token Bucket Algorithm

When a request arrives, we check if there are enough tokens in the refilling bucket. If yes, take away one bucket. Otherwise, drop the request.

Pros:

Easy to implement and memory efficient.

Able to burst in a short period of time.

Cons:

Difficult to fine tune two parameters (bucket size and token refill rate).

### Leaking Bucket Algorithm

It is known as a first in first out (FIFO) queue. When a request arrives, we check if the queue is full. If not, add into the queue. Otherwise, drop the request.

Pros:

Easy to implement and memory efficient.

Smooth out bursts of requests and processes them at an approximately average rate. Suitable for a stable outflow rate.

Cons:

The old requests occupy the queue and the recent requests could be dropped.

Also difficult to fine tune two parameters (bucket queue size and outflow rate).

### Fixed Window Counter Algorithm

Pros: Reset quota at the end of a fixed unit time window.

Cons: It could cause spike in traffic at the edges of two consecutive windows.

### Sliding Window Log Algorithm

Keep tracks of requested timestamps and stored them in cache such as Redis Sorted Set.

Pros: Very accurate.

Cons: It consumes a lot of memory.

### Sliding Window Counter Algorithm

Like the fixed window algorithm, we track a counter for each fixed window. Next, we account for a weighted value of the previous window’s request rate based on the current timestamp to smooth out bursts of traffic.

The number of requests in the rolling window = requests in the current window + requests in the previous window * time overlap percentage.

## High level Architecture

NOT USE database due to slowness of disk access.

Use in-memory cache such as Redis because it is fast and supports time-based expiration strategy. INCR and EXPIRE commands.

## Rules

Written in configuration files and saved on disk.

Workers frequently pull rules from the disk and store them in the cache for rate limiter to use.

## Distributed Environment

Use a centralized data store such as Redis or Cassandra, to store the counts for each window and consumer. But we have to overcome two issues: race condition and synchronization issue.

Do not use lock because it is going to cause major performance bottleneck and also does not scale well.

A better approach is to use a “set-then-get” mindset, relying on atomic operators that implement locks in a very performant fashion.

## Rate limit by IP or by user?

### By IP

Cons: Multiple users share a single public IP. Also tracking IPv6 addresses can make servers run out of memory.

### By User

Cons: The weakness would be that a hacker can perform a denial of service attack against a user by entering wrong credentials up to the limit; after that the actual user will not be able to log-in.

A right approach could be to do both per-IP and per-user rate limiting.

## Reference

[1] <https://aaronice.gitbook.io/system-design/system-design-problems/designing-an-api-rate-limiter>

[2] [Grokking the System Design Interview: Designing an API Rate Limiter](https://www.educative.io/courses/grokking-the-system-design-interview)
