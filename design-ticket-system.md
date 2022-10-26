# System Design - Ticket System

This blog is taking cinema booking system as an example.

UPDATED 2022-10-25

## Functional Requirements

Once the user selects a movie, the service should display the cinemas running that movie and their available show times.

The user should be able to choose a show time at a particular cinema and book their tickets.

The service should be able to show the user the seating arrangement of the cinema hall. The user should be able to select multiple available seats according to the map.

Users should be able to put a hold on the seats for 5 minutes before they make a payment to finalize the booking.

## Non-Functional Requirements

Highly Available

Highly concurrent & scalable to support the surge in traffic

Financial transactions follow the database ACID compliance

## Estimation

Traffic estimates: Let’s assume that our service has 3 billion page views per month and sells 10 million tickets a month.

QPS about 1k for viewing, less than 10 for booking ticket.

## APIs

searchMovie(keyword, city, radius, sorting_order, max_results, next_token)

searchCinemaForMovie(movie_id, city, radius, sorting_order, max_results, next_token)

searchMoviesInCinema(cinema_id, sorting_order, max_results, next_token)

searchSeats(cinema_id, room_id, start_time/show_id)

reserveSeats(movie_id, room_id, start_time/show_id, list of seats)

## Storage Database

User Table

Seats Table

Reservation Table

## High Level Design

Clients <-> Load Balancers <-> Web servers <-> Backend servers <-> Cache servers/Database servers

## Workflows

### Ticket Booking Workflow

Search for movies

Select a movie

Select a specific show with time/cinema_id.

Select a number of seats according to the map.

If the required number of seats are available, the user is shown a map of the theater to select seats. If not, the system will return a full/no available tickets error.

Once the user selects the seat, the system will try to reserve those selected seats.

If seats are reserved successfully, the user has five minutes to pay for the reservation. After payment, this transaction is marked complete. If the user is not able to pay within five minutes, all their reserved seats are released to become available to other users again.

## Components

### Booking service & Search service

Backend services for searching movie or booking seats for a cinema show.

### Queue

Put message into the queue whenever a reservation is made successfully (without payment). Set the delay for 5 minutes. Also we made a record to the Reservation DB and updates some seats' status inside the Seats DB.

### Queue Handler

Pull the message from the queue. Check each reservations and remove those seats in unsuccessful reservation back to available status again.

## Concurrency

We must guarantee that no two users are able to book the same seat. To achieve this, we can use transactions in SQL databases to avoid any clashes.

`Serializable` is the highest isolation level and guarantees safety from Dirty, Non-repeatable, and Phantoms reads.

## Fault Tolerance

Payment Service failure - Customers can not pay the ticket.

Queue or Queue Handler failure - Manually scan/check all the items in the Reservation/Booking Table. Compare the status with PENDING and creation timestamp 5 minutes ago.

We can also configure primary-secondary setup for databases to make them fault-tolerant.

## Partition

Partition by show_id instead of movie_id in Ticket Table.

## Reference

[1] <https://www.educative.io/courses/grokking-the-system-design-interview>

[2] <https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-delay-queues.html>
