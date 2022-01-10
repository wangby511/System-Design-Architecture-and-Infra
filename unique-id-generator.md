# Unique ID - Twitter Snowflake Approach

This is a 64-bit unique ID generator that was inspired by a similar service by Twitter called Twitter snowflake.

The IDs are made up of the following components:

1. Epoch timestamp in millisecond precision - 41 bits (gives us 69 years with a custom epoch)
2. Configured machine id - 10 bits (gives us up to 1024 machines)
3. Sequence number - 12 bits (A local counter per machine that rolls over every 4096)

IDs should ideally be 64 bits = 1 bit sign bit + 41 bits timestamp + 5 bits data center + 5 bits machine id + 12 bits sequence number.

Generated IDs should be sortable by time. But there is a hidden assumption that all ID generation servers have the same clock to generate the timestamp. Like 1636231966771 in millisecond (See https://currentmillis.com/)

(1 0111 1100 1111 0111 0000 0110 0111 0000 0011 0011)2 (41 bits)

(17CF7067033)HEX

## Others

MongoDB’s ObjectId is 12 bytes long and encodes the timestamp as the first component.

UUID is 128 bits long and it is completely random without natural sort.

## Reference

[1] <https://instagram-engineering.com/sharding-ids-at-instagram-1cf5a71e5a5c>

[2] <http://blog.gainlo.co/index.php/2016/06/07/random-id-generator/>

[3] <https://www.callicoder.com/distributed-unique-id-sequence-number-generator/>

[4] <https://aaronice.gitbook.io/system-design/architecture-toolbox/id-generator>
