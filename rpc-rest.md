# RPC & REST

RPC: Remote Procedure Call is a software communication protocol that one program can use to request a service from a program located in another computer on a network without having to understand the network's details. RPC is used to call other processes on the remote systems like a local system.

REST, or REpresentational State Transfer, is an architectural style for providing standards between computer systems on the web, making it easier for systems to communicate with each other.

## Difference examples

The RPC API thinks in terms of "verbs", exposing the restaurant functionality as function calls that accept parameters, and invokes these functions via the HTTP verb that seems most appropriate - a 'get' for a query, and so on, but the name of the verb is purely incidental and has no real bearing on the actual functionality, since you're calling a different URL each time. Return codes are hand-coded, and part of the service contract.

The REST API, in contrast, models the various entities within the problem domain as resources, and uses HTTP verbs to represent transactions against these resources - POST to create, PUT to update, and GET to read. All of these verbs, invoked on the same URL, provide different functionality. Common HTTP return codes are used to convey status of the requests.

## Example

Placing an Order:

RPC: http://MyRestaurant:8080/Orders/PlaceOrder (POST: {Tacos object})
REST: http://MyRestaurant:8080/Orders/Order?OrderNumber=asdf (POST: {Tacos object})

Retrieving an Order:

RPC: http://MyRestaurant:8080/Orders/GetOrder?OrderNumber=asdf (GET)
REST: http://MyRestaurant:8080/Orders/Order?OrderNumber=asdf (GET)

In conclusion, the key difference is that REST is noun-centric and RPC is verb-centric.

## Reference

[1] <https://stackoverflow.com/questions/26830431/web-service-differences-between-rest-and-rpc>
