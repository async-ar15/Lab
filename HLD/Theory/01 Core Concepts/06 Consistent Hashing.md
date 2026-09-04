Consistent Hashing
High Priority
13 min read
Updated May 27, 2026
AI Mock Interview
Practice this topic in a realistic system design interview


Listen to this chapter
Unlock Audio

Distributed systems often need a stable way to decide which node owns a key.

Examples:

Which cache node stores user:123?
Which storage shard owns order:987?
Which worker should process events for customer:42?
Which vector index partition should receive a document embedding?
A simple approach is:


That works while the node count stays fixed. It breaks badly when nodes are added or removed. If number_of_nodes changes from 5 to 6, most keys get a different result. For a cache, that means a large cache miss storm. For a storage system, it means a large data movement event. For a stateful worker fleet, it means many keys move to new owners at once.

Consistent hashing solves this by minimizing key movement when membership changes. When a node is added or removed, only the keys near that node move. Most keys keep the same owner.

Consistent hashing is used in distributed caches, Dynamo-style databases, storage systems, load balancers for stateful traffic, queue partitioning, and routing layers. The exact implementation varies, but the design goal is the same: stable ownership with controlled movement.


1. The Problem with Modulo Hashing
Modulo hashing maps a key to a node by taking the hash modulo the number of nodes.

Suppose we have 5 nodes:


For each key:





key

hash(key)

mod 5

S0

S1

S2

S3

S4

This gives deterministic routing. The same key goes to the same node as long as the node list and node count do not change.




user:1
hash 42

42 mod 5 is 2

S2

user:2
hash 37

37 mod 5 is 2

S2

user:3
hash 18

18 mod 5 is 3

S3

user:4
hash 91

91 mod 5 is 1

S1

Adding a Node
Now add S5.

The formula changes from:


to:





After: 6 nodes

user:1 hash 42
goes to S0

user:2 hash 37
goes to S1

user:3 hash 18
goes to S0

user:4 hash 91
still S1

Before: 5 nodes

user:1 hash 42
goes to S2

user:2 hash 37
goes to S2

user:3 hash 18
goes to S3

user:4 hash 91
goes to S1

That small formula change remaps most keys.

This is painful because the system does not just change a routing table. It also moves operational load:

Cache entries become cold on their new owners.
Storage shards need data migration.
Stateful workers lose locality.
In-flight requests may hit a different owner than earlier requests.
Downstream databases can see a sudden read spike after cache misses.
Removing a Node
If S4 fails, the formula changes from:


to:





After: 4 nodes

user:1 hash 42
still S2

user:2 hash 37
goes to S1

user:3 hash 18
goes to S2

user:4 hash 91
goes to S3

Before: 5 nodes

user:1 hash 42
goes to S2

user:2 hash 37
goes to S2

user:3 hash 18
goes to S3

user:4 hash 91
goes to S1

Again, most keys move even though only one node changed.

Modulo hashing is fine for fixed partition counts. For example, Kafka-style partitioning often hashes keys into a stable number of partitions, then moves partitions between brokers separately. The problem appears when the hash target is the live node count.


2. How Consistent Hashing Works
Consistent Hashing, the Ring
Servers and keys hash onto a ring. Each key belongs to the first server clockwise.


clockwise
↻
0x0000 to 0xFFFF
S1
0x2AAB
S2
0x8000
S3
0xD555
Key distribution
S1
3 (33%)
S2
2 (22%)
S3
4 (44%)
Servers
3
Keys
9
Ring positions
3
3 servers and 9 keys placed on the ring. Each key belongs to the first server clockwise.
Add server
Remove server
Add key
Replicas
1
3

Fullscreen
Consistent hashing maps both nodes and keys into the same fixed hash space.

The hash space is usually shown as a ring:

Hash values run from 0 to a large maximum, such as 2^64 - 1.
After the maximum value, the ring wraps back to 0.
Each node is placed on the ring using hash(node_id).
Each key is placed on the ring using hash(key).
A key belongs to the first node found while moving clockwise from the key's position.



