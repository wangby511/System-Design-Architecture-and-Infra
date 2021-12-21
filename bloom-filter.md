# Bloom Filter

A Bloom filter is a data structure used to detect whether an element is in a set in a time and space efficient way.

An empty Bloom filter is a bit-array of m bits, all set to 0. There are also k different hash functions, each of which maps a set element to one of the m bit positions.

To add an element, feed it to the hash functions to get k bit positions, and set the bits at these positions to 1.

To test if an element is in the set, feed it to the hash functions to get k bit positions. If any of the bits at these positions is 0, the element is definitely not in the set. If all are 1, then the element may be in the set.

E.g.

[0, 0, 0, 0, 0, 0, 0, 0, 0, 0] and if hash(key1) = {3,6,7}

Then bits array = [0, 0, 0, 1, 0, 1, 0, 1, 0, 0]

False positive matches are possible, but false negatives are not.

E.g. Cassandra uses Bloom filters to determine whether an SSTable has data for a particular row.
