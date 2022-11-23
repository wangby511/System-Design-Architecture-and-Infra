# System Design - Instagram

CREATED 2022/09/01

UPDATED 2022/11/21

## Funtional Requirements

Upload photos

Retrieve photos' feed from followers

Follow & unfollow users / like posts

Post stories / Stories feed

## Non-functional Requirements

Relatively low latency < 0.5 s

Highly available

Highly reliable (data never lost)

## Estimation

Daily active users (DAU) ~ 800 million

They post a photo every 10 days => 800 write request per second

They read feed 10 times per day => 80k read request per second

**Read Heavy Application** - supporting read 80k QPS, write 800 QPS

Total Storage - 1MB *800 million*1/10*5 years*365 = 146 PB

## Storage / Schema

### Post Table

postId(*), userId, imageUrls list, createdTime

### User Table

userId(*), userName, profileImageUrl, joinTime, description

### Feed Table

userId(*), postId,

### Follow Table

userId, followerId

## APIs

### Upload Image

```
POST /v1/images
Request {
    imageUrls: list<Image>,
    title: String
}
Response {
    postId: String
}
```

### Get Feed / Timeline

```
GET: /v1/feeds?limit={limit}&nextToken={nextToken}

Response {
    feeds: Feed[]
}
Feed: {
    postId: String,
    imageUrls: list<Image>,
    title: String
}
```

### Follow user

```
POST: /follows/{userId}
```

## Infra Graph

![](https://blog.acecodeinterview.com/content/images/2020/05/instagram-diagram-final.PNG)

## Components

Post/Upload Service -> save posts into Cache & DB. Upload images into blob storage

Internal Fan Out Service -> generate followers' feeds and save into cache & DB

Feed Service -> fetch a user's own feeds timeline

## Fault Tolerance

Purposes of fault tolerance: 1) 无单点故障 (No single point of failure). 2) Fail gracefully.

For load balancer, we can have secondary ready for back up if the primary one fails.

For Post Service and Feed Service, if one of the servers fails, we can re-route the traffic to other servers.

For replicas, we use primary-replicas replication mechanism.

## Cache

Set cache on user's timeline feed so that it can load the timeline feed in advance and also the user fetches the next page.

## Data Sharding

Like posts, we can do **consisten hashing** based on postId.

## Reference

[1] <https://nikhilgupta1.medium.com/instagram-system-design-f62772649f90>

[2] <https://blog.acecodeinterview.com/instagram/>
