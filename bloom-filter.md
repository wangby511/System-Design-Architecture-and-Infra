# Bloom Filter

A Bloom filter is a data structure used to detect whether an element is in a set in a time and space efficient way.

An empty Bloom filter is a bit-array of m bits, all set to 0. There are also k different hash functions, each of which maps a set element to one of the m bit positions.

To add an element, feed it to the hash functions to get k bit positions, and set all the bits at these positions to 1.

To test if an element is in the set, feed it to the hash functions to get k bit positions. If any of the bits at these positions is 0, the element is definitely not in the set. If all are 1, then the element may be in the set.

E.g.

[0, 0, 0, 0, 0, 0, 0, 0, 0, 0] and if hash(key1) = {3,6,7}

Then bits array = [0, 0, 0, 1, 0, 1, 0, 1, 0, 0]

False positive matches are possible, but false negatives are not.

## Explanation

Bloom filters are a probabilistic data structure for set membership testing that may yield false positives. A large bit vector represents the set. An element is added to the set by computing N hash functions of the element and setting the corresponding bits. An element is deemed to be in the set if the bits at all N of the element’s hash locations are set. Hence, a document may incorrectly be deemed to be in the set, but false negatives are impossible.

## Used Example & Applications

How to avoid crawling duplicate URLs at large scale? - Introduce a big Bit Vector and a few hash functions. Google Chrome uses bloom filters to spot malicious URLs. The URL entered by the user is looked up in a bloom filter.

![image](https://media-exp1.licdn.com/dms/image/C5622AQFRyCkWiwDXFw/feedshare-shrink_2048_1536/0/1649259170391?e=2147483647&v=beta&t=nM_HQQn9M8Yy2BljknwBm5brb2qud9UCSswHaNEI9a4)

Databases like Apache Cassandra, Google BigTable and Apache HBase optimize performance of query operations by lowering the number of searches for non-existent rows and columns.

## Conclusion

A bloom filter is a data structure (similar to hash table) designed to tell you, rapidly and memory-efficiently, whether an element is present in a set.

It could be **false positive**, but false negatives are impossible. Therefore it can avoid **cache miss attack**(A scenario that a user queries a key which doesn't exist in the database nor cached. So the database can easily be overloaded.).

However, increasing the number of hash functions also adds latency to the insertion and lookup operations of the bloom filter. The time complexity for a bloom filter is O(k) where k is the number of hash functions involved.

## Reference

[1] <https://www.educative.io/courses/grokking-the-system-design-interview>

[2] <https://www.linkedin.com/posts/alex-xu-a8131b11_systemdesign-coding-interviewtips-activity-6917494340315463680-O0sG?utm_source=linkedin_share&utm_medium=member_desktop_web>
