Large systems spend a surprising amount of time looking for things that are not there.

Before reading from disk, checking a cache, or calling another service, it is useful to ask a cheap question first: "Can this item possibly exist here?"

A Bloom filter answers that question with very little memory. It can say:

"No, this item is definitely not present."
"Maybe. This item might be present."
That makes Bloom filters useful as fast pre-checks. A database can skip a disk file that cannot contain a key. A cache can skip a lookup for a key that was never cached. A crawler can avoid re-processing a URL it has probably seen before.

This chapter explains how Bloom filters work, why they sometimes say "maybe" for missing items, how to size them, and where they are useful in real systems.




Key 1
Key 2
1
1
0
1
1
0
1
Bloom Filter
Set k bits per item, then test membership. A clear bit proves absence, all-set can be a false positive.

Add then check
False positive
Definite miss
0/48 bits set
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0
0



0 / 10

1x

Press Run to start

1. What Is a Bloom Filter?
A Bloom filter is a space-saving data structure for checking whether an item is in a set.

The important word is "probably."

It does not store the actual items. Instead, it stores a compact pattern of bits created from those items. Because of that, it can be very small, but its answers are not symmetric:

If it says no, the item is definitely absent.
If it says yes, the item is only probably present.
It stores:

a bit array of length m
k hash functions
The bit array starts full of 0s. Hash functions turn each item into positions inside that array.




Data
Hash 1
Hash 2
Hash 3
1
0
0
1
0
0
1
To add an item:

Hash the item k times.
Map each hash to a bit position.
Set those bits to 1.
To query an item:

Hash it the same k ways.
Check the same bit positions.
If any bit is 0, the item is definitely absent.
If all bits are 1, the item is probably present.
Bloom filters shine when a definite "no" lets the system skip expensive work.


2. Example: URL Deduplication
Suppose a web crawler wants to avoid fetching the same URL again and again.

Keeping every visited URL in an exact in-memory set can become expensive when the crawler sees millions or billions of URLs. A Bloom filter gives the crawler a small, fast first check.

Initialize the Filter
Start with an empty bit array.




Bit Array
0
0
1
0
2
0
3
0
4
0
5
0
6
0
7
0
8
0
9
0
For a toy example, use:

bit array size m = 10
hash functions k = 2
Real filters are much larger and use stronger hash functions. The small numbers here are only to make the idea easy to see.

Add a URL
Add example.com.

The two hash functions point to positions 3 and 7.




Bit Array
0
0
1
0
2
0
3
1
4
0
5
0
6
0
7
1
8
0
9
0
Set bits 3 and 7 to 1.

Add Another URL
Add algomaster.io.

The two hash functions point to positions 1 and 4.




Bit Array
0
0
1
1
2
0
3
1
4
1
5
0
6
0
7
1
8
0
9
0
^
^
Set bits 1 and 4 to 1.

Query a Present URL
Check example.com.




Bit Array
0
0
1
1
2
0
3
1
4
1
5
0
6
0
7
1
8
0
9
0
^
^
Both bits are 1, so the answer is "probably present."

The filter cannot prove that example.com was inserted, because other URLs might have set those same bits. But in this case, we know it was inserted.

Query an Absent URL
Check nonexistent.com.




Bit Array
0
0
1
1
2
0
3
1
4
1
5
0
6
0
7
1
8
0
9
0
^
^
At least one checked bit is 0, so the answer is "definitely absent."

This is the key insight. If nonexistent.com had been added earlier, all of its hash positions would have been set to 1. A single 0 proves it was never added.


3. False Positives
Bloom filters can be wrong in only one direction.

They do not return false negatives for items that were added correctly.
They can return false positives for items that were never added.
A false positive happens when a missing item hashes to positions that other items already set to 1.

Think of it like checking whether a parking spot is occupied from far away. If you see an empty spot, you know there is no car there. If the spot looks occupied, you may need to walk closer and check. The Bloom filter's "yes" means "go check the real source."

False positives are safe only when they cause extra work, not wrong behavior.

Good:

"This SSTable might contain the key; check it."
"This URL might be seen; maybe skip or verify elsewhere depending on crawler policy."
"This cache might contain the object; perform the cache lookup."
Risky:

"This user is definitely blocked."
"This payment has definitely been processed."
"This customer has definitely accepted the legal agreement."
For decisions where correctness matters, use the Bloom filter only as a first filter. Follow every "probably present" answer with an exact check.


4. Sizing a Bloom Filter
Bloom filters need to be sized before you use them.

If you add far more items than planned, too many bits become 1. Once that happens, missing items are more likely to look present, and the false-positive rate rises quickly.

Variables:

n: how many items you expect to add
p: the false-positive rate you can tolerate
m: number of bits
k: number of hash functions
Common formulas:








12
Approximate bits per item:

Target False Positive Rate	Bits per Item	Hash Functions
10%	~4.8	~3
1%	~9.6	~7
0.1%	~14.4	~10
0.01%	~19.2	~13
Example:

For 100 million keys and 1% false positives:








123
That is much smaller than storing 100 million full keys in a hash set, but it is still real memory. Bloom filters trade exact answers for smaller storage. They are not free.


5. Hash Functions
A Bloom filter depends on good hashing.

Two things matter:

