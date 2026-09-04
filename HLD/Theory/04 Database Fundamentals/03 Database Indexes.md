Imagine a users table with 100 million rows. If someone logs in with an email address and there is no index on email, the database may have to check row after row until it finds the match.

That is slow. The database needs a shortcut to jump close to the right row. That shortcut is an index.

A database index is an extra data structure that helps the database find rows without scanning the whole table.

Indexes are one of the most useful database performance tools, but they are not free. They can make some reads much faster, but every index also needs storage and must be updated when rows are inserted, changed, or deleted.

An index that helps no real query is just extra work for the database.

Good indexing starts with the queries your application actually runs. In this chapter, we will look at how indexes work, the common index types, and how to choose indexes for real query patterns.

Database Indexing, Full Scan vs B-Tree Lookup
Run the same query as a full table scan and as a B-tree index lookup, row by row.

Full Scan
Index Lookup
Write Overhead

query
SELECT * FROM users WHERE email = 'mia@mail.com'
Table users (heap, insertion order)
page 1
#1
ava@mail.com
free
#2
liam@mail.com
pro
#3
zoe@mail.com
free
#4
ben@mail.com
pro
page 2
#5
kai@mail.com
free
#6
eve@mail.com
pro
#7
noa@mail.com
free
#8
ida@mail.com
pro
page 3
#9
mia@mail.com
free
#10
raj@mail.com
pro
#11
lou@mail.com
free
#12
sam@mail.com
pro
idx_users_email (B-tree)
no usable index
root
ezra | mia
leaf 1
< ezra
ava
#1
ben
#4
eve
#6
leaf 2
ezra .. mia
ida
#8
kai
#5
liam
#2
lou
#11
leaf 3
>= mia
mia
#9
noa
#7
raj
#10
sam
#12
zoe
#3
0
Rows touched
0
Comparisons
0
Pages read
Press Play to run the query step by step.
Ready



0 / 8
1x


1. What Is a Database Index?
An index stores selected column values in a lookup-friendly structure, usually with a reference back to the table row.

For a normal B-tree index, the indexed values are kept in sorted order. That lets the database find a value without checking every row in the table.




users table

Index on email (sorted)

alice@example.com → row 7

bob@example.com → row 3

carol@example.com → row 9

dave@example.com → row 1

row 1: Dave

row 3: Bob

row 7: Alice

row 9: Carol

For example:

Sql







123456789
Now a query like this has a fast path:

Sql







123
Without an index, the database may need to scan the table. With the index, it can search for the email in the index and then fetch the matching row.

The exact behavior depends on the database, table size, statistics about the table, and query plan. Small tables are often faster to scan. Large tables with filters that match only a small number of rows usually benefit from indexes.


2. How Indexes Work
Most indexes work in two steps:

Search the index for matching values.
Use the row reference to fetch the table rows, unless the index already contains everything needed.



Query
WHERE email = ...

Index on email

Row Reference

Users Table

Result

An index helps most when it greatly reduces the number of rows the database must inspect.

This idea is called selectivity. In plain terms, it means "how much does this filter narrow the search?" A highly selective condition matches only a small part of the table.

Column	Example	Selectivity	Index Usefulness
email	Unique per user	High	Usually very useful
user_id	Many rows per user, but still narrows the search	Medium to high	Often useful
status	ACTIVE, INACTIVE	Low	Sometimes useful with other columns
is_deleted	true, false	Very low	Usually weak alone
Columns that match many rows can still be useful as part of a multi-column index or filtered index. But an index on only a boolean column is often not worth much because true or false may match a large part of the table.


3. B-Tree Indexes
The most common index structure in relational databases is a B-tree, or a close version of it such as a B+ tree.




Root
[ 50 | 100 ]

[ 10 | 30 ]

[ 70 | 85 ]

[ 120 | 150 ]

1..9

10..29

30..49

50..69

70..84

85..99

100..119

120..149

150..200

B-tree indexes keep keys sorted. That makes them useful for:

Equality lookups: email = 'alice@example.com'
Range queries: created_at >= '2026-01-01'
Ordered reads: ORDER BY created_at DESC
Prefix matches in some cases: name LIKE 'Al%'
B-trees are built to work well with disks and memory. Each node contains many keys, so the tree stays shallow even with millions of rows. A lookup usually touches only a small number of pages instead of scanning the whole table.

B-tree indexes are the default for many CREATE INDEX statements because they work well for common application queries.


4. Composite Indexes
A multi-column index contains more than one column.

Sql







12
This index is useful for:

Sql







12345
The database can find one user's orders and read them in newest-first order.

Column order matters. The index (user_id, created_at) is not the same as (created_at, user_id).

For B-tree indexes, a useful rule of thumb is:

Put equality filters first.
Then range filters.
Then columns used for ordering.
This is not a universal law, but it is a good starting point. Always confirm with a query plan, such as EXPLAIN.


5. Covering Indexes
A covering index contains all columns needed by a query.

For example:

Sql







12345
A covering index might be:

Sql







12
If the database can answer the query from the index alone, it can avoid extra table lookups. PostgreSQL calls this an index-only scan when its internal visibility rules allow it. Other databases have similar ideas.

Covering indexes are wider, though. Wider indexes use more disk, more memory, and more write work.

