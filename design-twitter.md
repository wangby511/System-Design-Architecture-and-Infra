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

A read-heavy system (The read-to-write ratio of twitter is very high. It could be 1,000.)

### DAU & QPS

200 Million DAU

Tweets View (Read): 200M * 100 tweets ~= 20B per day => 230K QPS read

100 Million tweets => 1200 QPS write

Note: The traffic will be distributed unevenly throughout the day or in holidays. Peak 5000 QPS.

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

## Basic Flow - Post Tweet

A user posts a tweet to [**Tweets Server**]. The [**Tweets Server**] writes the data to [**Tweets Cache & DB**] as well as this user's [**User Profile Timeline Cache**].

[**Tweets Feed Servers**] next will do a [**Fan Out On Write**] operation to add this tweet to all his/her follower's timeline cache entry list. This cache is called [**Home timeline Cache**]. (Pushing Mode with time complexity - Read O(1) Write O(N))

**Fan Out**: From one to many points. Add a user's tweet into all his/her follower's home timeline cache entry.

However, if this user is a hot user or a celebrity (he/she has millions of followers), [**Tweets Server**] instead will choose the [**Fan Out On Read**] mode. In this situation, it just stores this post in this celebrity's home timeline cache and it does **not** write this post to all followers' home timeline.

Also, [**Tweets Server**] will call [**Notification Server**] if he/she has special follower(s).

## Basic Flow - Read Timeline / Read Tweets feed

### 1 User / Profile Timeline

The server fetches the data from [**User Profile Timeline Cache**] or [**Tweets Cache/DB**] to generate a timeline (list of tweets) for a specific person.

### 2 Home Timeline

~~**Simple steps**: get followers, get latest tweets, merge together and sort.~~

**Details**:

User makes request to [**Tweets Feed Servers**] and get all his/her tweets feed for the latest update.

[**Tweets Feed Servers**] will fetch data(urls json) from this user's home timeline cache.

Also the server also checks if this user has followed some celebrities by having a celebrity following table. Then the server also fetches the latest tweet data from each celebrity's profile timeline cache one by one.

Those tweets from non-celebrity and celebrity are mixed together and then returned as the whole list.

The terminal app will construct/load the page by fetching images and videos from CDN by giving specific urls. And finally the whole page will be shown to the users in their terminal apps.

## Infra Graph

![](https://coursehunters.online/uploads/default/original/1X/e50becc8e0ae08ed68f93c6a186bb59081b40732.png)

![](https://media.geeksforgeeks.org/wp-content/cdn-uploads/20200925115436/High-Level-Solution-for-Twitter-System-Design.png)

## Database Schema & Storage

### User DB (User Table)

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

### TweetDB (Tweet Table)

TweetId (Primary Key) | userId | Text | ImageIds[] | VideoId[] | Location | Timestamp | Hashtag | (originTweetId in Re-tweet)

### Follow DB ((Follower Table)

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

Insert this tweet into all the followers' in-memory home timelines

The followers can just check their own home timeline to see this tweet.

### Celebrity / Fan Out On Read

Generated only during read time / On-demand mode

Fetch tweets from all the celebrities he/she has followed

Merge those tweets from celebrities with other tweets in this user's home timeline

For inactive users, this mode works better.

## Data Sharding

### 1 Sharding based on creation date/time

Cons:

* Traffic load will not be distributed. All new tweets will be going to one server and the remaining ones will be sitting idle.

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

* Read Timeline needs to query multiple servers. which can result in higher latencies.

Store hot tweets in cache in front of database servers.

### 4 Combined timestamp into tweetID. Epoch time + auto-incrementing

31 bits for epoch seconds + 17 bits for auto incrementing sequence number

Pros:

* Reduce the latency for reading since we do not have secondary index. It will be quite quick to find the latest Tweets.

Cons:

* We still need to query multiple servers for timeline generation.

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

[3] <https://www.educative.io/courses/grokking-the-system-design-interview>

[4] <https://www.bilibili.com/video/BV1Sf4y1e7wc?spm_id_from=333.999.0.0>

[5] <https://www.1point3acres.com/bbs/thread-498444-1-1.html>

[6] <https://learnsystemdesign.blogspot.com/p/design-twitter-search.html>

[7] <https://aaronice.gitbook.io/system-design/system-design-problems/design-twitter>

[8] <https://www.geeksforgeeks.org/design-twitter-a-system-design-interview-question/>

[9] <https://github.com/donnemartin/system-design-primer/blob/master/solutions/system_design/twitter/README.md>

[10] <https://www.youtube.com/watch?v=wYk0xPP_P_8>
