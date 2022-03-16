# Chapter 1 - Reliable, Scalable, and Maintainable Applications

CREATED 2022/03/07

## Thinking About Data Systems

Some non-functional requirements like in system design: security, reliability, compliance, scalability, compatibility and maintainability.

## Reliability

fault-tolerant or resilient - makes systems work correctly, even when faults happen.

Fault - one component of the system deviating from its spec.

Failure - the system as a whole stops providing the required service to the user.

* hardware faults

* software errors

* humane errors - e.g. configuration errors.

Narrow down and discourage "doing the wrong thing". Provide non-production sandbox environments. Add unit tests, integration tests and automated testing. Quick recovery /roll back. Detailed and clear monitoring.

## Scalability

A system's ability to keep good performance with increased load.

E.g. Tweet's home timeline - hybrid of fan out and fetch on read together based on the number of followers per user

**throughput** - the number of records the system can process per second

**response time** - the time between a client sending a request and receiving a response

**percentiles** - measurement of statistics by sorting a list pof data in some order, take x percentage and take the value. E.g. p95, p99, p99.9 latency, ignoring some tail latencies.

**service level objectives(SLO) AND service level agreement(SLA)** - define the expected performance and availability of a service.

**scaling up** - vertical scaling, moving to a more powerful machine

**scaling out** - horizontal scaling, distributing the load across multiple smaller machines

**elastic** - detect load increase/decrease, and automatically the corresponding increase/decrease computing resources

## Maintainability

Operability (Keep the system running smoothly), Simplicity (Reduce complexity), Extensibility (Easy to make changes in the future)
