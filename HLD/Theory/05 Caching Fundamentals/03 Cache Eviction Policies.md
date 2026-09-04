Caching is a technique to make applications lightning fast, reduce database load, and improve user experience.

But, cache memory is limited - you can’t store everything.

So, how do you decide which items to keep and which ones to evict when space runs out?

This is where cache eviction strategies come into play. They determine which items are removed to make room for new ones.

In this article, we’ll dive into Top 7 Cache Eviction Strategies explaining what they are, how they work, their pros and cons.

If you’re enjoying this newsletter and want to get even more value, consider becoming a paid subscriber.

As a paid subscriber, you'll unlock all premium articles and gain full access to all premium courses on algomaster.io.

Unlock Full Access

1. Least Recently Used (LRU)
LRU evicts the item that hasn’t been used for the longest time.

The idea is simple: if you haven’t accessed an item in a while, it’s less likely to be accessed again soon.




Visualized using Multiplayer
How it Works
Access Tracking: LRU keeps track of when each item in the cache was last accessed. This can be done using various data structures, such as a doubly linked list or a combination of a hash map and a queue.

Cache Hit (Item Found in Cache): When an item is accessed, it is moved to the most recently used position in the tracking data structure (e.g., moving it to the front of a list).

Cache Miss (Item Not Found in Cache):

If the item isn’t in the cache and the cache has free space, it is added directly.

If the cache is full, the least recently used item is evicted to make space for the new item.

Eviction: The item that has been accessed least recently (tracked at the beginning of the list) is removed from the cache.

Consider a cache with a capacity of 3:

Initial State: Empty cache.

Add A → Cache: [A]

Add B → Cache: [A, B]

Add C → Cache: [A, B, C]

Access A → Cache: [B, C, A] (A becomes recently used)

