# System Design - Notification System

SET UP 2022/02/13

## Functional Requirements

What types of notifications does the system support? Push notifications, SMS messages and email.

A soft real-time and a slight delay is acceptable.

Support iOS, Android and laptops.

Triggered by what system?

Can users choose to opt-out/unsubscribe?

QPS? Like 10 million mobile push notifications (~100 QPS), 1 million SMS messages and 5 millions emails per day.

## Different Types of Notifications

APNS - Apple Push Notification Service

FCM - Firebase Cloud Messaging

SMS message service like Twilio

Email service

## Workflows

The API servers should gather user's info like phone number, email address and store them into DB.

Note: A user can own multiple devices or terminals.

User Table: [userId], email, phone number, ...

Device Table: [deviceId], userId, device token

## Components

Notification system: The notification system is the centerpiece of sending/receiving notifications. It provided APIs for services 1 to N, and builds notification payloads for 3rd party services by starting from only one server.

While integrating with 3rd party services, we need to pay extra attention to **extensibility**.

Message Queues: remove dependencies between components ans serves as buffers when high volumes of notifications need to be sent out. "Each notification type is assigned with a distinct message queue so that an outage in one service will not affect others."

![img](https://www.durichitayat.net/static/f533dc948b73e1481c75f7cc4a0de13f/e3189/push-notification-service.png)

![img](https://systeminterview.com/imgs/top10/notification.png)

## Reference

[1] <https://www.systemdesigntutorial.com/hld/notification-system>

[2] <https://tianpan.co/notes/162-designing-smart-notification-of-stock-price-changes>

[3] <https://www.youtube.com/watch?v=C3iLSm_G5EY>

[4] System Design Interview. Alex Xu. Chapter 10. Design a notification system.
