# System Design - Uber Backend

CREATED 2022-08-09

A ride-sharing service like Uber, which connects passengers who need a ride with drivers.

Uber enables its customers to book drivers for taxi rides. Uber drivers use their personal cars to drive customers around. Both customers and drivers communicate with each other through their smartphones using Uber app.

## Functional Requirements

Drivers need to notify the service about their current location and their availability to pick passengers regularly.

Passengers get to see all nearby available drivers.

Customer can request a ride; nearby drivers are notified that a customer is ready to be picked up.

Once a driver and a customer accept a ride, they share the same location until the trip finishes.

Upon reaching the destination, the driver marks the request as complete to become available for the next ride.

Summary: request a ride, accept/reject the ride.

## Non-Functional Requirements

## Capacity Estimation

We have 300M customer and 1M drivers, with 1M daily active customers and 500K daily active drivers.

We have 1M daily rides.

All active drivers notify their current location every three seconds.

## Table Schema

### Drivers Location Table

driverId(primary key), driver name, latitude, longitude, last updated time.

### Customer Info Table

customerId(primary key), customer name, email, phone, photo url, ...

### Customer Location Table

customerId(primary key), customer name, latitude, longitude, last updated time.

### Trip Table

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

## Reference

[1] <https://www.educative.io/courses/grokking-the-system-design-interview/>

[2] <https://xunhuanfengliuxiang.gitbooks.io/system-desing/content/design-uber.html>