Stable: the same item must map to the same bit positions every time.
Spread out: items should be distributed evenly across the bit array.
Do not use language-default hashes when the filter is saved to disk, shared between services, or reused after a restart. For example, Python's built-in hash() is randomized between processes by default.

Production implementations often compute two base hashes and use them to generate all k positions:








1
This is called double hashing. It gives the filter many positions without running many separate hash functions.

Hash quality matters. If the hashes cluster around the same positions, the filter fills unevenly and false positives become more common than expected.


6. Implementation
This implementation uses a small FNV-1a hash function and double hashing.

It is meant for learning. In production, use a well-tested library with strong hashing, serialization support, and clear sizing controls.


Java







12345678910111213141516171819202122232425262728293031
Expected output:








12345
The False result for user:999 is common while the filter is still fairly empty. As the filter gets closer to its planned capacity, some missing items will return True.


7. Production Uses
Bloom filters show up wherever a cheap "definitely not here" answer can save an expensive lookup.

LSM-Tree Databases
Bloom filters are heavily used in LSM-tree storage engines.

In systems such as Cassandra and RocksDB, data is stored in sorted files called SSTables. Once an SSTable is written, it is not changed in place. A single key lookup may need to check several of those files. Bloom filters let the engine skip files that definitely do not contain the key.

This reduces disk reads, especially for missing keys, and helps keep slow requests under control.

In Cassandra, the Bloom filter false-positive chance can be tuned per table. Lower false-positive rates use more memory. Existing SSTables may need compaction or rewriting before new settings fully take effect.

Caches
A Bloom filter can act as a negative pre-check for a cache:

If the filter says "definitely not present," skip the cache lookup.
If the filter says "probably present," check the cache.
This helps most when many requests are misses and the cache lookup is expensive enough to justify maintaining the filter.

Web Crawlers
Crawlers use Bloom filters to reduce duplicate URL processing.

False positives can cause a crawler to skip a URL it has not actually visited. Whether that is okay depends on the crawler's goal.

For broad discovery, skipping a tiny fraction of pages may be acceptable. For compliance, archiving, or exhaustive crawling, the crawler needs an exact visited set or a verification step.

RedisBloom
RedisBloom provides Bloom filters with commands such as:

Shell







123
RedisBloom can create scalable Bloom filters by adding sub-filters as capacity is reached. That is convenient, but it usually costs more memory and CPU than sizing the filter well up front.


8. Variants
A standard Bloom filter is simple, but it has two big limits: it cannot safely delete individual items, and it gets worse when it grows past the planned capacity.

Several variants handle those limits by spending more memory, adding more structure, or optimizing for a specific workload.

Counting Bloom Filter
A counting Bloom filter uses small counters instead of single bits.

To add an item, it increments the counters. To delete an item, it decrements them.

That makes deletion possible, but it costs more memory. It is also safe only if the system deletes items that were actually added. Deleting something that was never added can damage the filter's answers for other items.

Scalable Bloom Filter
A scalable Bloom filter adds new sub-filters as the set grows.

This avoids a hard capacity limit, but lookups may need to check multiple filters. The system also has to manage the combined false-positive rate across all of them.

Cuckoo Filter
Cuckoo filters answer the same "probably present or definitely absent" question, but they store short fingerprints in buckets instead of setting shared bits.

They are often a better fit when deletion is common.

Ribbon Filter
Ribbon filters are newer static filters designed to use less space than Bloom filters. RocksDB has explored Ribbon filters as an alternative for some workloads.

They are attractive when the set is built once and queried many times.


9. Limitations
Bloom filters have clear boundaries:

Limitation	Practical Impact
False positives	Verify a "yes" answer when correctness matters
Does not store original items	You cannot list the set or retrieve stored values
No safe deletion in the standard form	Clearing shared bits can break other items
Must be sized for capacity	Overfilling increases false positives
Depends on good hashing	Bad or unstable hashes make the filter unreliable
Not attack-resistant by default	Attackers may craft inputs that raise false positives
False positives are not bugs. They are part of the design. The real mistake is using a Bloom filter where a false positive changes the answer instead of only causing extra work.


10. Bloom Filters vs Related Structures
Bloom filters are part of a larger family of compact data structures. The easiest way to choose the right one is to ask what question the system needs answered.

Structure	Answers	False Positives	False Negatives	Stores Items?
Hash Set	Is this item present?	No	No	Yes
Bloom Filter	Is this item probably present?	Yes	No	No
Counting Bloom Filter	Is this item probably present, with deletion?	Yes	No, if used correctly	No
Cuckoo Filter	Is this item probably present, with deletion?	Yes	No, if used correctly	Fingerprints only
HyperLogLog	How many distinct items did we see?	N/A	N/A	No
Count-Min Sketch	How often did this item appear?	Overestimates counts	No underestimates in the standard model	No
Use a Bloom filter when you need a compact membership check. Use HyperLogLog for distinct counts. Use Count-Min Sketch for frequencies.


Summary
Bloom filters are compact filters that answer "definitely not present" or "probably present."

Used correctly, they have false positives but no false negatives for items that were added. Sizing matters: choose the expected item count and false-positive rate up front, then derive the bit-array size and number of hash functions.

They work best as first checks before expensive lookups, especially in databases, caches, crawlers, and distributed systems. Avoid them when you need exact answers, legal or security decisions, listing items, or frequent deletion unless you use a variant designed for that need.