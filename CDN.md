# Content Delivery Network (CDN)

As a system design case

CREATED 2022/01/14

## Why needs CDN

If millions of users worldwide use data-intensive applications and the service is deployed in a single data center to serve the users’ requests, there could be a **high-latency** problem.

For general requirements, real-time applications require a latency below 200 milliseconds(ms) in general. For the Voice over Internet Protocol (VoIP), latency should not be more than 150 ms.

## Functional Requirements

Save/Deliver - The origin servers should be able to send the content to CDN.

Retrieve based on request - The CDN should be able to execute a search against a user query for cached or otherwise stored content within the CDN infrastructure.

Respond - The CDN proxy servers should be able to respond to each user’s request.

## Non-functional Requirements

Performance - Minimizing latency

Availability - Be available at all times

Scalability - Be able to scale horizontally as the requirements increase

Reliability - Should ensure no single point of failure and keep the content not lost.

## Concept

**A CDN is a network of servers around the world that delivers content in different geographical locations with reduced latency. If the server you are getting content from is closer to your geographic location, the content will take less time (reduced latency) to be delivered from the server to you.**

We can put more popular items, like a file, video, audio, into CDN and leave those less popular items just into various data centers. A CDN caches an only considerable portion of the content depending on its capabilities, and it mostly caches the static content.

Use case: Blob (Binary Large Object) storage is used in combination with CDN as a data store for images, videos, etc.

## Infra Graph

```
                        ↙   Distribution System   ↖

Routing System <->  Clients       <->      Edge Proxy Servers   <->   Origin Servers

                                                    ↘  Management System ↙
```

## API Definitions

### Retrieve

from proxy server to origin server

retrieveContent(proxy_server_id, content_id, content_version)

### Deliver

from origin server to proxy servers

deliverContent(origin_id, proxy_server_id, content_id, content_version)

### Request

from clients to proxy servers

requestContent(user_id, proxy_server_id, content_id, content_version)

### Others

from proxy server to peer proxy servers

searchContent(content_id, content_version)

updateContent(content_id, content_version)

## Reference

[1] <https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/>