Use them for important, frequently run queries where you have measured the benefit.


6. Unique, Primary, and Clustered Indexes
Indexes are also used to enforce rules and control how some databases store table data.

Primary and Unique Indexes
A primary key usually creates a unique index automatically.

Sql







12345
The primary key index helps look up rows by id. The unique index on sku prevents duplicate SKUs and supports fast lookups by SKU.

Clustered Indexes
A clustered index affects the storage order of table data, depending on the database.

This is database-specific:

In SQL Server, the clustered index is the table storage structure.
In MySQL InnoDB, the table is clustered by the primary key.
In PostgreSQL, CLUSTER is a one-time operation that rewrites a table in the order of an index. New writes after that go wherever space is available, so the order slowly drifts and the command must be rerun if keeping nearby rows together matters.
Clustered layout can help range scans because nearby rows are stored close together. But it also affects writes. Random primary keys can scatter writes more than increasing keys.


7. Specialized Index Types
Different query patterns need different index structures.

Hash Indexes
A hash index passes each key through a hash function to find the bucket that holds the matching row reference. For equality lookups, this can be a direct jump to the right bucket.




alice@...

hash(key)

bob@...

carol@...

dave@...

bucket 0

bucket 1

bucket 2

bucket 3

row 3

row 7

row 1

row 9

Hash indexes do not help with range queries or ordered reads because hash values do not keep the original key order.

In many relational databases, B-tree indexes are still the safer default because they support equality, ranges, and ordering.

Bitmap Indexes
A bitmap index represents matching rows as a string of bits. Each bit says whether a row matches a value.

When a query has several filters, the database can combine those bitmaps quickly.




Rows
1: US, PRO, ACTIVE
2: EU, FREE, ACTIVE
3: US, PRO, INACTIVE
4: US, FREE, ACTIVE
5: EU, PRO, ACTIVE

region = US:
1 0 1 1 0

plan = PRO:
1 0 1 0 1

status = ACTIVE:
1 1 0 1 1

Bitwise AND

Result bitmap:
1 0 0 0 0
→ row 1 matches

Bitmap indexes work well for analytics, especially for columns with only a few possible values, such as region, plan type, or status. The query below maps cleanly to an AND across three column bitmaps:

Sql







123
Bitmap indexes are usually a poor fit for busy application tables with many writes. Updates can be expensive, and locking can be tricky depending on the database.

Partial or Filtered Indexes
A partial index stores only rows that match a condition.

Sql







123
This is useful when most queries care about only part of a table, such as open orders, active users, or undeleted records. The index is smaller and cheaper than indexing every row.

Expression Indexes
An expression index stores the result of an expression.

Sql







12
This supports queries such as:

Sql







123
Without an expression index, applying a function to a column can prevent a normal index from being used.

Full-Text, Spatial, and Inverted Indexes
Some queries need specialized indexes:

Full-text indexes for searching words and phrases.
Spatial indexes for location-based queries.
Inverted indexes for documents, arrays, JSON fields, or tags.
Block-range indexes, such as BRIN in PostgreSQL, for very large tables that are naturally ordered, such as append-only logs.
Use these when the query pattern does not fit a normal B-tree.


8. Costs of Indexes
Indexes improve some reads by adding write and storage work.

Every index adds cost:

Cost	Why It Matters
Storage	Indexes can be large, especially wide multi-column indexes
Slower writes	Inserts, updates, and deletes must update indexes
Memory use	Hot indexes compete with table data for cache
Migration time	Creating indexes on large tables can take time and resources
Planning work	Extra indexes give the database more choices to evaluate
An index on a column that changes often is more expensive than an index on stable data. A wide index over many columns is more expensive than a narrow one. An unused index is only extra cost.

This is why production systems should track unused and duplicate indexes.


9. Choosing the Right Index
Start from the query, not the table.

For each important query, ask:

Which rows does it filter?
Which columns does it join on?
How does it sort?
How many rows does it return?
Which columns does it select?
How often does it run?
How often do the indexed columns change?
For example:

Sql







123456
A reasonable index could be:

Sql







12
This matches the equality filters first, then the sort column.

Confirm with EXPLAIN. If the database does not use the index, the index may not match the query, the filter may match too many rows, or statistics about the table may be stale.


10. Common Mistakes
Avoid these mistakes:

Adding indexes without looking at real queries.
Creating many single-column indexes when one multi-column index matches the query.
Putting multi-column index columns in the wrong order.
Indexing columns alone when they match too much of the table.
Forgetting that indexes slow down writes.
Keeping unused indexes forever.
Assuming LIMIT makes a query cheap without a matching index.
Expecting a normal B-tree index to support LIKE '%term%'.
Applying functions to indexed columns without expression indexes.
Forgetting to check the plan after adding an index.

Summary
Indexes are extra data structures that help databases find, join, filter, and sort rows more efficiently.

B-tree indexes handle most common application queries. Multi-column indexes are often more useful than several single-column indexes when they match the query. Covering, partial, expression, full-text, spatial, bitmap, and hash indexes solve more specific problems.

The tradeoff is cost: indexes use storage, slow writes, and require maintenance. Good indexing starts with real queries, measured plans, and a clear understanding of which filters narrow the data.

The right index can turn an expensive query into a predictable lookup. The wrong set of indexes can slow the whole system down.