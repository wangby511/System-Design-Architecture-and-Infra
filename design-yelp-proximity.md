# System Design - Yelp or Proximity Service

FIRST CREATED 2021/01/17

## Functional requirements

Given a location, a user can search all the nearby places within a given radius.

A user can add his/her favorite places.

A user can post feedback/review/comment to a place: rating(required) + text/pictures(optional).

Business owners can add, delete or update a business place and it will be effective in the next day (OR in nearly real time), but no frequent updates.

## Non-Functional requirements

High Availability - Our search service should be highly available.

High Scalability - The system can handle the spike in traffic during peak hours or holidays.

Low Latency - Users should be able to access information as fast as possible. A real-time search experience with minimum latency.

## Scale

500M places and 100K queries per second (QPS)

Assume a 20% growth in the number of places and QPS each year

## Database Schema

### Place Table Schema

LocationID 8 bytes (hash key)

Name 256 bytes

Latitude 8 bytes

Longitude 8 bytes

Category 1 byte

Description 512 bytes

Address 32 bytes

TotalRating 1 byte

1 KB per place x 500Million ~= 500GB

### All Tables Schema

* Places Table

LocationID (Primary Key) | placeName | Address | Category  | Description | [ReviewInfo(ReviewText, [MediaID1, 2, 3,...])]

Business Media Table

MediaID (Primary Key) | LocationID | MediaURL | UserId

Review Table

ReviewID (Primary Key) | LocationID | UserId | Text | Rating

## APIs Design

### 1. Function1(Core): GET /v1/search/nearby

parameters: keyword, radius, category(optional), max_return(optional), sortMode(optional), filter(optional), next_token(optional)

return: A JSON containing information about a list of popular places matching the search query. Each entry contains the place name, address, category, star rating and thumbnail image.

Also for specific location/business/places, we have CRUD APIs for them:

GET /v1/businesses/:id

POST /v1/businesses

PUT /v1/businesses/:id

DELETE /v1/businesses/:id

### Function2: POST /v1/favorite  DELETE /v1/favorite (to/from customers' favorite list)

parameters: LocationID

### Function3: POST /v1/reviews (customer' reviews)

parameters: LocationID, Rating, Review text(optional), media_urls(optional)

## Location-based Service & Business Service

### 1 Location-based Service

Given a radius and location, return a list of nearby restaurants.

Find the nearby businesses for a given radius and location with high QPS. Read heavy and no write. Also it is stateless and easy to scale horizontally.

### 2 Business Service

Business owners can add/delete/update location information.

Customers view restaurant details.

This QPS is not high.

## Basic System Design and Algorithm

This is a read heavy & write less system since objects/places do not change their location usually.

### SQL solution

Store all the data in a relational database like MySQL. Each place will have its longitude and latitude stored separately in two different columns.

Select * FROM Places where Latitude between X-D and X+D and Longitude between Y-D and Y+D

Cons:

Not completely accurate and not efficient.

### Grids

The basic idea is to divide the world into small grids. One grid could have multiple locations and each location belongs to one grid.

GridID is a four bytes number. Based on a given location and radius, we can find all the neighboring grids and then query these grids to find nearby places.

We have 20 million grids and 500 million places. Since LocationID is 8 bytes, we would need about 4GB memory (ignoring hash table overhead) to store the index in the memory:

4 *20M + 8* 500M ~= 4 GB

### QuadTree

A tree which is commonly used to partition a 2-dimensional space by recursively subdividing it into 4 sub-grids. Therefore, each non-leaf node in the tree has 4 children nodes in order (up left, up right, down left, down right). Starting from the root node which represents the whole world, we keep splitting each node until there are no nodes left with > 100 locations. All information of places are stored in leaf nodes. This tree structure in which each node can have four children is called **QuadTree**.

For further use, we can use doubly linked list to connect neighbor's leaf nodes so that we can iterate forward or backward among the neighboring leaf nodes to find out our desired locations. Another approach is that we can keep a pointer to parent's node to know all its other children nodes.

**QuadTree's Workflow**: Find the node that contains the user’s location:

* Start from the root node and search downward the tree to the child node containing the desired location.

* If the leaf node has enough number of desired places, we can filter and then return them to the user.

* If not, we will keep expanding to the neighboring nodes (either through the parent pointers or doubly linked list) until either we find enough required number of places within the maximum radius.

**Calculation:**

For each Place, if we cache only LocationID and Lat/Long: 24 *500M ~= 12 GB

And internal nodes: 1Million grids *1/3* 4 pointers * 8bytes = 10 MB

The total is about 12.01 GB which can be easily fit into a modern-day server.

**Note:**

The QuadTree is an **in-memory data structure** and it is not a database solution. It runs on each Location Based Service.

Although the QuadTree index does not take much memory and can be fit in one server, we should use multiple servers to handle/spread the traffic.

### GeoHash

The basic idea is to reduce the two-dimensional longitude and latitude data into a one-dimensional string of letters and digits. It recursively divide the world into smaller and smaller places by adding additional bit.

GeoHash uses a string to represent a zone or location by splitting the map into grids. The world is split into 4 x 8 = 32 grids initially with a character representing each grid. Then we do further splitting in each grid by continually using **Z/N-order curve** to maintain locality and appending one more character into the GeoHash string. (Do 4 *8, then 8* 4, then 4 *8, ...)

