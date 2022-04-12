# System Design - News Feed System

SET UP 2022/02/17 UPDATE 2022/04/11

News feed is the constantly updating list of stories in the middle of your home page. It is a compilation of a complete scrollable version of your friends’ and your life story from photos, videos, locations, status updates and other activities.

Actual examples: Facebook newsfeed, Instagram feed, Twitter timeline etc.

## Functional Requirements

A user can publish a post and see his/her friends', groups' posts on news feed page. (Feed Publishing)

A user can comment, like and subscribe/follow others.

News feeds can contain text, images and videos in its content.

A user can fetch news feed. (Newsfeed Building)

Support both mobile app and web pages.

## Non-Functional Requirements

Fetch newsfeed in real-time, maximum latency would be at most 2 seconds.

## Scale Estimation

DAU - 300 million users and each user can have at most 5,000 friends.

Traffic Estimate - Each active user fetches news feed timeline 10 times a day - about 35k QPS. Assume a modern server can handle 50K concurrent connections at any time.

Storage Estimate - Each post is 1KB and every active user contains 500 posts. So the storage estimation is 500 KB * 300M = 150 TB. One server has 100GB and therefore we need 1,500 machines.

## Database Schemas(Relational Databases)

mainly **User** and **Post**

User - **userID**, name, email, address, birth date, creation date, last login date, ...

UserFollow - **userID1, userID2**

FeedItem - **FeedItemID**, userID, creation date, location, number of likes, contents, ...

(FeedItemMedia - **FeedID, MediaID**)

Media - **MediaID**, FeedItemID, type, path of file in blob storage, ...

e.g. Instagram uses Apache Cassandra (NoSQL), Facebook uses TAO (Cache layer) UDB.

## Post/Feed Publishing

When a user publishes a post, the corresponding data is written into cache and database(DB). Also it is populated to his/her friends' news feeds.

Feed publishing API: **POST v1/me/feed** with content, user_id and auth_token

Steps:

* A user sends a post feed request to web servers through load balancer.

* Web servers redirect traffic to different internal servers.

* [**Post Service**] writes the post into Post cache and Post DB.

* [**Fan-out Service**] adds the post into his/her friends'/followers' newsfeed, especially [**News Feed Cache**] for fast read/retrieval.

* [**Notification Service**] informs his/her friends/followers that there is a new content by sending notification.

For the fan-out service, there are two modes to choose from: **fan out on write and fan out on read (push mode vs pull mode)**.

### Fan Out on Write or "Push" model

We need [**Feed Generation Service**].

pros:

The news feed is **pre-computed**/generated in real-time and can be pushed to friends immediately. Also it is fast to read.

We can let users maintain a **Long Poll** request with the server for receiving the live updates.

cons:

If the number of followers is big, it could cause system's overload(called hot key problem). Precomputing is wasteful for long-inactive users.

### Fan Out on Read or "Pull" model

pros:

The news feed is generated on demand during read time. It is suitable for hotkey user/celebrity.

The long-inactive users are not wasting resources.

cons:

Fetching may be slow because it is not pre-computed.

New data might not be shown to the users until they make the pull request.

It’s hard to find the right pull cadence, as most of the time pull requests will result in an empty response if there is no new update.

### Hybrid Mode

We adopt a hybrid of both approaches - For majority of users we use push mode.

Specifically, we can stop pushing posts from users with a high number of followers (a celebrity user) and only push data for those users who have a few hundred (or thousand) followers.
For celebrity users, we can let the followers pull the updates on-demand. For normal users we can take the push mode.

### Steps of Data Interaction

* Fetch friend IDs from the cache or graph DB.

* Get friends info from the user cache. Filter some friends with mute setting.

* Send friends list and the new post ID to message queues (loose couple components with message queues).

* Fan-out workers fetch data from the message queue and appends to the news feed table in format <postID, userID> (with only IDs stored).Store it into news feed cache.

## News Feed Building

Aggregate friends' posts in approximately reverse time order.

Newsfeed retrieval API: **GET v1/me/feed** with user_id auth_token

Steps:

* A user sends request, load balancer redirects traffic to web servers.

* [**Newsfeed Service**] fetches the user's news feed timeline from cache (**News Feed Cache**) by pagination. It retrieve news feed IDs first and then does rendering by reading details from Post cache/Post DB.

### Data Interaction Steps

* News feed service gets a list of post/feed IDs from the news feed cache.

* Fetch from caches/DBs for constructing/rendering the content like user name, profile picture, post content, images etc.

* Return to client in JSON format for rendering to construct the fully hydrated news feed.

### Offline Algorithm

The news feed cache is aggregated in offline algorithm and when the user is online, he/she is going to fetch them by a certain batch size with pagination.

We can store FeedItemIDs in a data structure similar to LinkedHashMap which can allow us to not only jump to any feed item but also iterate (like fetch next batch) through the map easily.

## Cache

* User's news feed cache (home timeline)

* User's profile timeline cache

* Posts content cache

* Social graph cache - user friends list cache (follower & following)

* Action (liked, replied, ...)

## Feed Ranking

The way to rank posts in a newsfeed is not only by the creation time of the post.

Related to:

1) Special following friends or not

2) Number of likes, comments, shares, time of the update

3) Viewed or not

A final score can be calculated using these features. Even with ads post for revenue inserted in the middle of the news feed.

## Data Partitioning

### Sharding Posts and metadata

Shard by postID.

### Sharding Feed Data

For feed data which is being stored in memory, we can partition it based on **UserID**. We can try storing all the data of a user on one server. Therefore, to get the feed of a user, we would always have to **query only one server**. Also, for any given user, since we don’t expect to store more than 500 FeedItemIDs. Therefore, we will not run into a scenario where feed data for a user doesn’t fit on a single server.

## Bottlenecks and Follow Up

Long inactive users are not going to be provided by newsfeed.

How to determine a user is a celebrity? Set a ~~fixed number threshold~~ **range** number of followers because the detailed number of followers is fluctuating. 10k -> yes, back to 7k -> no.

Notification - a user can make preference.

Ranking system - machine learning model to determine a user's preference by ranking. Insert some ads feeds into newsfeed timeline.

Cache is down? - （Avalanche effect). Reload data from the database. Try to maintain it as **robust** as possible. If the cache is down, that could be a catastrophic failure. The question becomes "How to maintain cache not down?". Add more (sync or async) replicas and availability, even consistency.

One of micro-services is down? - Add replicas to handle single point failure.

We can use graph structure to store users' relationships in NoSQL.

For mobile devices, we can choose not to push data, instead, let users "Pull to Refresh" to get new posts.

## Reference

[1] <https://www.youtube.com/watch?v=kAmmSTdIm1E>

[2] <https://systeminterview.com/chapter_diagram.php?ch=11>

[3] <https://www.educative.io/courses/grokking-the-system-design-interview>

[4] System Design Interview Book. Alex Xu. Chapter 11. Design A News Feed System.
