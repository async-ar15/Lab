Once you add a cache, you still have an important design choice to make: how should reads and writes move through it?

Should the application load missing data into the cache, or should the cache do that work? When data changes, should the write go to the cache, the database, or both?

A caching strategy answers those questions. Each strategy makes a different trade-off between speed, freshness, complexity, and failure risk.

This chapter compares the five strategies you will see most often in system design: cache-aside, read-through, write-through, write-around, and write-back. For each one, we will look at how reads work, how writes work, what can go wrong, and when the strategy is a good fit.


1. The Core Trade-offs
Two decisions shape every caching strategy:

On a cache miss, who fetches the data from the database?
On a write, what gets updated first, and when do we tell the caller the write is done?
Those answers decide how fast the system feels, how fresh the cached data is, and what happens when the cache or database has a problem.


2. Cache-Aside (Lazy Loading)
In cache-aside, the application manages the cache directly.

For reads, the application checks the cache first. If the data is missing, it reads from the database, stores a copy in the cache, and returns the result. For writes, the application updates the database and usually deletes the old cache entry so it can be loaded fresh next time.

Cache-Aside, Lazy Loading
The application checks the cache, loads from the database on a miss, then fills the cache.

Read (Hit)
Read (Miss)
Write

Application
Your service
Cache
Redis
Database
Postgres
Cache
user:42
=
Alice
fresh
Database
user:42
=
Alice
durable
Idle, press Play to begin



0 / 2
1x

The cache is treated as a speed boost, not as required storage. If the cache is down, the application can still read from the database, but requests will be slower and the database will work harder.

Cache-aside works well with simple key-value stores such as Redis or Memcached. It is flexible because the application can choose different TTLs and rules for different kinds of data.

The cost is extra application code. The application has to handle cache reads, cache fills, and cache deletes. The first read for a key is still slow because it has to hit the database, and stale data is possible if the application forgets to delete an old cache entry.

Best for: Read-heavy workloads where the application team is comfortable owning the cache logic. Common examples include product catalogs, user profiles, and configuration data.


3. Read-Through
In a read-through cache, the application asks the cache for data and lets the cache handle misses.

If the value is missing, the cache layer loads it from the database, stores it, and returns it to the application.

Read-Through, Cache Loads
The application only talks to the cache, which loads from the database itself on a miss.

Read (Hit)
Read (Miss)

Application
Your service
Cache
Redis
Database
Postgres
Cache
user:42
=
Alice
fresh
Database
user:42
=
Alice
durable
Idle, press Play to begin



0 / 2
1x

In other words, the "what do we do on a miss?" logic lives in the cache layer, not in every application service.

Read-through is useful when many services need the same read behavior. It keeps the loading logic in one place, so application code can stay simpler.

The trade-off is that the cache layer now needs to know how to load data from the database. Cold reads still pay the database cost, and if the loader breaks, reads through the cache can break too.

Best for: Systems where many services share the same read pattern and benefit from one shared loading path. Local cache libraries such as Caffeine make this pattern easy.


4. Write-Through
In write-through, the application writes to the cache layer, and the cache layer writes to the database before reporting success.

Write-Through, Synchronous
The application writes to the cache, which synchronously writes to the database before it acks.

Write
Read

Application
Your service
Cache
Redis
Database
Postgres
Cache
user:42
=
Alice
fresh
Database
user:42
=
Alice
durable
Idle, press Play to begin



0 / 4
1x

This means a read right after a write usually finds the new value in the cache.

Write-through keeps the cache and database closely aligned. It also removes some manual cache-delete code from the application. The cost is slower writes, because each write must update both the cache path and the database.

There is one important rule: all writers must use the same cache path. If one service writes directly to the database, the cache may keep serving the old value.

Best for: Workloads where reads commonly follow writes, slower writes are acceptable, and all writers can be routed through the cache layer.


5. Write-Around
In write-around, writes go directly to the database and skip the cache.

The cache is updated later only if someone reads the data and the read path fills the cache, usually using cache-aside.

Write-Around, Skip Cache
Writes go straight to the database, so the cache stays cold until a later read fills it.

Write
Read After Write

