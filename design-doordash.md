# System Design - Doordash

CREATED 2022/08/29

DoorDash is an American-based prepared food delivery service.

Hint: think through requirements, every component, including the data model, keeping scalability, redundancy, fault tolerance, etc in mind.

Discuss the scope of the problem. Identify the users (actors) of your application.

## Functional Requirements

### Customers

Search for restaurants based on location, food item name, etc.

Add menu items to the cart

Place a order

Track the status of the order / Cancel the order

(Create/Update their account and contact information)

### Restaurants

Create their profile and add new menu items & pictures

Receive notifications about the newly placed orders and the assigned DoorDasher

Update the status of an order.

### DoorDashers

Receive notifications about the available orders in the area

Choose to pick up an order & should be able to know when the order is available for pickup

(Inform the customer/restaurant of any problems they encounter while executing the order pickup/delivery)

## Non-Functional Requirements

Latency - should be acceptable and not too high.

Consistency - should be critical on orders. But thecconsistency requriement is not high for menu updates (eventual consistency).

Availability - high

High throughput

In general, the searching of menus/restaurants will be read-heavy and the ordering functionality will be write-heavy.

## Out of scope

Recommendations, Maps, ETA

## Estimation

Number of customers: 40M. Every customer place an order every day. Order QPS - 400. Peak - 1000.

## Data Model

### User Table

UserId, address, email, phone, ...

Same as doordasher Table, adding another location attribute

### Order Table

OrderId, userId, restaurantId, foodList, status, timestamp, ...

### Restaurant Table

RestaurantId, location, description, foodItemList, ...

### Menu Table

FoodItem, pictureUrl, description, price, ...

### Where to store?

We choose **a mix of relational and non-relational databases** for our data storage.

The amount of data of restaurants, menu descriptions, user data, Dasher data, etc., is going to be huge and the structure of data, especially attributes might also vary from restaurants to restaurants. Hence, a NoSQL/ columnar database like Cassandra could be used.

Pictures (restaurants, menu items) can be stored in an object storage.

Ordering is a transactional process and can be stored in relational databases.

### Data partitioning

Based on:

* Area code - cons: hot key problem

* pure RestaurantId

* Menu items/cuisines/dishes - cons: Need to query a few partitions to retrieve a resaurant's full menu

* Combination of area code and RestaurantId

## Components

### UI Client

Via mobile, web, tablets, etc.

Four actors - customer, restaurant, DoorDasher and admin.

### Restaurant Search Service

**Elasticsearch** has a Geo-distance query, which can be leveraged to return all the restaurants/menus that the user is searching for based on a defined radius from the user’s location.

Step 1 - When the restaurants create/update profiles/menu/items to [**Restaurent Profile Service**], it will not only do CRUD to database, but also post an event to the queue.

Step 2 - The [**Data Indexer**] then picks up the event, runs a query against the database to formulate a document as per the correct format, and posts the data into the search cluster.

Step 3 - The [**Restaurant Search Service**] query the search cluster or look up data in the cache. Then return the list of items to the user client.

### Ordering Service

Order placement is transactional in nature, the best idea is to use a relational database for Orders DB.

Mainly for users to place orders. Also the users can retrive history or orders as well as check the status of the order.

### Order Fulfilment Service

Mainly for restaurants to update order status (CREATING -> READY_FOR_PICKUP) and for doordashers to update status (READY_FOR_PICKUP -> DISPATCHING -> DELIVERED).

### DoorDasher Dispatch Service

Subscribe from the orders queue, send them to the nearby doordashers for them to choose to accept.

### User Profile Service & Restaurant profile service

E.g. CRUD for users' profiles or restaurant's menu dishes, images

### External Payment Gateway

Connected to [**Ordering Service**]

The interaction with e.g. PayPal, ApplePay, Visa, Mastercard, should be synchronous in nature.

### Notification Service

Actors could also receive in-app notifications. The service is responsible to notify:

To Customer - an order is placed successfully, acceptance of order by restaurant, or the dispatch of the order, and finally delivered.

To Restaurant - an order is newly placed, or a DoorDasher chooses the order, or that the DoorDasher is on their way to pick up the order.

To DoorDasher - an order is available to choose, the order is ready to be picked up, etc.

## System APIs

searchFoods(name, radius, location, maxResults, nextToken)

addToCart(restaurantId, foodItem, specificFilter)

placeOrder(restaurantId, foodItemsList, deliverTimeChoice)

checkOrderStatus(orderId)

get/retrieveOrderDetail(orderId)

cancelOrder(orderId)

## Replication & Fault Tolerance

Horizontally scalable.

Each components should have a replica/backup. Moreover, the NoSQL infrastructure will also have multiple nodes, and the queues can have partitions and replication as well.

## Load Balancing

Since there are multiple instances running for every service.

## Cache

Images of restaurants and dishes could be cached. The cache will hold the popular or most-ordered menu items/restaurants in a particular area.

Redis, Memcached.

Least Recently Used (LRU) or Least Frequently Used (LFU) algorithm.

CDN is not useful here.

## Reference

[1] <https://www.educative.io/courses/system-design-interview-doordash>
