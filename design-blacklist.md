# Design BlackList

## Functional requirements

We call isMalicious() API to determine whether an IP is malicious but this API has a bit high latency. If it is, we add it into blacklist. Next time its request arrives, we block it directly and do not need to call this API again next time.

QPS 12000/s

3% bad IP 600/s

## Components

### Bloom Filter

### Blacklist Cache/DB
