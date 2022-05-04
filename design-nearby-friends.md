# System Design - Nearby Friends

FIRST CREATED 2022/03/21

## Functional Requirements

Return all the nearby online/active friends within a limit of distance like 5 miles on their mobile apps.

The list should be update every few seconds/dynamic. E.g. refresh interval like 30 seconds.

DAU 100 million. Concurrent users 10% - 10 million. QPS = 10 million / 30 ~= 330k.

An user has an average of 400 friends. Is there a hard limit for the number of friends a user can have?

## Non-Functional Requirements

Low latency.

Reliability.

## High Level Design

Behind Load balancers, there are two kind of servers:

RESTful API servers - handles the request of adding/removing friends, updating profile.

WebSocket servers - a cluster of stateful, bi-directional WebSocket servers that handles the near real-time update of users' locations. Websocket: real-time communication between clients and servers.

Storages:

Redis Location Cache - store the most recent location data for each active user with TTL(time to live) in the records - {key: user_id, value: [latitude, longitude, timestamp]}. Why Redis? fast read and write of location data.

User Database - includes user profiles and user friendships

Location History Database - [user_id, latitude, longitude, timestamp]

Pub/Sub Server:

Redis Pub/Sub server - lightweight message bus channels to direct messages(location updates) from one user to all his/her online friends. For each friend, the server subscribes to the friend's channel in the Redis Pub/Sub server.

A user subscribes to each friend's channel, no matter the friend is online or not - trading higher memory use for a simpler architecture.

## API Design

periodic location update

client receives location updates

websocket initialization

subscribe/unsubscribe to a new friend

## Scale

vertical scaling - CPU usage, load or I/O.

Removing existing WebSocket servers should guarantee that all the connections should be stopped and released first because they are **stateful**.

Distributed Redis Pub/Sub server cluster by using consistent hashing (distribute different channels on different servers). There are hundreds of Redis Pub/Sub servers and there are many service discovery packages like **etcd** and **ZooKeeper**. The cluster is usually **over-provisioned** to make sure it can handle daily peak traffic with some extra room.

callback - whenever a new friend is added, send a message to the WebSocket server to subscribe to the new friend's Pub/Sub channel.

## Nearby random person

Add a pool of Pub/Sub channels by geo-hash ID. Each grid is assigned with a channel.

## Source

[1] System Design Interview Book Volume II. Alex Xu. Chapter 2. Nearby Friends.
