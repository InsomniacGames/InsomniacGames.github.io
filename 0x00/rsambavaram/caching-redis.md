As was pointed out in [earlier topic on Memcached](caching-memcached.html), one the key drawbacks of Memcached is its inability to support persistence (forcing architectures to rely on a slow DB fall over.) The advantage and disadvantage of Memcached is its simplicity and it doesn’t support any advanced data types like lists, sets and hashes. These disadvantages bring in a void especially for certain use cases where performance is key (where a slow DB just doesn’t cut it) and we where want atomic operations with frequent changes like counters and leaderboards among other cases.

Redis (open sourced, in memory DB/cache with persistence) steps in to fill that void (for write speed and latency) and does so with high performance where quite a few data structures are stored entirely in-memory cache without need for DB write through. Twitter, Uber, SnapChat, flickr are among the long list of companies that rely on Redis. In games (MMOGs from Blizzard, Zinga) it is used for leaderboards, session management and player profile among other things.

Redis uses memory as main storage and disk only for persistence and has lot of utility data structures:

* Strings (up to 512MB for each string). It is binary safe, so one can store html, json, images.
* Lists (supports push and pop from head or tail) great for message queue and timelines.
* Hashes (data structures for object representation supporting operations on individual fields.)
* Sets
* Sorted sets (set with value and score and sorted by the score), can retrieve FIFO or LIFO, and supports range queries.
* Bitmaps (for efficient large bit field masks especially to support memberships efficiently)
* Atomic operations (supports transactions)
* Single threaded 

The advent of Redis is attributed to common features found in websites like counters, user tracking, leaderboards, where performance is key. Redis is written in C and also uses SIMD. It can support 3.5TB with 20 million reads/sec and 4.5 mil writes/sec. And can support up to 15 shards per cluster.

Redis typically achieves persistence by starting from a full snapshot and using Append-To-File to write the changes to the disc.

Redis keys are binary safe, so one can use any binary sequence (even images if one wishes.)

Maximum allowed key size is 512MB.

It supports easy to setup master-slave asynchronous replication, with very fast non-blocking synchronization, auto-reconnection with partial resync on net split. (typically used to direct read traffic to slaves for scalability and multi region support.)

Redis supports Lists using linked lists. But for small (configurable global parameter) lists it uses a structure called ZipList where it is doubly linked list without pointers using sequential storage of data. Each element has a header which stores offset to previous and next.
