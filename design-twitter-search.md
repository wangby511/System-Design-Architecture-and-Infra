# System Design - Twitter Search

## Functional requirements

Allow a user to search over all the users' tweets by adding filter (returning by timestamp or other ordering).

Low latency and availability.

## Estimations

### DAU

1.5 billion users

800 million daily active users

400 million new tweets every day with average size of 300 bytes

500 million search every day => 5700 QPS

### Storage

Tweets 140 characters maximum per tweet (280 bytes), metadata 30 bytes : 400M * (280 + 30) bytes = 124 GB / day => 1.43 MB/s

## Basic API

search(api_dev_key, search_key_word, sort_pattern, maximum_results_return, next_page_token)

Note: sort_pattern can be: Latest first (0 - default), Best matched (1), Most liked (2), ...

## Basic Flow - Index Server

User posts a tweet to [**Tweets Server**]. The [**Tweets Server**] will write the data to [**Storage (Cache & DB)**] as well as [**Index Server**].

### Index Storage Estimation

Assume that we have around 300K English words and 200K nouns, then we will have 500k total words in our index.

average length = 5 characters. 500K * 5 = 2.5 MB

The whole tweets in the past 2 years: 2 *365* 400M = 292 Billion tweets

292B * 5bytes / tweetID = 1460 GB

So our index would be like a big distributed hash table, where ‘key’ would be the word and ‘value’ will be a list of TweetIDs of all those tweets which contain that word.

Then we assume that there are around 15 words in each tweet that need to be indexed.

The whole storage = (1460 GB * 15) + 2.5MB ~= 21 TB

Assuming a high-end server has 144GB of memory, we would need around 152 such servers to hold the whole index.

### Index Server Functions

The Index servers split and fetch some key words from a tweet's content.

For each key word, it sends a message to the corresponding message queue with the message (keyword, tweetId).

Then the backend servers receive the message and writes batch into cache and DB storage.

Note: An inverted index would allow you to efficiently look up a list of Tweet IDs by a term in their text.

### BottleNeck

Postings list can be extremely long so that compact storage of postings is important.

## Basic Flow - Query Server

When Twitter gets a query, it sends the query to all the servers or data centers and it queries every shard. All the tweets which match with the query in any shards is going to return the result.

The results are returned, sorted, aggregated, and re-ranked. The ranking is done based on the number of retweets, replies, and the popularity of the tweets. Finally all the results are shown to the user.

## Sharding

### 1 Sharding on words

Cons:

Hot words issue brings with heavy traffic.

Some words can end up storing a lot of TweetIDs compared to others, which causes unbalanced data.

Solution:

To recover from these situations we either have to repartition our data or use Consistent Hashing.

### 2 Sharding based on tweetId

Sharding based on the tweet object: While storing, we will pass the TweetID to our hash function to find the server and index all the words of the tweet on that server. While querying for a particular word, we have to query all the servers, and each server will return a set of TweetIDs. The results will be aggregated together to return to the user.

Pros:

Balanced data distributed.

Cons:

Need to query all storage servers (A centralized server aggregates these results to return them to the user).

## Cache

Hot search word and tweets.

Select Least Recently Used (LRU) as cache eviction policy.

## Fault Tolerance

Primary index server and secondary index server.

## Reference

[1] <https://www.educative.io/courses/grokking-the-system-design-interview>

[2] <https://www.1point3acres.com/bbs/thread-684174-1-1.html>

[3] [Twitter Blog - Omnisearch Index Formats](https://blog.twitter.com/engineering/en_us/topics/infrastructure/2016/omnisearch-index-formats)

[4] [Twitter System Design - Part II - System Design Twitter Search - Youtube](https://www.youtube.com/watch?v=NSmFpZk4H2I)
