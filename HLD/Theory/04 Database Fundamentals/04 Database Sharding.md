Eventually, some databases outgrow one machine.

You can add indexes, tune queries, add read replicas, and buy a larger server. Those are usually the right first moves. But if one primary database can no longer handle the writes, storage, or hot data in memory, you may need to split the data across multiple machines.

That is sharding.

Sharding means splitting rows across multiple database nodes. Each node stores only part of the data. Together, the shards still represent one complete dataset, but no single database server owns all the rows.




After sharding

Application

Shard Router

Shard 1

Shard 2

Shard 3

Before sharding

Application

Single Primary
at capacity

Sharding can scale writes and storage, but it is one of the most expensive database scaling decisions you can make. It spreads data and load across machines, but it also makes routing, transactions, joins, migrations, and rebalancing harder.

Use it when the bottleneck is real and simpler techniques are no longer enough.

This chapter explains how sharding works and what extra work it adds.


1. What is Database Sharding?
Database sharding splits a dataset into smaller pieces called shards.

Each shard stores part of the data and usually runs on a separate database server or cluster.




users dataset
(30M rows)

Shard 1
user_id 1 to 10M

Shard 2
user_id 10M to 20M

Shard 3
user_id 20M to 30M

For example, users might be split by user_id:

Shard	Users
Shard 1	user_id 1 to 10M
Shard 2	user_id 10M to 20M
Shard 3	user_id 20M to 30M
When the application needs user 18M, it routes the query to Shard 2.

The column used to choose the shard is the shard key. Picking the shard key is the most important design decision in sharding, because it decides where rows live and which queries stay simple.


2. Why Shard?
Sharding helps when one primary database cannot handle the workload.

Common reasons:

Storage: the dataset no longer fits comfortably on one machine.
Write volume: one primary cannot process all writes.
Hot data size: the frequently used data no longer fits in memory on one node.
Customer isolation: large customers need their own capacity.
Regional placement: data needs to live near users or inside legal boundaries.
Problem	How Sharding Helps
One node runs out of disk	Data is split across multiple nodes
Writes overload one primary	Writes are distributed by shard key
Hot data set is too large	Each shard caches a smaller part
Large tenants affect everyone	Tenants can be isolated onto separate shards
Data-location rules matter	Shards can be placed by region
Sharding is usually not the first tool for scaling reads. Read replicas and caching often reduce read pressure with less complexity. Sharding is most valuable when you need to scale writes, storage, or separation between customers or regions.


3. How Sharding Works
A sharded system needs three things:

A shard key.
A mapping from shard key to shard.
A router that sends each request to the right shard.



Application

Shard Router

Shard 1

Shard 2

Shard 3

The router may live in application code, a data-access library, a proxy, or the database system itself.

For a request like:

Sql







123
the router can use user_id = 123 to find the correct shard.

If the query does not include the shard key, routing becomes harder. The system may need to ask every shard and merge the results. That is called a scatter-gather query. It is one of the major costs of sharding.


4. Choosing a Shard Key
The shard key decides where data lives.

A good shard key spreads both data and traffic evenly across shards so no single shard becomes overloaded. It also matches the queries the application runs most often. When possible, it keeps related data together so common operations stay on one shard. It should leave room to grow as the dataset changes over time.

This is harder than it sounds, and it is where many sharding projects succeed or fail.

Good Shard Key: user_id
For many consumer applications, user_id is a reasonable shard key.

Most user-specific queries include the user ID:

Sql







123
This routes cleanly to one shard.

Risky Shard Key: created_at
Sharding by timestamp can create hot shards.

If all new writes go to the shard for "today" or "this month," one shard receives most writes while older shards sit idle.

Time-based sharding can work for append-heavy logs or archive systems, but it needs careful design around hot ranges and old data cleanup.

Risky Shard Key: Fields With Few Values
Fields like country, status, or plan_type usually have too few possible values.

If you shard by country, one large country may dominate the system. If you shard by status, almost all active users may live on one shard.

Shard keys with only a few values often produce uneven data and uneven traffic.


5. Sharding Strategies
There are several ways to map keys to shards.

Sharding, Routing by Shard Key
Watch a router send each write to a shard, and see how the strategy shapes the load.

Range
Hash
Directory

Write stream
690
2737
4954
5002
5955
7714
7746
8055
8374
9704
Router
range
routes by shard key
avg 2.5
shard-0
(0)
0-2,499
shard-1
(0)
2,500-4,999
shard-2
(0)
5,000-7,499
shard-3
(0)
7,500-9,999
Load
0
0
0
0
Idle, press Play to route writes



0 / 30
1x

Hash-Based Sharding
Hash-based sharding applies a hash function to the shard key.




0

1

2

3

user_id = 12345

hash(user_id)

% 4

Shard 0

Shard 1

Shard 2

Shard 3

Example:








1
Hashing usually spreads data more evenly than ranges.

Rebalancing is the hard part. If you change number_of_shards, many keys may move to different shards. Production systems often use virtual shards or consistent hashing, which are techniques for moving less data when capacity changes.

Range-Based Sharding
Range-based sharding maps ranges of key values to shards.




1 to 10M

10M to 20M

20M to 30M

user_id = 15M

Range Lookup

Shard 1

Shard 2

Shard 3

Example:

Shard	Key Range
Shard 1	user_id 1 to 10M
Shard 2	user_id 10M to 20M
Shard 3	user_id 20M to 30M
Range sharding is easy to understand and supports range queries well.

