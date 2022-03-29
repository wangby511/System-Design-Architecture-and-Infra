# Unique ID - Twitter Snowflake Approach

Twitter creates its own random ID generator called **Twitter Snowflake** to create unique IDs for tweets, users, messages and all other objects that need identification.

This is a 64-bit unique ID generator that was inspired by the above similar service.

## Functional Requirements

* Each time the ID generator is called, it returns a key that is unique across all node points for the system.

* The ID has a timestamp. A timestamp allows the user to order the entries according to the time they were created.

## Non Functional Requirements

* The system should be highly consistent. There should not be any duplicate IDs created across all data centers.

* The system should be able to scale to accommodate a large number of requests each second.

## Solution

1. Epoch timestamp in millisecond precision - 41 bits (gives us 69 years with a custom epoch)
2. Configured machine id - 10 bits (gives us up to 1024 machines)
3. Sequence number - 12 bits (A local counter per machine that rolls over every 4096)

IDs should ideally be 64 bits = 1 bit extra bit + 41 bits timestamp + 5 bits data center + 5 bits machine id + 12 bits sequence number.

Generated IDs should be sortable by time. But there is a hidden assumption that all ID generation servers have the same clock to generate the timestamp. Like 1636231966771 in millisecond (See https://currentmillis.com/)

(1 0111 1100 1111 0111 0000 0110 0111 0000 0011 0011)2 (41 bits)

(17CF7067033)HEX

Pros:

It can accommodate 1024 machines where each machine can generate 4096 unique IDs in each millisecond.

Cons:

On the downside, the design requires Zookeeper to maintain machine IDs.

It may not be applicable to applications where security is a requirement since the future IDs are predictable.

The 41 bits of time gives the system 69.73 years before rolling back.

## Flow

new request -> (LB) -> (App Server) <-> (ID Generator)

(App Server) <-> (Database)

## Others

MongoDB’s ObjectId is 12 bytes long and encodes the timestamp as the first component.

UUID is 128 bits long and it is completely random without natural sort.

## Reference

[1] <https://instagram-engineering.com/sharding-ids-at-instagram-1cf5a71e5a5c>

[2] <http://blog.gainlo.co/index.php/2016/06/07/random-id-generator/>

[3] <https://www.callicoder.com/distributed-unique-id-sequence-number-generator/>

[4] <https://aaronice.gitbook.io/system-design/architecture-toolbox/id-generator>

[5] <https://medium.com/double-pointer/system-design-interview-scalable-unique-id-generator-twitter-snowflake-or-a-similar-service-18af22d74343>