Application
Your service
Cache
Redis
Database
Postgres
Cache
user:42
=
Alice
fresh
Database
user:42
=
Alice
durable
Idle, press Play to begin



0 / 2
1x

If that key was already cached before the write, the cached value can become stale until it expires or gets deleted.

Write-around is good at keeping rarely-read data out of the cache. It also keeps writes simple because the write only touches the database.

The downside is that the first read after a write is often slow. Existing cached values can also be wrong unless you delete them when the write happens.

Best for: Write-heavy workloads where written data is rarely read soon after, such as bulk imports, event ingestion, and large append-only logs.


6. Write-Back (Write-Behind)
In write-back, the application writes to the cache, and the cache reports success right away. The database write happens later in the background.

Write-Back, Async Flush
The cache acks writes instantly and flushes dirty entries to the database later in the background.

Write
Flush

Application
Your service
Cache
Redis
Database
Postgres
Cache
user:42
=
Alice
fresh
Database
user:42
=
Alice
durable
Idle, press Play to begin



0 / 2
1x

This is fast, but it comes with a serious risk: the caller may think the write is saved even though it has not reached the database yet. If the cache fails before the background write finishes, that data can be lost.

Write-back gives very low write latency. Background workers can also batch many small writes together, which reduces database load during write spikes.

The downsides are significant. To use write-back safely, you need a reliable place to store writes that have not reached the database yet, such as a write-ahead log or replicated queue. You also need to handle ordering, retries, duplicate writes, and recovery after failures.

Some systems use Redis AOF, replicated queues, or a separate log for this buffer. Redis AOF helps, but the safety depends on how often data is forced to disk. With the common everysec setting, you can still lose about one second of writes. With always, writes are safer but much slower, which removes much of the reason to use write-back.

Best for: Low-risk, high-volume data where the database write can happen later, such as counters, view counts, metrics, and certain session state. Not appropriate for payments, audit logs, user-generated content, or any write where loss is unacceptable.


7. Comparison
Each strategy answers three questions differently: who handles cache misses, when writes reach the database, and how stale the cache can become.

This table puts the main differences side by side.

Strategy	Read Miss Handling	Write Path	Success Returned	Freshness	When to Use
Cache-Aside	Application loads from DB	App writes DB, then deletes cache entry	After DB write and cache delete	Eventually fresh through TTL or delete	Read-heavy, infrequent updates
Read-Through	Cache layer loads from DB	Not a write strategy	Not applicable	Eventually fresh	Shared read paths across services
Write-Through	Cache layer loads from DB if paired with read-through	App writes through cache; cache writes DB	After DB write succeeds	Fresh if all writers use the cache path	Fresh reads after writes
Write-Around	Application loads from DB	App writes DB directly; cache is skipped	After DB write succeeds	Cached entries can be stale	Write-heavy, recent writes rarely read
Write-Back	Application loads from DB	App writes cache; DB write happens later	After cache write	Risk of loss unless pending writes are safely stored	Many writes where some loss is tolerable

8. Choosing a Strategy
Pick a strategy based on the workload and the cost of serving stale or losing data.

Workload Shape	Strategy to Consider
Reads dominate, writes are occasional	Cache-Aside or Read-Through
Reads frequently follow writes	Write-Through
Many services share the same read logic	Read-Through
Writes are frequent, recent writes rarely read	Write-Around
Writes must feel very fast and the data is low-risk	Write-Back, with a safe queue for pending writes
Stale or lost data is unacceptable	Read directly from the database; cache only data that is safe to be a little old
Many production systems combine strategies. For example, cache-aside reads with write-around writes are common when newly written data is rarely read right away. Write-back is usually reserved for narrow, high-volume, low-risk paths rather than used across the whole system.


Summary
There is no single best caching strategy. Each one is a different answer to the same question: how much speed, complexity, and risk are acceptable for this feature?

Cache-aside is the most flexible default, with cache logic living in the application. Read-through puts the read-miss logic in the cache layer. Write-through keeps cache and database closely aligned, but writes are slower. Write-around keeps rarely-read writes out of the cache.

Write-back makes writes feel very fast, but only fits data where delayed database writes and possible loss are acceptable.

Choose based on the read/write ratio, how much stale or lost data the feature can tolerate, and where you want the complexity to live.