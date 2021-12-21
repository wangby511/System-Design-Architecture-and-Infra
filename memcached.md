# Memcached

Memcached is an in-memory key-value store for small chunks of arbitrary data (strings, objects) from results of database calls, API calls, or page rendering.

Memcached supports a key of only 250 bytes and values are capped at 1MB by default. Memcached only stores data as strings.

Memcached is, by design, very simple to set up and use. If you had an extremely simple application on only a few servers, and only required simple string interpretation for your application.

When you restart memcached your data is gone.

Memcached is multithreaded.