wraps clockwise

Hash Ring
0 to 2^64 minus 1
wraps clockwise

S0
pos 100

S1
pos 300

S2
pos 500

S3
pos 700

S4
pos 900

Mapping a Key
To route user:123:

Compute hash("user:123").
Locate that position on the ring.
Walk clockwise until the next node.
Route the key to that node.



wraps clockwise

next clockwise

next clockwise

wraps to

S0
pos 100

S1
pos 300

S2
pos 500

S3
pos 700

S4
pos 900

user:123
hash 250

order:987
hash 620

item:42
hash 950

If the key lands exactly on a node position, it belongs to that node. In practice, with a large hash space, exact collisions are rare. A real implementation still needs deterministic collision handling.

Adding a Node
When a new node joins, it claims the range between its predecessor and itself.

Suppose S5 is placed between S1 and S2.




wraps clockwise

S0
pos 100

S1
pos 300

S5 NEW
pos 400

S2
pos 500

S3
pos 700

S4
pos 900

Only keys in range (S1, S5]
move from S2 to S5

Only keys in the range (S1, S5] move from S2 to S5. Keys owned by other nodes stay where they are.

With N evenly balanced nodes, adding one node moves roughly 1 / (N + 1) of the keys. It does not remap the whole keyspace.

Removing a Node
When a node leaves, only its range moves to the next clockwise node.




now wraps directly to

keys move clockwise to

S0
pos 100

S1
pos 300

S2
pos 500

S3
pos 700

S4 REMOVED
pos 900

If S4 leaves, the keys previously assigned to S4 move to its successor. Other ranges are unchanged.

This is the central benefit: membership changes cause local movement, not global reshuffling.


3. Virtual Nodes
Basic consistent hashing places each physical node at one point on the ring. That is usually not enough.

With only one point per node, the ranges can be uneven. One node may own a large slice of the ring while another owns a small slice. If a node fails, its entire range moves to one successor, which can overload that successor.

Virtual nodes, often called vnodes, fix this by placing each physical node at many positions on the ring.




wraps clockwise

S1-A

S3-A

S2-A

S1-B

S2-B

S3-B

S1-C

S2-C

S3-C

Each physical node (S1, S2, S3) appears
at multiple positions on the ring

Instead of this:


use this:




wraps clockwise

pos 10
S1-0

pos 30
S3-0

pos 50
S2-0

pos 70
S1-1

pos 80
S2-1

pos 90
S3-1

pos 120
S1-2

pos 140
S3-2

pos 160
S2-2


Each virtual node maps back to a physical node.

Benefits:

Better distribution: many small ranges are easier to balance than a few large ranges.
Smoother failure behavior: when a physical node fails, its ranges are spread across several successors.
Weighted capacity: larger nodes can receive more virtual nodes than smaller nodes.
Incremental migration: operators can add or remove virtual nodes gradually instead of moving a huge range at once.
Virtual nodes are the difference between the clean classroom version of consistent hashing and a version you can operate in production.


4. Replication with Consistent Hashing
Consistent hashing decides the primary owner of a key. Many real systems also need replicas.

A common approach is:

Hash the key onto the ring.
Pick the first node clockwise as the primary.
Continue clockwise to pick the next distinct physical nodes as replicas.
For a replication factor of 3, a key may be stored on:


Production systems usually add placement constraints:

Do not place two replicas on the same physical node.
Avoid placing all replicas in the same rack or availability zone.
Prefer region-local reads when the consistency model allows it.
Rebalance when node capacity, disk usage, or failure domains change.
Consistent hashing is placement logic. It does not by itself provide replication, quorum reads, conflict resolution, or durability. Those are separate parts of the storage design.


5. Code Implementation
The implementation below uses:

A sorted list of ring positions.
A map from ring position to physical server.
Multiple virtual nodes per physical server.
Binary search to find the first position greater than or equal to the key hash.
For production code, use a fast, stable, well-distributed non-cryptographic hash such as MurmurHash, xxHash, or CityHash. The examples use SHA-256 because it is available in the standard libraries and produces stable output across runs.

