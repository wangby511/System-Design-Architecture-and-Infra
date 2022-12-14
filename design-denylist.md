# Design DenyList

CREATED 2022-01-21

UPDATED 2022-12-06

## Functional requirements

Design a global deny list service. Among all the traffice, about 5% are from malicious IPs.

We can call isMalicious() API to determine whether an IP is malicious but this API has a bit high latency. If it is, we add it into deny list. Next time its request arrives, we block it directly and do not need to call this API again next time.

The difficulty is how to sync data between data centers and how to make trade off between availability and consistency. And how to handle single point of failure.

## Calculation

QPS 12000/s

5% bad IP 600/s

## Components

Bloom Filter: Bloom Filter can not be used here since a good IP address can be treated as a malicious IP due to some possibility.

Primary-secondary system: 4 billion IPs -> malicious are 4 billion x 5% = 200 million.

200 million x 4 Bytes/IP = 800MB which can be fit in one modern server. So we can write to primary machine and let the secondary machines syncing from it. Also to avoid single point of failure, we may add another backup primary machine by configuring a stand-by machine.

### Blacklist Cache/DB

## Reference

[1] <https://instant.1point3acres.com/thread/829894>
