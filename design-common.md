# Common Knowledge for System Design Interview

CREATED 2022/03/22

## Interview Signals

Workable solution

Analysis and communication

Tradeoff/Pros and Cons

Knowledge base

## Common Non-Functional Requirements

Reliability - probability a system will fail in a given period. A distributed system is considered reliable if it keeps delivering its services even when one or several of its software or hardware components fail.

Scalability - capability of a system to handle or manage growing and increasing demands or requests.

Availability - simple measure of the percentage of time that a system, service, or a machine remains operational under normal conditions.

## Common Component

Use a queue - de-coupling the services and act as a buffer in case throttling

## Normal Follow Ups

Scaling database - vertical vs horizontal

SQL or NoSQL

Leader-follower replica

Database Sharding - hotkey issue and handling mechanisms

## Delivery Protocols

**Long Polling** - Long polling is technique where the server elects to hold a client connection open for as long as possible, delivering a response only after data becomes available or timeout threshold has been reached.

**WebSocket** - WebSocket is a computer communication protocol which provides full-duplex communication channels. Suitable for bi-directional real-time communication.

**Server-Sent Events(SSE)** - Unlike WebSockets, Server-Sent Events are a one-way communication channel where events flow from server to client only. Server-Sent Events allows browser clients to receive a stream of events from a server over an HTTP connection without polling.

## Reference

[1] <https://medium.com/geekculture/cracking-the-system-design-interview-theory-basics-c57f5326181b>

[2] <https://medium.com/system-design-blog/long-polling-vs-websockets-vs-server-sent-events-c43ba96df7c1>
