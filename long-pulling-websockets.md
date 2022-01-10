# Long-Polling & WebSockets

## Long-Polling

With Long-Polling, the client requests information from the server exactly as in normal polling, but with the expectation that the server may not respond immediately.

The client makes an initial request using regular HTTP and then waits for a response. The server delays its response until an update is available or a timeout has occurred. When an update is available, the server sends a full response to the client.

## WebSockets

WebSocket provides a full duplex communication channels over a single TCP connection.

It provides a persistent connection between a client and a server that both parties can use to start sending data at any time. Therefore, a two-way (bi-directional) ongoing conversation can take place between a client and a server.

## Server-Sent Events (SSEs)

Under SSEs the client establishes a persistent and long-term connection with the server. The server uses this connection to send data to a client. If the client wants to send data to the server, it would require the use of another technology/protocol to do so.

1. Client requests data from a server using regular HTTP.
2. The requested webpage opens a connection to the server.
3. The server sends the data to the client whenever there’s new information available.

SSEs are best when we need real-time traffic from the server to the client or if the server is generating data in a loop and will be sending multiple events to the client.

## Reference

[1] <https://aaronice.gitbook.io/system-design/distributed-systems/long-polling-vs-websockets-vs-server-sent-events>
