# System Design - Yelp or Proximity Service

## Functional requirements

Given a location, a user can search all the nearby places within a give radius.

A user can add his/her favorite places.

A user can post feedback/review/comment to a place: rating(required) + text/pictures(optional)

## Non-Functional requirements

A real-time search experience with minimum latency.

A Read heavy & Write little system.

## Scale

500M places and 100K queries per second (QPS)

## APIs Design

### 1. Function1(Core): GET /search

parameters: keyword, radius, category(optional), max_return(optional), (optional)sortOrder, (optional)filter, (optional)next_token

return: A JSON containing information about a list of popular places matching the search query. Each result entry will have the place name, address, category, rating and thumbnail image.

### Function2: POST/add  DELETE/remove (to/from favorite list)

parameters: placeID

### Function3: POST/add

parameters: placeID, rating, review text(optional), photo_urls(optional)

## Database Schema

### Database of Places

placeID 8 bytes (hash key)

name 256 bytes

latitude 8 bytes

longitude 8 bytes

category 1 byte

description 512 bytes

### Business Profile DB

Place DB

placeID (Primary Key) | placeName | Address | Category  | Description | [ReviewInfo(ReviewText, [mediaId1,2,3,...])]

Business Media DB

mediaID (Primary Key) | placeID | mediaURL | userId

Review DB

reviewID (Primary Key) | placeID | text | userId | rating

## Basic System Design and Algorithm

### SQL

Select * from Places where Latitude between X-D and X+D and Longitude between Y-D and Y+D

Not completely accurate and not efficient.

### Grids

GridID is a four bytes number. Based on a given location and radius, we can find all the neighboring grids and then query these grids to find nearby places.

20 million grids and 500 million places. Since LocationID is 8 bytes, we would need 4GB of memory (ignoring hash table overhead) to store the index in the memory:

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

## Partition

### 1. By region or zip-code

This could cause the issue of hot places. To avoid one of the server in the cluster receive too many requests.

### 2. By locationId

This could make a request querying too many shard servers. May not be efficient.

### 3. By Geo-hash/Google S2

To avoid the issue of hot shard, try to make that cells are continuous in one shard and adjust the number of cells in one shard by the number of places.

Geo-hash is widely adopted in the open source community (e.g. ElasticSearch, MongoDB, and others).

## Cache

Between backend databases and application servers, we can store all data about hot places.

Least Recently Used (LRU) seems suitable in this case.

## Load Balancing

Round Robin (distributed equally)

More intelligent LB can also take traffic/load/server status as consideration by periodically querying backend server and then adjust the traffic volumes.

## Ranking

## CDN

A CDN is a system of globally distributed servers that deliver web content to a user based on the geographic locations of the user, the origin of the web page and a content delivery server. CDNs replicate content in multiple places. User can get the content from the nearest CDN.

Push CDNs/Pull CDN

Disadvantage: expensive/read stale data

## Reference

[1] <https://medium.com/swlh/design-a-proximity-server-like-nearby-or-yelp-part-1-c8fe2951c534>

[2] <https://codeburst.io/design-a-proximity-server-like-yelp-part-2-d430879203a5>

[3] <https://www.educative.io/courses/grokking-the-system-design-interview/B8rpM8E16LQ>

[4] <https://gis.stackexchange.com/questions/91788/are-there-trade-offs-between-using-a-geohash-vs-using-a-quadkey-as-a-spatial-ind>

[5] <https://www.cnblogs.com/wangby511/p/14293242.html>

[6] <https://www.cnblogs.com/wangby511/p/15641667.html>