Add D → Cache: [C, A, D] (B is evicted as it's the “least recently used”)

Pros:
Intuitive: Easy to understand and widely adopted.

Efficient: Keeps frequently accessed items in the cache.

Optimized for Real-World Usage: Matches many access patterns, such as web browsing and API calls.

Cons:
Metadata Overhead: Tracking usage order can consume additional memory.

Performance Cost: For large caches, maintaining the access order may introduce computational overhead.

Not Adaptive: Assumes past access patterns will predict future usage, which may not always hold true.

2. Least Frequently Used (LFU)
LFU evicts the item with the lowest access frequency. It assumes that items accessed less frequently in the past are less likely to be accessed in the future.

Unlike LRU, which focuses on recency, LFU emphasizes frequency of access.




Visualized using Multiplayer
How it Works
Track Access Frequency: LFU maintains a frequency count for each item in the cache, incrementing the count each time the item is accessed.

Cache Hit (Item Found in Cache): When an item is accessed, its frequency count is increased.

Cache Miss (Item Not Found in Cache):

If the cache has available space, the new item is added with an initial frequency count of 1.

If the cache is full, the item with the lowest frequency is evicted to make room for the new item. If multiple items share the same lowest frequency, a secondary strategy (like LRU or FIFO) resolves ties.

Eviction: Remove the item with the smallest frequency count.

Consider a cache with a capacity of 3:

Initial State: Empty cache.

Add A → Cache: [A (freq=1)]

Add B → Cache: [A (freq=1), B (freq=1)]

Add C → Cache: [A (freq=1), B (freq=1), C (freq=1)]

Access A → Cache: [A (freq=2), B (freq=1), C (freq=1)]

Add D → Cache: [A (freq=2), C (freq=1), D (freq=1)] (B is evicted as it has the lowest frequency).

Access C → Cache: [A (freq=2), C (freq=2), D (freq=1)]

Pros:
Efficient for Predictable Patterns: Retains frequently accessed data, which is often more relevant.

Highly Effective for Popular Data: Works well in scenarios with clear "hot" items.

Cons:
High Overhead: Requires additional memory to track frequency counts.

Slower Updates: Tracking and updating frequency can slow down operations.

Not Adaptive: May keep items that were frequently accessed in the past but are no longer relevant.

3. First In, First Out (FIFO)
FIFO evicts the item that was added first, regardless of how often it’s accessed.

FIFO operates under the assumption that items added earliest are least likely to be needed as the cache fills up.




Visualized using Multiplayer
How It Works
Item Insertion: When an item is added to the cache, it is placed at the end of the queue.

Cache Hit (Item Found in Cache): No changes are made to the order of items. FIFO does not prioritize recently accessed items.

Cache Miss (Item Not Found in Cache):

If there is space in the cache, the new item is added to the end of the queue.

If the cache is full, the item at the front of the queue (the oldest item) is evicted to make space for the new item.

Eviction: The oldest item, which has been in the cache the longest, is removed to make room for the new item.

Let’s assume a cache with a capacity of 3:

Add A → Cache: [A]

Add B → Cache: [A, B]

Add C → Cache: [A, B, C]

Add D → Cache: [B, C, D] (A is evicted because it was added first).

Access B → Cache: [B, C, D] (Order remains unchanged).

Add E → Cache: [C, D, E] (B is evicted because it was the oldest remaining item).

Pros:
Simple to Implement: FIFO is straightforward and requires minimal logic.

Low Overhead: No need to track additional metadata like access frequency or recency.

Deterministic Behavior: Eviction follows a predictable order.

Cons:
Ignores Access Patterns: Items still in frequent use can be evicted, reducing cache efficiency.

Suboptimal for Many Use Cases: FIFO is rarely ideal in modern systems where recency and frequency matter.

May Waste Cache Space: If old but frequently used items are evicted, the cache loses its utility.

Share

4. Random Replacement (RR)
RR cache eviction strategy is the simplest of all: when the cache is full, it evicts a random item to make space for a new one.

It doesn't track recency, frequency, or insertion order, making it a lightweight approach with minimal computational overhead.




Visualized using Multiplayer
This simplicity can sometimes be surprisingly effective, especially in systems with unpredictable or highly dynamic access patterns.

How It Works
Item Insertion: When an item is added to the cache and there is space, it is stored directly.

Cache Hit: If the requested item exists in the cache, it is served, and no changes are made to the cache.

Cache Miss: If the item is not in the cache and the cache is full, a random item is removed.

Eviction: The randomly selected item is removed, and the new item is added to the cache.

Let’s assume a cache with a capacity of 3:

Add A → Cache: [A]

Add B → Cache: [A, B]

Add C → Cache: [A, B, C]

Add D → Cache: [B, C, D] (A is randomly evicted).

Add E → Cache: [C, E, D] (B is randomly evicted).

Pros:
Simple to Implement: No need for metadata like access frequency or recency.

Low Overhead: Computational and memory requirements are minimal.

Fair for Unpredictable Access Patterns: Avoids bias toward recency or frequency, which can be useful in some scenarios.

Cons:
Unpredictable Eviction: A frequently used item might be evicted, reducing cache efficiency.

Inefficient for Stable Access Patterns: Doesn’t adapt well when certain items are consistently accessed.

High Risk of Poor Cache Hit Rates: Random eviction often leads to suboptimal retention of important items.

5. Most Recently Used (MRU)
MRU is the opposite of Least Recently Used (LRU). In MRU, the item that was accessed most recently is the first to be evicted when the cache is full.

The idea behind MRU is that the most recently accessed item is likely to be a temporary need and won’t be accessed again soon, so evicting it frees up space for potentially more valuable data.




Visualized using Multiplayer
How It Works
Item Insertion: When a new item is added to the cache, it is marked as the most recently used.

Cache Hit (Item Found in Cache): When an item is accessed, it is marked as the most recently used.

Cache Miss (Item Not Found in Cache):

If the cache has available space, the new item is added directly.

If the cache is full, the most recently used item is evicted to make room for the new item.

Eviction: The item that was accessed or added most recently is removed.

Let’s assume a cache with a capacity of 3:

Add A → Cache: [A]

Add B → Cache: [A, B]

Add C → Cache: [A, B, C]

Access C → Cache: [A, B, C] (C is marked as the most recently used).

Add D → Cache: [A, B, D] (C is evicted as it was the most recently used).

Access B → Cache: [A, B, D] (B becomes the most recently used).

Add E → Cache: [A, D, E] (B is evicted as it was the most recently used).

Pros:
Effective in Specific Scenarios: Retains older data, which might be more valuable in certain workloads.

Simple Implementation: Requires minimal metadata.

Cons:
Suboptimal for Most Use Cases: MRU assumes recent data is less valuable, which is often untrue for many applications.

Poor Hit Rate in Predictable Patterns: Fails in scenarios where recently accessed data is more likely to be reused.

Rarely Used in Practice: Limited applicability compared to other strategies like LRU or LFU.

6. Time to Live (TTL)
TTL is a cache eviction strategy where each cached item is assigned a fixed lifespan. Once an item’s lifespan expires, it is automatically removed from the cache, regardless of access patterns or frequency.

This ensures that cached data remains fresh and prevents stale data from lingering in the cache indefinitely.




Visualized using Multiplayer
How It Works
Item Insertion: When an item is added to the cache, a TTL value (e.g., 10 seconds) is assigned to it. The expiration time is usually calculated as current time + TTL.

Cache Access (Hit or Miss): When an item is accessed, the cache checks its expiration time:

If the item is expired, it is removed from the cache, and a cache miss is recorded.

If the item is valid, it is served as a cache hit.

Eviction: Expired items are automatically removed either during periodic cleanup or on access.

Let’s assume a cache with a TTL of 5 seconds:

Add A with TTL = 5s → Cache: [A (expires in 5s)]

Add B with TTL = 10s → Cache: [A (5s), B (10s)]

After 6 seconds → Cache: [B (expires in 4s)] (A is evicted because its TTL expired).

Add C with TTL = 5s → Cache: [B (4s), C (5s)]

If an item is accessed after its TTL expires, it results in a cache miss.

TTL is often implemented in caching systems like Redis or Memcached, where you can specify expiration times for each key.

Pros:
Ensures Freshness: Automatically removes stale data, ensuring only fresh items remain in the cache.

Simple to Configure: TTL values are easy to assign during cache insertion.

Low Overhead: No need to track usage patterns or access frequency.

Prevents Memory Leaks: Stale data is cleared out systematically, avoiding cache bloat.

Cons:
Fixed Lifespan: Items may be evicted prematurely even if they are frequently accessed.

Wasteful Eviction: Items that haven’t expired but are still irrelevant occupy cache space.

Limited Flexibility: TTL doesn’t adapt to dynamic workloads or usage patterns.

7. Two-Tiered Caching
Two-Tiered Caching combines two layers of cache—usually a local cache (in-memory) and a remote cache (distributed or shared).




Visualized using Multiplayer
The local cache serves as the first layer (hot cache), providing ultra-fast access to frequently used data, while the remote cache acts as the second layer (cold cache) for items not found in the local cache but still needed relatively quickly.

How It Works
Local Cache (First Tier):

Resides on the same server as the application, often in memory (e.g., HashMap, LRUCache in the application)..

Provides ultra-fast access to frequently accessed data, reducing latency and server load.

Examples: In-memory data structures like HashMap or frameworks like Guava Cache.

Remote Cache (Second Tier):

Shared across multiple servers in the system. Slightly slower due to network overhead but offers larger storage and shared consistency.

Used to store data that is not in the local cache but is still frequently needed.

Examples: Distributed cache systems like Redis or Memcached.

Workflow:

A client request checks the local cache first.

If the data is not found (cache miss), it queries the remote cache.

If the data is still not found (another cache miss), it retrieves the data from the primary data source (e.g., a database), stores it in both the local and remote caches, and returns it to the client.

Pros:
Ultra-Fast Access: Local cache provides near-instantaneous response times for frequent requests.

Scalable Storage: Remote cache adds scalability and allows data sharing across multiple servers.

Reduces Database Load: Two-tiered caching significantly minimizes calls to the backend database.

Fault Tolerance: If the local cache fails, the remote cache acts as a fallback.

Cons:
Complexity: Managing two caches introduces more overhead, including synchronization and consistency issues.

Stale Data: Inconsistent updates between tiers may lead to serving stale data.

Increased Latency for Remote Cache Hits: Accessing the second-tier remote cache is slower than the local cache.