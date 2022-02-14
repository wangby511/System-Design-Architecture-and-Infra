# System Design - Yelp or Proximity Service

## Functional requirements

Given a location, a user can search all the nearby places within a give radius.

A user can add his/her favorite places.

A user can post feedback/review/comment to a place: rating(required) + text/pictures(optional)

## Non-Functional requirements

High Availability - Our search service should be highly available.

Scalable - There will be peak demand on the Holiday season or certain tourist attractions.

Low Latency - Users should be able to access information as fast as possible. A real-time search experience with minimum latency.

## Scale

500M places and 100K queries per second (QPS)

## Database Schema

### Database of Places

LocationID 8 bytes (hash key)

Name 256 bytes

Latitude 8 bytes

Longitude 8 bytes

Category 1 byte

Description 512 bytes

TotalRating 1 byte

### Business Profile DB

Place DB

LocationID (Primary Key) | placeName | Address | Category  | Description | [ReviewInfo(ReviewText, [MediaID1, 2, 3,...])]

Business Media DB

MediaID (Primary Key) | LocationID | MediaURL | UserId

Review DB

ReviewID (Primary Key) | LocationID | UserId | Text | Rating

## APIs Design

### 1. Function1(Core): GET /search

parameters: keyword, radius, category(optional), max_return(optional), sortMode(optional), filter(optional), next_token(optional)

return: A JSON containing information about a list of popular places matching the search query. Each result entry will have the place name, address, category, rating and thumbnail image.

### Function2: POST/add  DELETE/remove (to/from favorite list)

parameters: LocationID

### Function3: POST/add

parameters: LocationID, Rating, Review text(optional), media_urls(optional)

## Basic System Design and Algorithm

This is a read heavy & write less system since objects/places do not change their location usually.

### SQL solution

Store all the data in a relational database like MySQL. Each place will have its longitude and latitude stored separately in two different columns.

Select * FROM Places where Latitude between X-D and X+D and Longitude between Y-D and Y+D

Cons:

Not completely accurate and not efficient.

### Grids

GridID is a four bytes number. Based on a given location and radius, we can find all the neighboring grids and then query these grids to find nearby places.

We have 20 million grids and 500 million places. Since LocationID is 8 bytes, we would need about 4GB memory (ignoring hash table overhead) to store the index in the memory:

4 *20M + 8* 500M ~= 4 GB

### QuadTree

A tree in which each node has 4 children nodes. Starting from the root node which represents the whole world, we keep splitting each child node until there are no nodes left with more than 500 locations.

All information of places are stored in leaf nodes. (A node represents a grid with no more than 500 places).

With double linked pointer to other leaf nodes and parent pointer to parent node, we can find neighboring grids of a given grid.

### Workflow

First we find the node that contains the user’s location.

If that node has enough desired places, we can return them to the user.

If not, we will keep expanding to the neighboring nodes (either through the parent pointers or doubly linked list) until either we find enough required number of places within the maximum radius.

Storage 24 Bytes x 500M = 12 GB (potentially)

## GeoHash

GeoHash uses a string to represent a zone or location by splitting the map into grids. The world is split into 4 x 8 = 32 grids initially with a character representing each grid. Then we do further splitting in each grid by continually using **Z/N-order curve** to maintain locality and appending one more character into the GeoHash string. (Do 4 *8, then 8* 4, then 4 *8, ...)

The more similar the prefix shared by two strings, the closer two places are.

The longer a GeoHash string is, the more precise location it demonstrates. E.g. "wtw3" - Shanghai City center. "wtw3sy" - LuJiaZui.

length: 6 -> 1.2 km x 600m, the error delta distance is 500 meters. length 8 -> the error delta distance is only 7 meters. 8 bytes.

### Use GeoHash

Calculate a user's GeoHash string based on his/her geo location (latitude and longitude).

If the grids are too small, we could search too many number of grids. If the grids are too big, the search result could contain those unnecessary far-away locations. Usually we search for up to nine (9) grids in total OR also we can include some neighbor grids, e.g. GeoHash6 >= "bc1234" and GeoHash6 <= "bc123s".

## Data Partition

### 1. By region or zip-code

Cons:

The issue of hot places - One of the server in the cluster receive too many requests.

The issue of unevenly distributed data - Some regions can end up storing a lot of places compared to others.

### 2. By LocationId

This could make a request querying too many shard servers.

 To find places near a location, we have to query all servers and each server will return a set of nearby places. A centralized server will aggregate these results to return them to the user. May not be efficient.

### 3. By Geo-hash/Google S2

To avoid the issue of hot shard, try to make that cells are continuous in one shard and adjust the number of cells in one shard by the number of places.

Geo-hash is widely adopted in the open source community (e.g. ElasticSearch, MongoDB, and others).

## Replication

Primary QuadTree server serves write traffic and applies to all other replicas (secondaries), which only serve read traffic.

## Cache

Between backend databases and application servers, we can store all data for hot places.

Least Recently Used (LRU) seems to be suitable in this case.

## Load Balancing

Round Robin (distributed equally)

More intelligent LB can also take traffic/load/server status as consideration by periodically querying backend server and then adjust the traffic volumes.

## Ranking

E.g. While searching for the top 100 places within a given radius, we can ask each partition of the QuadTree to return the top 100 places with maximum popularity. Then the aggregator server can determine the top 100 places among all the places returned by different partitions.

## CDN

A CDN is a system of globally distributed servers that deliver web content to a user based on the geographic locations of the user, the origin of the web page and a content delivery server. CDNs replicate content in multiple places. User can get the content from the nearest CDN.

Push CDNs/Pull CDN

Disadvantage: expensive/read stale data

## Reference

[1] <https://medium.com/swlh/design-a-proximity-server-like-nearby-or-yelp-part-1-c8fe2951c534>

[2] <https://codeburst.io/design-a-proximity-server-like-yelp-part-2-d430879203a5>

[3] <https://www.educative.io/courses/grokking-the-system-design-interview>

[4] <https://gis.stackexchange.com/questions/91788/are-there-trade-offs-between-using-a-geohash-vs-using-a-quadkey-as-a-spatial-ind>

[5] <https://www.cnblogs.com/wangby511/p/14293242.html>

[6] <https://www.cnblogs.com/wangby511/p/15641667.html>

[7] <https://www.bilibili.com/video/BV1GR4y1j7B9?t=573.0>

[8] <https://www.systemdesigntutorial.com/hld/yelp>
