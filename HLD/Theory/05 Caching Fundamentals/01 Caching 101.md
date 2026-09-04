Reading from a database or calling another service takes time. Under heavy traffic, that cost adds up quickly.

A cache is a fast storage layer that keeps copies of frequently used data. When the application needs that data again, it can read from the cache instead of going all the way back to a slower system.

The idea is simple, but the payoff is large. Used well, caching can make requests faster, reduce database load, lower cost, and help the system survive traffic spikes.

The tricky part is that a cache stores a copy. If the real data changes, the cached copy can become old.

This chapter covers what a cache is, where caches live in a system, what makes data a good fit for caching, and the trade-offs that come with every cached copy.

Caching, TTL and Hit Rate
Request resources and watch the cache serve hits instantly, fetch misses from the database, and expire entries after their TTL.

Clear

👤
Client
Cache
Database
Hits: 0
Misses: 0
Hit Rate: 0%
Cache Contents (0 entries)
Empty
TTL:
5s
10s
20s
Request a resource:
user:1
user:2
product:1
product:2
config
Click a resource to request it. Cached items (marked with ⚡) return instantly. Items expire after TTL.

Fullscreen

1. The Cost of Hitting the Database
Consider a social media application. When a user opens their feed, the application must:

Authenticate the user (database query)
Fetch the user's profile (database query)
Retrieve the list of followed accounts (database query)
Fetch recent posts from followed accounts (many database queries)
Get like counts and comment counts for each post (more queries)
Retrieve profile pictures for all post authors (even more queries)
Without caching, a single feed load might trigger 50+ database queries. Multiply that by millions of users, and the database becomes the bottleneck.




Without Caching

50+ queries

User Request

Application

Database

With Caching

Hit: 45 queries

Miss: 5 queries

User Request

Application

Cache

Database

Example Performance Impact:

Metric	Without Cache	With Cache	Improvement
Response time	500ms	50ms	10x faster
Database queries/sec	500,000	50,000	10x reduction
Database CPU	95%	30%	Headroom for growth
Infrastructure cost	$50,000/month	$20,000/month	60% savings
The Shape of a Cache
At its core, a cache is usually a key-value store built for fast lookups. The application gives the cache a key, and the cache returns the value stored under that key.




Cache Structure

key: user:123 → value: {name: 'Alice', ...}

key: product:456 → value: {title: 'Widget', ...}

key: session:abc → value: {userId: 123, ...}

Key Parts
Keys are unique names for cached data. A good key is predictable: the same request should always produce the same key. It should also describe what it stores, and different data should not accidentally share the same key.








123456789
Values are the cached data itself. A value might be JSON, a string, binary data, an HTML fragment, or a precomputed result like a count or summary.

Metadata is extra information about the cached entry, such as when it was created, when it expires, how often it has been used, and how large it is.


2. Cache Layers
Caching happens at multiple levels in a system. Each layer has a different job:




User

Browser Cache

CDN Cache

Load Balancer

Application Cache

Distributed Cache
Redis/Memcached

Database
Buffer Pool

Browser Cache
The browser cache is the closest cache to the user. It stores static assets and sometimes API responses on the user's device. If the browser already has a valid copy, it may not need to make a network request at all.

Behavior is controlled through HTTP headers like Cache-Control and ETag, which makes browser caching a natural fit for images, scripts, stylesheets, and data that does not change often.








12
CDN Cache
Content Delivery Networks, or CDNs, cache content at edge locations around the world. Users can download content from a nearby edge instead of from your main servers.

CDNs work best for static content, media files, public pages, and other content that many users can share.

Application Cache
The application cache lives in memory inside the application process itself. It is extremely fast because the data is already inside the running application.

The trade-off is that each application instance has its own local cache. That can duplicate memory and create different cached values on different instances.

Distributed Cache
A distributed cache is a separate caching service shared by all application instances. Redis and Memcached are common choices.

Lookups are slower than local memory because they cross the network, but the cache is shared and can hold much more data. Distributed caches are commonly used for shared state, session storage, and database query results.




App Server 1

Redis Cluster

App Server 2

App Server 3

Database

Database Buffer Pool
Databases also keep their own internal cache. They store frequently accessed database pages in memory on the database server, so repeated reads do not always need to hit disk.

This layer is usually invisible to application code. The database manages it automatically.


3. Cache Hit and Miss
When the application asks the cache for data, two things can happen:

Cache Hit: The data exists in the cache. Return it immediately.

Cache Miss: The data is not in the cache. Fetch it from the database or another source, optionally store it in the cache, then return it.




Yes: Cache Hit

No: Cache Miss

Request for key X

Key X in cache?

Return cached value

Fetch from database

Store in cache

Return value

Cache Hit Ratio
The hit ratio is the percentage of cache lookups served from the cache:








1
Once you have that number, this table gives a rough sense of where it stands:

