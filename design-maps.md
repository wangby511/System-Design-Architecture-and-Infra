# System Design - Google Maps

FIRST CREATED 2022/01/24

## Functional Requirements

User location update

Direction/Navigation/Route Search - considering traffic conditions

Map rendering - TB of raw map data

Places, photos and reviews (Not discussed here)

## Estimation

1 billion DAU

For navigation service, one user uses 35 minutes per week. => 5 billion minutes per day

Send GPS coordinates every second - 3 million QPS

If changed to send GPS every 15 seconds => 200k QPS

## Location Service

Mobile user -> Load Balancer -> Location Service -> User Location DB

Locations history can be buffered on the client (by every second) and sent in batch to the server at a lower frequency.

POST /v1/locations - (latitude, longitude, timestamp)

1 million location updates every second - need a database that supports fast writes, supports availability and partition tolerance. Cassandra is a good fit.

Schema: user id (hash key), timestamp (range key), lat, long, user mode, navigation mode

It can be linked to a Kafka which can further supports Traffic Update Service, Machine Learning Service for Personalization, Routing Tile Processing Service or even Analysis.

## Map Rendering

The world’s map is projected into a huge 2D map image (3D to 2D is called **Map Projection**). It is broken down into small image blocks called "tiles" which are static. They don’t change very often. An efficient way to serve static tile files is with a CDN backed by cloud storage like S3. The users can download the necessary nearby tiles to compose a map from nearby CDN.

### Tile

**Geo-hashing** - encodes a geographic area into a short string of letters and numbers. We use geo-hashing for map tiling.

For example, Google Maps uses 21 levels. At zoom level 0, the entire map is represented by a single tile of size 256 x 256 pixels. Then at zoom level 1, the number of map tiles doubles in both north-south and east-west directions, while each tile stays at 256 x 256 pixels. So we have 4 tiles at zoom level 1, and the whole image of zoom level 1 is 512 x 512 pixels. Then it does the recursion of splitting into 4 tiles with increasing each level. With each increment, the entire set of tiles has 4 times as many pixels as the previous level. The increased pixel count provides an increasing level of details to the user.

Tiling - The server pre-calculates the map blocks with different zoom levels loaded by the users and return the relevant tiles for the area the user is currently in by specific zoom level.

Optimization - use vector tiles(compression and zoom experience are both much better).

### Road Segments

The road can be considered a list of many road segments. And two places are connected if the corresponding road segments are reachable. In this way, finding a path between two locations becomes a shortest-path problem where we can leverage Dijkstra or A* algorithms.

Routing tiles - For each grid, we convert the roads within the grid into a small graph data structure that consists of nodes and edges(roads) inside the geographical area covered by the grid.

Hierarchical Routing tiles - especially 3 levels: local roads level, arterial roads level and major state/highways level.

### Storage Estimation

100KB x (1 + 4 + 16 + ... + 4^21) = 440 PB. But it can be compressed into 100 PB.

### Steps

Serve a pre-generated set of map tiles at each zoom level. These static images are stored in CDN with POPs(Point of Presence) closer to the users.

Mobile user <-> CDN <-> Pre-computed Map Images(Origin)

When a user moves to a new location or to a new zoom level, the map tile service determines which tiles are needed and translates that information into a set of tile URLs to retrieve.

## Navigation Service

This service is responsible for finding a reasonable route from point A to point B.

GET /v1/nav?origin=1149+37pl+street,LA&destination=Ferry+Building,SF&mode=DRIVE

### Geocoding Service

Resolve the given address to a latitude/longitude pair.

GET api/geocode/json?address=1149+37pl+street,LA - to get the latitude and longitude before passing to downstream services.

### Route Planner Service

This service does three things in sequence:

Calculate top-K shortest paths from A to B.

Calculate the distance and estimation of time for each path based on current traffic and historical data.

Rank the paths by time predictions and user preference.

1 Shortest-Path Service - starting from the original routing tile, expands larger to additional neighboring tiles and uses A* path-finding algorithms until a set of best routes is found.

2 ETA Service - calculates the time based on machine learning and historical data.

3 Ranker Service - compare different routes sorted by time, distance. filtered by toll roads, freeway roads.

4 Updater Service - update current traffic conditions, new roads or closed roads.

## Follow Up

Find out whether a user is affected by a nearby traffic change or a shorted ETA is found.

Delivery protocols - push data to clients. (Not using mobile push notification nor long-polling) We prefer WebSocket(bi-directional communication) and SSE(Server-Sent Events).

## Reference

[1] <https://www.linkedin.com/posts/alex-xu-a8131b11_systemdesign-coding-interviewtips-activity-6891421685749313536-g9_6>

[2] <https://www.linkedin.com/posts/alex-xu-a8131b11_systemdesign-coding-interviewtips-activity-6889974038777724928-G3u2>

[3] System Design Interview Book Volume II. Alex Xu. Chapter 3. Google Maps.
