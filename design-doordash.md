# System Design - Doordash

CREATED 2022/08/29

DoorDash is an American-based prepared food delivery service.

Hint: think through every component, including the data model, keeping scalability, redundancy, fault tolerance, etc in mind.

Discuss the scope of the problem. Identify the users (actors) of your application.

## Functional Requirements

### Customers

Search for restaurants based on location, menu items, etc.

Add menu items to the cart

Place a order

Track the status of the order / Cancel the order

(Create/Update their account and contact information)

### Restaurants

Create their profile and add new menu items & pictures

Receive notifications about orders placed, assigned DoorDasher and update the status of an order

### DoorDashers

Receive notifications about the available orders in the area

Choose to pick up an order

(Inform the customer/restaurant of any problems they encounter while executing the order pickup/delivery)

## Reference

[1] <https://www.educative.io/courses/system-design-interview-doordash>
