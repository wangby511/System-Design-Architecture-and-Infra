# Chapter 1 - Reliable, Scalable, and Maintainable Applications

CREATED 2022/03/07

## Thinking About Data Systems

## Reliability

fault-tolerant or resilient.

Fault - one component of the system deviating from its spec.

Failure - the system as a whole stops providing the required service to the user.

* hardware faults

* software errors

* humane errors - e.g. configuration errors.

Narrow down and discourage "doing the wrong thing". Provide non-production sandbox environments. Add unit tests, integration tests and automated testing. Quick recovery /roll back. Detailed and clear monitoring.