Hit Ratio	Interpretation
> 95%	Excellent. Cache is highly effective.
80-95%	Good. Normal for most applications.
50-80%	Moderate. May need tuning or different caching strategy.
< 50%	Poor. Cache may be undersized or data not cache-friendly.
A 90% hit ratio means 90% of requests avoid the database for that data. If your database can handle 10,000 queries per second for those reads, a 90% hit ratio can let the system handle roughly 100,000 read requests per second.


4. What to Cache
Not all data benefits equally from caching.

Caching pays off the most when the same data is read many times, when the data is expensive to compute, when the source is slow, or when the data changes rarely.

It is a poor fit when data changes constantly, when every request needs different data, when objects are too large, or when even a little stale data is unacceptable.

The 80/20 Rule
In most applications, 20% of the data serves 80% of the requests. Focus caching efforts on that hot 20%:








123456789
Caching that hot slice captures most of the benefit while keeping memory use small. Rarely requested keys usually do not repeat enough to be worth caching aggressively.


5. Cache Consistency
Cache consistency means keeping cached data reasonably aligned with the real data. When the database changes, the cache can become stale.




Stale Cache Problem

Updated, but
cache not

Database
price: $20

Cache
price: $15

User sees $15
Incorrect!

Consistency Approaches
There are a few common ways to keep cached data fresh enough:

Approach	How It Works	Trade-off
TTL-based	Data expires after a time period	Simple, but data may be stale until it expires
Invalidation	Remove or update cache when data changes	Fresher, but harder to implement correctly
Write-through	Update cache and database together	Fresher reads, but slower writes
Accept short staleness	Allow old data briefly	Fast, but only safe for data that can be a little old
The right approach depends on the feature. A product description being stale for 5 minutes might be fine. A bank account balance being stale for 5 seconds is not.


6. Caching Anti-Patterns
Caching has costs as well as benefits. The patterns below often cause more problems than they solve.

Cache Everything
Blindly caching everything fills memory with data that may never be read again. The cache becomes larger, more expensive, and often less useful.

Infinite TTL
Data that never expires can stay wrong forever. If one cache delete is missed, users may keep seeing old data until someone notices and fixes it.

Cache as Primary Storage
Treating the cache as the real data store is dangerous. Caches can evict data, restart, or fail. If the database was never updated, that data may be gone.








123456789101112
The Thundering Herd
When a popular cache entry expires, many requests can hit the database at the same time:




TTL expires

Cache Expired - Thundering Herd

OVERLOADED

Cache: empty

1000 requests/sec

Database

System Crash

Popular Item Cached

Cache: product:123

1000 requests/sec

Common fixes include per-key locking, early refresh before expiration, and background refresh.


7. Cache in Distributed Systems
In distributed systems, caching has a few extra problems because there are many servers and cache nodes.

Consistency Across Nodes
When multiple application servers or cache nodes store copies of the same data:




Cache Cluster

App Servers

Server 1

Server 2

Server 3

Cache Node 1

Cache Node 2

Cache Node 3

Cache entries may need to be deleted or refreshed in several places. Some nodes may see the new value before others.

Data Partitioning
Large caches split data across multiple nodes. A technique called consistent hashing helps avoid moving too many keys when nodes are added or removed:








123456789
Failure Handling
What happens when cache is unavailable?

Strategy	Behavior	Use Case
Fail open	Skip cache and hit database directly	Cache is optional
Fail closed	Return an error	Cache is required for correctness
Graceful fallback	Serve stale data if available	Availability matters more than perfect freshness
Picking the wrong mode can turn a cache outage into a full outage. Decide in advance whether the database can safely handle traffic without the cache.


8. Measuring Cache Performance
The key metrics to track are hit ratio, cache latency, memory usage, eviction rate, and miss latency.

Hit ratio tells you whether the cache is useful. Cache latency tells you whether cache lookups are fast. Memory usage and evictions tell you whether the cache is too small. Miss latency tells you how expensive it is when requests fall through to the database.

In latency metrics, p50 means the typical request, and p99 means one of the slowest requests users still regularly see.








12345678
A sudden drop in hit ratio or spike in evictions is a warning sign. Traffic may have changed, the cache may be too small, or keys may be getting deleted too aggressively.


Summary
Caching stores frequently accessed data in a faster layer to reduce latency and database load. Caches exist across the stack: browser, CDN, application, distributed cache, and database buffer pool.

Hit ratio is one of the most important metrics to watch. A 90% hit ratio can give roughly a 10x improvement for the reads being cached.

What gets cached matters more than how much gets cached. Read-heavy, expensive-to-compute, and stable data benefit the most. Write-heavy data and one-off requests usually benefit less.

Common anti-patterns include caching everything, setting infinite TTLs, treating the cache as the real data store, and ignoring stampedes when popular entries expire.

Understanding what caching is and why it matters sets the stage for the next question: how exactly should an application interact with the cache?

That brings us to caching patterns, starting with the most common one, the cache-aside pattern, where the application explicitly manages what goes in and out of the cache.