The more similar the prefix shared by two strings, the closer two places are.

The longer a GeoHash string is, the more precise location it demonstrates. E.g. "wtw3" - Shanghai City center. "wtw3sy" - LuJiaZui.

length: 6 -> 1.2 km x 600m, the error delta distance is 500 meters. length 8 -> the error delta distance is only 7 meters. 8 bytes.

Radius -> GeoHash length

0.5 km -> 6

2 km -> 5

5 km -> 4

Steps of using GeoHash:

* Calculate a user's GeoHash string based on his/her geo location (latitude and longitude).

* Find the corresponding GeoHash grid which matches the user. Filter the locations in this grid and return.

If the size of grid is too small, we could search too many number of grids. If the grids are too big, the search result could contain those unnecessary far-away locations. Usually we search for up to nine (9) grids in total OR also we can include some neighbor grids, e.g. GeoHash6 >= "bc1234" and GeoHash6 <= "bc123s".

**Note**:

There are boundary issues/corner cases in using GeoHash: Two places can be very close but they have different GeoHash prefix name.

### Google S2

It is still an in-memory solution. It maps a sphere to a ID index based on the Hilbert curve (a space-filling curve).

S2 is great for geofencing because it can cover arbitrary areas with varying levels.

### Conclusion

GeoHash and QuadTree are preferred.

GeoHash - pros: Easy to implement. Updating the index is easy. cons: It can not dynamically adjust the grid size based on population density,

Still go for **GeoHash** approach.

## Data Partition

There are two tables: Business Table and Geo-spatial Index Table. Also we need to discuss QuadTree first.

### Quad Tree Partition

Sharding based on Region: will bring hot region and possible unbalanced distribution of places data those two problems.

Sharding based on LocationID: When we find places near a location, we have to query all servers and do aggregation operation from each server's results.

### Business Table

For Business Table, we simply do sharding based on Location/Business ID base on the three options as below.

1.**By regions or zip-code**

Cons:

The issue of hot places - One of the server in the cluster receive too many requests.

The issue of unevenly distributed data - Some regions can end up storing a lot of places compared to others.

2.**By LocationID**

This could make a request querying too many shard servers.

 To find places near a location, we have to query all servers and each server will return a set of nearby places. A centralized server will aggregate these results to return them to the user. May not be efficient.

3.**By Geo-hash/Google S2**

To avoid the issue of hot shard, try to make that cells are continuous in one shard and adjust the number of cells in one shard by the number of places.

Geo-hash is widely adopted in the open source community (e.g. ElasticSearch, MongoDB, and others).

### Geo-spatial Index Table

<geohash, business_id> instead of <geohash, list_of_business_ids>

500 million * (8 bytes per id + 8 bytes per geohash) = 8 GB

**It can be fit in the working set of a modern database server. However, depending on the read volume, a single database server might not have enough CPU or network bandwidth to handle all read requests. So there is no strong reason to shard among multiple servers. A better approach is to have a series of read replicas to help with the read load.**

## Replication And Fault Tolerance

Primary-Secondary Configuration.

Primary QuadTree server serves write traffic and applies to all other replicas (secondaries), which only serve read traffic.

If both primary and secondary machines die?

* Add a new server and rebuild the same (sub) QuadTree in its memory.

* Brute force to iterate the whole DB or maintain a Reverse Index (mapping locationIDs to which QuadTree servers).

## Cache

Between backend databases and application servers, we can store all businesses in a hot place or all detailed data for a hot business.

<geohash, list_of_business_ids>

and

<business_id, Business Object>

Least Recently Used (LRU) seems to be suitable in this case.

## Load Balancing

We should add LB layer between clients and application servers since one machine can not handle such high QPS.

Strategy: Round Robin (distributed equally)

More intelligent LB can also take traffic/load/server status/latency as consideration by periodically querying backend server and then adjust the traffic volumes.

## Ranking

E.g. While searching for the top 100 places within a given radius, we can ask each partition of the QuadTree to return the top 100 places with maximum popularity. Then the aggregator server can determine the top 100 places among all the places returned by different partitions.

Assuming the popularity of a place is not expected to reflect in high frequency like every hour, we can decide to update them once a day even once a few days.

## CDN

A CDN is a system of globally distributed servers that deliver web content to a user based on the geographic locations of the user, the origin of the web page and a content delivery server. CDNs replicate content in multiple places. User can get the content from the nearest CDN.

Push CDNs/Pull CDN

Disadvantage: expensive/read stale data.

However, we do not use CDN in this Yelp/Proximity case design.

## Reference

[1] <https://medium.com/swlh/design-a-proximity-server-like-nearby-or-yelp-part-1-c8fe2951c534>

[2] <https://codeburst.io/design-a-proximity-server-like-yelp-part-2-d430879203a5>

[3] <https://www.educative.io/courses/grokking-the-system-design-interview>

[4] <https://gis.stackexchange.com/questions/91788/are-there-trade-offs-between-using-a-geohash-vs-using-a-quadkey-as-a-spatial-ind>

[5] <https://www.cnblogs.com/wangby511/p/14293242.html>

[6] <https://www.cnblogs.com/wangby511/p/15641667.html>

[7] <https://www.bilibili.com/video/BV1GR4y1j7B9?t=573.0>

[8] <https://www.systemdesigntutorial.com/hld/yelp>
