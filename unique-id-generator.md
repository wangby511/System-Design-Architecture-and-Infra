# Unique ID - Twitter Snowflake Approach

1 bit sign bit + 41 bits timestamp + 5 bits datacenter + 5 bits machine id + 12 bits sequence number.

Need clock synchronization. Like 1636231966771 in millis (See https://currentmillis.com/)

(1 0111 1100 1111 0111 0000 0110 0111 0000 0011 0011)2 (41 bits)

(17CF7067033)HEX
