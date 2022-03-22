# System Design - News Feed System

SET UP 2022/02/17 UPDATE 2022/03/~~09~~20

News feed is the constantly updating list of stories in the middle of your home page. It is a compilation of a complete scrollable version of your friends’ and your life story from photos, videos, locations, status updates and other activities.

Example: Facebook newsfeed, Instagram feed, Twitter timeline etc.

## Functions

A user can publish a post and see his/her friends' posts on the news feed page.

Feeds can contain text, images and videos in its content.

10 million DAU and a user can have at most 5,000 friends.

## Post/Feed Publishing

When a user publishes a post, the corresponding data is written into cache and database(DB). Also it is populated to his/her friends' news feeds.

Feed publishing API: **POST v1/me/feed** with content and auth_token

Steps:

* A user sends a post feed request to web servers through load balancer.

* Web servers redirect traffic to different internal servers.

* [**Post Service**] writes the post into Post cache and Post DB.

* [**Fan-out Service**] adds the post into his/her friends'/followers' newsfeed, especially [**News Feed Cache**] for fast read/retrieval.

* [**Notification Service**] informs his/her friends/followers that there is a new content by sending notification.

For the fan-out service, there are two modes to choose from: **fan out on write and fan out on read (push mode vs pull mode)**.

### Fan Out on Write

pros: The news feed is pre-computed/generated in real-time and it is fast to read.

cons: It the number of followers is big, it could cause system's overload(called hot key problem). Precomputing is wasteful for long-inactive users.

### Fan Out on Read

pros: The news feed is generated on demand during read time. It is suitable for hotkey user/celebrity.

cons: Fetching may be slow because it is not pre-computed.

We adopt a hybrid of both approaches - For majority of users we use push mode. For celebrities or users who have many followers/friends, we let followers pull news content on-demand.

### Steps of Data Interaction

* Fetch friend IDs from the cache or graph DB. Filter some friends with mute setting.

* Send friends list and the new post ID to message queues (loose couple components with message queues).

* Fan-out workers fetch data from the message queue and appends to the news feed table in format <postID, userID> (with only IDs stored).

![img](https://systeminterview.com/imgs/chapters/ch11/11-04.png)

## News Feed Building

Aggregate friends' posts in approximately reverse time order.

Newsfeed retrieval API: **GET v1/me/feed** with auth_token

Steps:

* A user sends request, load balancer redirects traffic to web servers.

* [**Newsfeed Service**] fetches the user's news feed timeline from cache (**News Feed Cache**) by pagination. It retrieve news feed IDs first and then does rendering by reading details from Post cache/Post DB.

### Data Interaction Steps

* News feed service gets a list of post/feed IDs from the news feed cache.

* Fetch from caches/DBs for constructing/rendering the content like user name, profile picture, post content, images etc. Return to client in JSON format for rendering.

![img](https://systeminterview.com/imgs/chapters/ch11/11-07.png)

## Cache

* User's news feed cache (home timeline)

* User's profile timeline cache

* Posts content cache

* Social graph cache - user friends list cache (follower & following)

* Action (liked, replied, ...)

## Feed Ranking

Related to:

1)Special following friends or not

2)Creation time order

3)Viewed or not

A final score can be calculated using these features. Even with ads post inserted in the middle of the news feed.

## Data Partitioning

### Sharding Feed Data

For feed data which is being stored in memory, we can partition it based on **UserID**. We can try storing all the data of a user on one server. Therefore, to get the feed of a user, we would always have to **query only one server**. Also, for any given user, since we don’t expect to store more than 500 feed items.

## Reference

[1] <https://www.youtube.com/watch?v=kAmmSTdIm1E>

[2] <https://systeminterview.com/chapter_diagram.php?ch=11>

[3] <https://www.educative.io/courses/grokking-the-system-design-interview>

[4] System Design Interview Book. Alex Xu. Chapter 11. Design A News Feed System.
