# Memcached

Memcached is an in-memory key-value store for small chunks of arbitrary data (strings, objects) from results of database calls, API calls, or page rendering.

Memcached supports a key of only 250 bytes and values are capped at 1MB by default. Memcached only stores data as strings.

Memcached is, by design, very simple to set up and use. If you had an extremely simple application on only a few servers, and only required simple string interpretation for your application.

When you restart memcached your data is gone.

Memcached is multi-threaded. Memcached implements a multi-threaded architecture by utilizing multiple cores. Therefore, for storing larger datasets, Memcached can perform better than Redis.

Another benefit of Memcached's multi-threaded architecture is its high scalability, achieved by utilizing multiple computational resources. You can handle more operations by scaling up compute capacity.

## Reference

[1] <https://people.cs.uchicago.edu/~junchenj/34702/slides/34702-MemCache.pdf>

[2] <https://www.baeldung.com/memcached-vs-redis>
