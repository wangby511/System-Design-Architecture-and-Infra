# System Design - Google Maps

FIRST CREATED 2022/01/24

## Functional Requirements

Locations update

Direction/Navigation/Route Search

Map rendering

## Map Rendering

The world’s map is projected into a huge 2D map image (3D to 2D is called **Map Projection**). It is broken down into small image blocks called "tiles" which are static. They don’t change very often. An efficient way to serve static tile files is with a CDN backed by cloud storage like S3. The users can download the necessary nearby tiles to compose a map from nearby CDN.

The server pre-calculates the map blocks with different zoom levels loaded by the users and return the images when needed.

### Tile level

**Geo-hashing** - encodes a geographic area into a short string of letters and numbers. We use geo-hashing for map tiling.

For example, at zoom level 0, The entire map is represented by a single tile of size 256 x 256 pixels. Then at zoom level 1, the number of map tiles doubles in both north-south and east-west directions, while each tile stays at 256 x 256 pixels. So we have 4 tiles at zoom level 1, and the whole image of zoom level 1 is 512 x 512 pixels.

With each increment, the entire set of tiles has 4 times as many pixels as the previous level. The increased pixel count provides an increasing level of details to the user.

### Road Segments

The road can be considered a list of many road segments. And two places are connected if the corresponding road segments are reachable. In this way, finding a path between two locations becomes a shortest-path problem where we can leverage Dijkstra or A* algorithms.

## Navigation Service

This service is responsible for finding a reasonable route from point A to point B.

### Geocoding Service

Resolve the given address to a latitude/longitude pair.

### Route Planner Service

This service does three things in sequence:

Calculate top-K shortest paths from A to B.

Calculate the distance and estimation of time for each path based on current traffic and historical data.

Rank the paths by time predictions and user preference.

## Reference

[1] <https://www.linkedin.com/posts/alex-xu-a8131b11_systemdesign-coding-interviewtips-activity-6891421685749313536-g9_6>

[2] <https://www.linkedin.com/posts/alex-xu-a8131b11_systemdesign-coding-interviewtips-activity-6889974038777724928-G3u2>

[3] System Design Interview Book Volume II. Alex Xu. Chapter 3. Google Maps.
