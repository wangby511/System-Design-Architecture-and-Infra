# System Design - Uber Backend

A ride-sharing service like Uber, which connects passengers who need a ride with drivers.

CREATED 2022-08-09

## Functional Requirements

Drivers need to notify the service about their current location and their availability to pick passengers regularly.

Passengers get to see all nearby available drivers.

Customer can request a ride; nearby drivers are notified that a customer is ready to be picked up.

Once a driver and a customer accept a ride, they share the same location until the trip finishes.

Upon reaching the destination, the driver marks the request as complete to become available for the next ride.

## Reference

[1] <https://www.educative.io/courses/grokking-the-system-design-interview/>