Java

Run







12345678910111213141516171819202122232425262728293031
Lookup is O(log V), where V is the number of virtual nodes. Adding or removing a physical server is O(R log V), where R is the number of virtual nodes assigned to that server.


6. Operational Considerations
Consistent hashing is useful, but production systems still need operational guardrails.

Use a Stable Node Identity
The node identifier must not change accidentally. If the ring uses an IP address and the instance gets a new IP, the system may treat the same machine as a different node and move keys unnecessarily.

Prefer stable identifiers:

cache-a-17
shard-003
az1-rack4-node12
A persistent node UUID stored on disk
Use Enough Virtual Nodes
Too few virtual nodes produce uneven load. Too many increase memory usage and membership update cost.

The right number depends on fleet size, key distribution, node capacity differences, and how often membership changes. In many cache clients, tens to hundreds of virtual nodes per physical node is a common starting point. Storage systems may use a more deliberate token or partition assignment strategy.

Account for Uneven Key Popularity
Consistent hashing balances key ownership, not request volume.

If one key is extremely hot, the node that owns it can still overload. Common mitigations include:

Request coalescing
Local caching
Hot-key replication
Splitting large tenants or high-traffic keys
Application-level load shedding
Do Not Confuse Routing with Rebalancing
Changing the ring changes where keys should live. It does not move the data by itself.

A storage system needs background migration, repair, checksums, throttling, and progress tracking. A cache can often tolerate cold misses, but a database cannot simply forget the old owner.

Keep Membership Consistent Enough
Clients need a reasonably consistent view of the ring. If two clients use different membership lists, they may route the same key to different owners.

This is usually handled with:

A central configuration service
Gossip membership with versioning
A control plane that publishes ring snapshots
Client-side ring versions attached to requests
Consider Alternatives
Consistent hashing is not the only stable assignment technique.

Technique	Good Fit	Notes
Consistent hashing ring	Caches, storage shards, stateful routing	Familiar and supports vnodes
Rendezvous hashing	Selecting one or more owners from a node set	Simple, no ring structure, good for small to medium node sets
Jump consistent hash	Fast mapping from key to bucket number	Great when buckets are numbered and mostly append-only
Fixed partitions	Logs, queues, databases with partition movement	Decouples key hashing from physical node membership
Many modern systems use fixed logical partitions rather than hashing directly to physical nodes. Keys map to partitions, and partitions move between nodes through a control plane. This gives operators more control over rebalancing, placement, and failure recovery.


7. Where Consistent Hashing Works Well
Consistent hashing is a good fit when:

Keys need stable owners.
Nodes are added or removed over time.
Moving all keys on membership change would be expensive.
Approximate balance is acceptable with virtual nodes or weights.
The system can handle the operational work of migration or cache warming.
Common use cases:

Distributed caches
CDN cache routing
Sharded key-value stores
Dynamo-style storage systems
Stateful request routing
Worker assignment for keyed streams
Tenant-to-shard routing
Vector index shard routing
Feature store partition routing
It is less useful when requests are stateless and any node can serve any request. In that case, ordinary load balancing is usually simpler and more flexible.


Summary
Consistent hashing provides stable key ownership in a changing cluster.

Key takeaways:

Modulo hashing remaps too many keys when the live node count changes.
Consistent hashing uses a fixed hash space and maps both keys and nodes onto it.
A key belongs to the next node clockwise from the key's hash position.
Adding a node moves only the keys in that node's new range.
Removing a node moves only the keys owned by that node.
Virtual nodes improve balance and make failures less concentrated.
Consistent hashing is placement logic, not a full storage system. Replication, migration, durability, and conflict handling are separate concerns.
Modern systems often combine consistent hashing with control planes, fixed partitions, weights, and failure-domain-aware placement.
Use consistent hashing when stable ownership matters. Do not use it as a substitute for ordinary load balancing when every node can handle every request.