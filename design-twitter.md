# System Design - Twitter

Social network service

## User stories, functional requirements of the system

Post tweet (including text, image, video) / Delete

Get Timeline / News feed (a. Home b. User) (Aggregating tweets in reverse chronological order)

Follow / Un follow other users

Like / Dislike / Comment

Mark tweets as favorites

## Extended requirements

Retweet / forward

Search for tweets by keyword

HashTag / Rank Trending Topics

## Non-Functional requirements

highly available(Without the guarantee that the latest news will be returned).

Acceptable latency of the system is 200ms for timeline generation.

Consistency (Eventual Consistency)

Fault tolerance

## Estimations

A read-heavy system (compared to write)

### DAU

200 Million DAU

Tweets View (Read): 200M * 100 tweets ~= 20B per day => 230K QPS read

100 Million tweets => 1200 QPS write

Note: The traffic will be distributed unevenly throughout the day.

### Storage

Tweets 140 characters maximum per tweet (280 bytes), metadata 30 bytes : 100M * (280 + 30) bytes = 31 GB / day

Images 100 M x 20% x 200 Bytes/image = 4TB / day

Videos 100 M x 10% x 2 MB/video = 20TB / day

Total multi-media storage every day ~= 24 TB / day.

### Bandwidth

35 GB/s

## Basic APIs

1 tweet(Post Tweet content)

E.g. tweet(api_dev_key, tweet_text, location, media_ids)

media_ids[]: All the media photo, video, etc. need to be uploaded separately.

Returns: the URL to access that tweet.

2 GetTimeline(pagination)

E.g. get_tweets(api_dev_key, max_number_to_return, next_page_token)

3 follow(api_dev_key, userId)

## Basic Flows

### Post Tweet

A user posts a tweet to [**Tweets Server**]. The [**Tweets Server**] will write the data to [**Tweets Cache**] as well as [**Tweets DB**].

[**Tweets Server**] next will do a [**Fan Out On Write**] operation to add this tweet to all his/her follower's timeline by [**Tweets Feed Servers**] as well as to [**Tweets Feed Cache**]. (Pushing Mode with time complexity - Read O(1) Write O(N))

If this user is a hot user/celebrity, [**Tweets Server**] next will do a [**Fan Out On Read**] operation to store this post in this celebrity's timeline own cache.

Also [**Tweets Server**] will call [**Notification Server**] if he/she has special follower(s).

### Read Timeline / Read Tweets feed

User makes request to [**Tweets Feed Servers**] and get all his/her tweets feed for the latest update.

[**Tweets Feed Servers**] will fetch data(urls json) from [**Tweets Feed Cache**] and [**User Cache**], [**Tweets Cache**].

The tweets are mixed from non-celebrity and celebrity together in the runtime of user's request.

Construct the page by fetching images and videos from CDN by giving specific urls. And finally it is shown to the users in mobile terminals.

## Infra Graph

![](https://coursehunters.online/uploads/default/original/1X/e50becc8e0ae08ed68f93c6a186bb59081b40732.png)

## Database Schema

### User DB

userId (Primary Key) | Name | Description | PhotoId | Email | DateOfBirth | CreationDate | ~~Tweets[] | favoriteTweets[]~~

```
User
---
PK -UserID: int
    UserName: varchar(32)
    Email: varchar(32)
    Name: varchar(30)
    DataOfBirth: date
    CreationDate: datetime
    LastLogin: datetime
```

### TweetDB

TweetId (Primary Key) | userId | Text | ImageIds[] | VideoId[] | Location | Timestamp | Hashtag | (originTweetId in Re-tweet)

### Follow DB (if SQL)

(userId1, userId2) (Primary Key)

```
UserFollow
---
PK UserID1: int UserID2: int
```

### Tweet Favorite DB (if SQL)

~~FavoriteId (Primary Key) | TweetId | userId | timestamp~~

(TweetId , userId)(Primary Key) | createdTimestamp

```
Favorite
---
PK TweetID: int UserID: int
    CreationDate: datetime
```

### Storage DB

ImageId (Primary Key) | image_url

VideoId (Primary Key) | video_url

## FanOut Service

### Non-celebrity / Fan Out On Write

Save tweet to DB/Cache

Fetch all the followers that follow user A

Inject this tweet into all the followers' queues/in-memory timelines

Finally, all the followers can see this tweet in their timelines.

Not suitable for inactive users.

### Celebrity / Fan Out On Read

Generated during read time / On-demand mode

Mixed with the tweets from celebrity in the runtime of user's request

For inactive users, this mode works better.

## Data Sharding

### 1 Sharding based on creation date/time

Cons:

Traffic load will not be distributed. All new tweets will be going to one server and the remaining ones will be sitting idle.

### 2 Sharding based on userId

Cons:

* Hot user problem

* Unbalanced data (some users can end up storing a lot of tweets. Not uniform distribution.)

* Unavailability of all of the user's data on the shard - if that shard is down or higher latency if it's serving high load for specific user

To recover from these situation: repartition/redistribute our data or use consistent hashing.

### 3 Sharding based on tweetId

Pros:

* Balanced data distributed and solve the problem of hot users.

Cons:

* Read Timeline needs to query multiple servers.

Store hot tweets in cache in front of database servers.

### 4 Combined timestamp into tweetID. Epoch time + auto-incrementing

31 bits for epoch seconds + 17 bits for auto incrementing sequence number

Pros:

Reduce the latency for reading

Cons:

Read Timeline still needs to query multiple servers

## Cache

* Improve read performance and reduce database pressure

* Least recently used (LRU)

* Try to cache 20% tweets which have 80% traffic of reading (size of cache) in the past 3 days

* Due to limit of number of connections. It should be split into multiple servers.

* Celebrities timeline should be in the cache.

## Replication and Fault Tolerance

We can have multiple secondary database servers for each DB partition. Secondary servers will be used for read traffic only. All writes will first go to the primary server and then will be propagated to secondary servers.

Whenever the primary server goes down, we can failover to a secondary server.

## Monitor/Metrics

Number of tweets per second/hour/day...

Latency of refreshing timeline (P90 or P99)

Server Internal error rate (500 error code)

## Reference

[1] <https://github.com/donnemartin/system-design-primer/tree/master/solutions/system_design/twitter>

[2] <https://medium.com/@narengowda/system-design-for-twitter-e737284afc95>

[3] <https://www.educative.io/courses/grokking-the-system-design-interview/xV9mMjj74gE>

[4] <https://www.bilibili.com/video/BV1Sf4y1e7wc?spm_id_from=333.999.0.0>

[5] <https://www.1point3acres.com/bbs/thread-498444-1-1.html>

[6] <https://learnsystemdesign.blogspot.com/p/design-twitter-search.html>

[7] <https://aaronice.gitbook.io/system-design/system-design-problems/design-twitter>