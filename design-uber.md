# System Design - Uber Backend

CREATED 2022-08-09

A ride-sharing service like Uber, which connects passengers who need a ride with drivers.

Uber enables its customers to book drivers for taxi rides. Uber drivers use their personal cars to drive customers around. Both customers and drivers communicate with each other through their smartphones using Uber app.

## Functional Requirements

Drivers need to notify the service about their current location and their availability to pick passengers regularly.

Passengers can **see all nearby available drivers.**

Customer can **request a ride** by entering pickup & drop-off location and the nearby drivers are notified that a customer is ready to be picked up.

The driver can choose to **accept or ignore** this ride.

Once a driver and a customer accept a ride, they share the same location until the trip finishes.

Upon reaching the destination, the driver marks the request as complete to become available for the next ride.

Generate the receipt and continue payment after the ride.

**Summary: See drivers in your nearby area; ETA and approximate price; Book a ride; Location tracking.**

## Non-Functional Requirements

Scalability

Availability

## Capacity Estimation

We have 300M customer and 1M drivers, with 1M daily active customers and 500K daily active drivers.

We have 10M rides per day. About 100 rides per second.

All active drivers notify their current location every three seconds.

## Table Schema

### Drivers Location Table

driverId(primary key), driver name, latitude, longitude, last updated time.

### Customer Info Table

customerId(primary key), customer name, email, phone, photo url, ...

### Trip Record Table

tripId(primary key), driverId, customerId, pick up location, destination, status, routeId, start time, end time...

## Flows

### Request a Ride

When a customer requests a ride, the service find all the nearby drivers according to the customer's location.

[lat, lng] -> geohash -> [driver1, driver2, ...]

If not found enough, expand the search area to search for more drivers.

### Match a Driver

Change status in Driver Table and Trip Table. The customer can track the matched driver's location.

driver X -> [lat, lng]

### In Trip

Change status in Trip Table, like PICKING -> PICKED.

### Trip End

Change status in Driver Table, let the driver available again, and Trip Table like PICKED -> END. Send to payment service for charge fee.

## Details

### QuadTree

Unlike designing Yelp, here we need huge amount of fast updates in drivers' locations by removing a driver from the previous grid and move/reinsert to the current location grid.

### Hash Table

```bash
DriverID (3 bytes - 1 million drivers)

Old latitude (8 bytes)

Old longitude (8 bytes)

New latitude (8 bytes)

New longitude (8 bytes)

(May be we should add lastUpdatedTimeStamp, newTimeStamp)
```

Total = 35 bytes, the total memory is 1 million * 35 bytes = 35 MB

If we receive this information (19 Bytes) every three seconds from 500K daily active drivers, we will be getting 9.5MB per three seconds.

Since all this information can easily be stored on **one server** but, for scalability, performance, and fault tolerance, we should distribute DriverLocationHashTable onto multiple servers. And we can update QuadTree server every 10 or 15 seconds.

### How to efficiently broadcast?

We can build our Notification service on a publisher/subscriber model.

Maintain a list of customers (subscribers) interested in knowing the location of a driver and, whenever we have an update in DriverLocationHashTable for that driver.

Suppose a driver has 5 subscribed customers, we will need 500K *3 + 500K* 5 * 8 ~= 21 MB of memory.

bandwidth 5 *500K* 19 bytes = 47.5 MB/s

### Push mode or Pull mode

* For the Notification service, we can either use **HTTP long polling** or **push notifications**.

* For how does the customer know what drivers are near-by? **Pull** mode is simpler here.

Clients can send their current location (only if then open app and they are going to make a request), and the server will find all the nearby drivers from the QuadTree to return them to the client. The client can refresh their screen and query every 5 seconds e.g. to reflect the current positions of the drivers.

Pros: Save resources for those inactive users.

## Components of Services

## Ride Request Service

Customers can book a request.

## Drivers Service

Drivers' locations will be reported every a few seconds.

## Location/Map Service

The location service will store the location information in a Cassandra. It also talks to a map service to figure out which area the driver belongs to and update the mapping of which drivers belong in which area.

## Notification Service

Notify both drivers and customers. All the active drivers who are ready to accept trips are connected to a WebSocket handler. Websocket handlers will keep an open connection with the drivers to get location pings or send notifications about trips.

## Trip Service & DB

The Trip Service exposes all trip-related APIs, like get or list ride trip(s).

Trip DB, also known as Ride DB. And we can have some archived data to move to Ride History DB.

## Payment Service & DB

MYSQL based DB. The payment service will also be connected with a payment gateway.

## Steps

### How would "Request Ride" use case work?

The customer will put a request for a ride.

One of the [**Aggregator servers**] will take the request and asks [**QuadTree servers**] to return nearby drivers.

The [**Aggregator server**] collects all the results and sends notifications to those drivers simultaneously by using [[**Notification Server**].

The driver accepts the request first will be assigned the ride. The other drivers will receive an out-of-date request. If none of the three drivers respond, the Aggregator will pick another list of nearby drivers.

Once a driver accepts a request, the customer is notified. The customer can see the specific driver's location.

## Fault Tolerance

Primary & Secondary with persistent storage for recovery.

## Follow Up

### How will we handle clients on slow and disconnecting networks?

Adjust the frequency of client's request based on client side's internet speed?

Client lost contact? Show "Internet Not Connected" on app side. Check current status of Trip Table and Driver/Customer Table to see if there is an unfinished trip.

## Reference

[1] <https://www.educative.io/courses/grokking-the-system-design-interview/>

[2] <https://xunhuanfengliuxiang.gitbooks.io/system-desing/content/design-uber.html>
