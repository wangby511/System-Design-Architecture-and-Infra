# System Design - Notification System

SET UP 2022/02/13

A notification system is a system to send notification alerts or information updates to the users.

## Functional Requirements

The system should be able to send notifications to the subscribed consumers.

What types of notifications does the system support? Push notifications, SMS messages and email.

Support iOS, Android and laptops.

It should be an isolated system that is easy to integrate into an existing system.

### Other Factors

Can users choose to opt-out/unsubscribe?

QPS? Like 10 million mobile push notifications (~100 QPS), 1 million SMS messages and 5 millions emails per day.

Are we going to use 3rd party services or in-app?

## Non Functional Requirements

Available: Always available because send critical information is needed to sent to customers.

Latency: Latency should be low.

(Pluggable if using 3rd party service: Our notification system should allow additional channels without much effort and redesign.)

Scalable: The system should be scalable to cater to increased loads without much increase in latency.

Reliable: The data can not be lost.

## Different Types of Notifications

APNS - Apple Push Notification Service

FCM - Firebase Cloud Messaging

SMS message service - E.g. Twilio

Email service

## API

### POST /api/v1/SMS

```
{
    "user_ids" : [
        "1001", 
        "1011",
        ...,
        "1022"
    ],
    "sub": "The Big Festival sale starts in 10 minutes!",
    "body": {
        "type": "text/plain",
        "value": "Don't miss the latest deals, visit www.abcd.com now!"
    }
}
```

### Topic Related

1 createTopic(topicName) with api_dev_key, creatorId

2 publish(topicName, message) with api_dev_key, creatorId

3 subscribe(topicName) with api_dev_key, userId

## Workflows

The API servers should gather user's info like phone number, email address and store them into DB.

Note: A user can own multiple devices or terminals.

## DB Schema

User Table: [userId], email, phone number, timestamp, ...

(Device Table: [deviceId], userId, lastLoginTimestamp, ...)

Also there will be Topics and Subscriptions information tables.

Write heavy, read light - For NoSql, we could choose Cassandra, log-structured merge tree storage engine

## Components

Notification system: The notification system is the centerpiece of sending/receiving notifications. It provided APIs for services 1 to N, and builds notification payloads for 3rd party services by starting from only one server. While integrating with 3rd party services, we need to pay extra attention to **extensibility** (A flexible system that can easily plugging or unplugging of a third-party device).

It queries the database or cache to fetch data needed to render a notification. Also it does validation step before sending them to message queues for parallel processing.

Message Queues: **Decouple system components** and serves as **buffers when high volumes** of notifications need to be sent out. Each notification type is assigned with a distinct message queue so that an outage in one service will not affect others. (Asynchronous way of handling the messages, messages are buffered in the queue till they are picked by the workers for processing. Different queues are used for different channels like email, SMS, etc.)

Handlers: Various handlers are connected after to the messaging queue. The handler picks up the messages and sends them to the subscribers. Each handler is assigned to handle requests from a specific message type, e.g. the Email handler, the SMS handler.

Third-party Services: Send notifications to users' devices.

## State

START -> PENDING -> SENT (OR ERROR) -> DELIVER -> CLICK / UNSUBSCRIBE

A async thread scans the DB for the records newly saved with PENDING status.

## Infra Graph

![img](https://www.durichitayat.net/static/f533dc948b73e1481c75f7cc4a0de13f/e3189/push-notification-service.png)

![img](https://miro.medium.com/max/875/0*Lc7xwqKOC-LTL6CL)

## Others

### Reliability

We should avoid any data loss even during the peak traffic.

1 We can get acknowledgement from workers that the message has been sent successfully.

2 We can persist notification data into append-only log like **Notification Log**.

### De-dup

If the message queue's implementation is following at least once delivery, we can check the event/notification ID in server side.

Client side - store a rotation of message IDs in an only period of time (Like recent 10k slots or most recent 2 hours). E.g. iOS App.

### Email Or Message Template

Fill in the content with user name, date and detailed activity words.

### Rate Limiter

The rate limiter checks the messages for two things:

* If the publisher is allowed to send that many requests.

* If the subscriber can receive as many notifications. Consumer/Users can choose to mute or opt-out specific channel.

### User Setting

Before we send data, we should check whether a user would like to receive messages by checking DB.

[userId, channel, opt_in(boolean), ...]

### Retry Mechanism

We should retry those unsuccessfully consumed messages and put still unsuccessfully processed messages into dead-letter queue after a fixed times of retrying.

### Order

In this case, the order of messages can not be guaranteed in order to maintain availability, except some special use cases like discount.

In-order: E.g. FIFO queue + hash on userId. The same person's messages are going to the same queue based on hash. SequenceId(Auto-incremental, generated in Workers side) may also be applied in the client side in case some late messages arrive first. The client side could check every 5 seconds to make the messages in order.

### Priority of Messages

Three queues are maintained on the basis of priority: high, medium, and low. The events from the high priority queue are picked first with a fast rate, then the medium priority and then the low priority at a slower rate.

1 OTP password for login

2 Transaction notification

3 Promotion message like advertisements

### Monitoring/Analytics Service

Tracking and monitoring from users' feedback by collecting engagement statistics like open rate and click through rate.

A service called “Notification Tracking” continuously reads events from the messaging queue. It maintains notifications metadata for system analytics. It is composed of three elements: Message queue, Notification Tracker, Cassandra Cluster and Analytics Service.

### Follow-up

At-least-once, exactly-once(cost is high, like financial system), delivery with ordering. Most important - stable delivery.

When is the client side going to send ACK? - After client side finishes the application logic or writes to local DB in case it fails.

How to handle offline mobiles? - When the mobile app goes online again, it is going to pull messages from somewhere again.

Differentiate the border of each micro services.

As long as we use the 3rd part service like APNs, we can not rely on customers' click as ACK message. The idea is from video [3].

## Reference

[1] <https://www.systemdesigntutorial.com/hld/notification-system>

[2] <https://tianpan.co/notes/162-designing-smart-notification-of-stock-price-changes>

[3] <https://www.youtube.com/watch?v=C3iLSm_G5EY>

[4] System Design Interview. Alex Xu. Chapter 10. Design A Notification System.

[5] <https://medium.com/double-pointer/system-design-interview-notification-service-86cb5c266218>