Hot ranges are the trade-off. New records often land in the highest range, and time-ordered keys make that worse. You may need to split ranges ahead of time, split busy ranges while the system runs, or choose a different shard key.

Directory-Based Sharding
Directory-based sharding uses a lookup table or small service to map keys to shards.




tenant = acme

Directory
acme → 7
globex → 2
initech → 11

Shard 7

Shard 2

Shard 11

Example:

Tenant	Shard
acme	Shard 7
globex	Shard 2
initech	Shard 11
This is flexible. You can move one tenant to another shard by updating the directory.

The directory itself becomes critical. If it is down or wrong, routing breaks. It must be highly available, cached carefully, and updated safely during migrations.

Geo-Based Sharding
Geo-based sharding places data by region. A typical setup puts US users in US shards, EU users in EU shards, and India users in India shards.

This can reduce latency and help meet data-location rules. Global queries become harder, and users or organizations that move regions need special handling.


6. Cross-Shard Queries
The best sharded queries include the shard key.

This is good:

Sql







123
The router sends it to one shard.

This is harder:

Sql







12345
If status is not the shard key, the system may need to ask every shard, merge the results, sort them across all shards, and return the top 100.




Global Query

Shard 1

Shard 2

Shard 3

Merge / Sort

Result

Scatter-gather queries are slower, more expensive, and harder to make reliable. They also get worse as the number of shards grows.

You can avoid most of them by designing APIs around the shard key and keeping related data on the same shard. The goal is for common requests to hit one shard, not all of them.

When a query truly needs a global view, serve it from a separate system instead of asking every shard on every request. Maintain read models for those query patterns, use search indexes for global search, and push global reporting to an analytics system.


7. Joins and Transactions Across Shards
Sharding changes how you design relationships between tables.

Joining two tables on the same shard can be fine. Joining data across shards is expensive because the database cannot do a normal local join.

For example, if orders and order_items are both sharded by user_id, user-specific order queries stay local. This often means storing user_id on order_items too, so both tables use the same shard key.

If orders are sharded by user_id but products are sharded by product_id, joining orders to products may require cross-shard coordination or a copied version of product data.

Transactions have similar issues.

A transaction inside one shard is a normal database transaction. A transaction across multiple shards needs either a distributed transaction, where several databases coordinate one commit, or application code that can undo or repair partial work. That adds latency, new failure cases, and more work to operate.

Good sharded systems try to keep the most important transactions on one shard.


8. Rebalancing and Resharding
Shards do not stay balanced forever.

One shard may grow faster. One tenant may become huge. One range may become hot. Hardware changes. Traffic changes.

Rebalancing means moving data so load is spread more evenly.

This is difficult because the system has to copy data to the new shard and keep the old and new locations in sync while the move is happening.

During that window, it must route reads and writes correctly even though the data may be in two places. It also has to verify the copy, move traffic safely, and clean up the old data only when the team is confident.

Simple modulo hashing makes rebalancing painful because changing the number of shards can move a large fraction of keys. Several techniques reduce that pain.

Virtual shards or buckets and consistent hashing both limit how much data moves when capacity changes. A directory mapping lets you move customers one at a time. Splitting hot ranges relieves pressure without touching the rest of the system. The most important step is building migration tooling before you urgently need it.


9. Hot Shards
A hot shard receives more traffic or stores more data than the others. Common causes include a bad shard key, a large tenant, a celebrity user, a time-based hot range, a popular product or event, and uneven regional traffic.

Hot shards are dangerous because the whole system can become limited by one overloaded shard.

To cool a hot shard, move large tenants onto dedicated shards. For very hot entities, add a second split key so their load spreads across more nodes.

Caching hot reads and buffering bursty writes can take pressure off the shard without changing the data layout. Splitting hot ranges helps when a range-based key concentrates writes. If one entity keeps dominating traffic no matter what, revisit the data model itself.

Even with good hashing, traffic can be uneven because users are not equally active.


10. When Not to Shard
Sharding is expensive. Avoid it if a simpler option solves the problem.

Start by adding or fixing indexes and optimizing slow queries. One bad query plan can explain a surprising amount of database pain.

If reads are the bottleneck, add caching for hot reads and use read replicas for read-heavy workloads.

You can shrink the hot data set by archiving cold data and splitting large tables inside one database. Scaling up the primary is still worth considering when the hardware has room to grow. When expensive reads remain, store precomputed read models before reaching for sharding.

Do not shard because it sounds scalable. Shard because one database can no longer meet a measured requirement and the workload has a shard key that keeps important operations local.


11. Practical Rules of Thumb
Use these guidelines when designing a sharded system:

Pick the shard key from real query patterns, not just data distribution.
Keep the most important reads and writes single-shard.
Avoid shard keys with only a few possible values.
Watch for hot tenants, hot users, and hot time ranges.
Serve global queries from separate read models when possible.
Avoid cross-shard transactions on critical paths.
Plan rebalancing before the first emergency.
Track data size, queries per second, latency, and error rate per shard.
Keep shard routing simple and easy to monitor.
Treat shard-map changes like production migrations.

Summary
Sharding splits data across multiple database nodes so the system can scale beyond one machine's storage, write capacity, or separation limits.

The central design choice is the shard key. A good shard key spreads data and traffic while keeping common operations on one shard. A bad shard key creates hot shards, scatter-gather queries, and painful migrations.

Sharding is powerful, but it is not a first-line optimization. It makes joins, transactions, routing, rebalancing, and operations harder. Use it only after simpler techniques are no longer enough and the query patterns justify the extra complexity.