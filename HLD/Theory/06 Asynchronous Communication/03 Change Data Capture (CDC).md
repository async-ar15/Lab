Most production databases do not live alone. The same order, user, product, or payment data often needs to show up in several other places: a search index, a cache, a data warehouse, a fraud system, a recommendation pipeline, or another service.

A common but painful approach is to run batch jobs:

Query the database every few minutes or hours.
Find rows that changed.
Copy them somewhere else.
That can work for reports that do not need fresh data. But it has real costs:

The copied data is often stale.
The database may be scanned again and again.
Failure cases are messy because a job can stop halfway through.
Change Data Capture (CDC) means capturing database changes as they happen and sending those changes to other systems.

Change Data Capture, Log-based
Mutate a source table and watch Debezium tail the log and stream each change through Kafka to every downstream sink.


Source DB
Postgres
customers
1
Ada
120
2
Lin
80
WAL / binlog
transaction log
no changes yet
Debezium
connector
offset: 40
idle
Kafka topic
dbserver1.inventory.customers
no events yet
Search index
1:120
2:80
Cache
1:120
2:80
Data warehouse
1:120
2:80
Insert
Update
Delete
Back
Step
Speed: 1×
Mutate the source table. Each change flows through the log to Debezium, Kafka, and the sinks.

Fullscreen
CDC is especially useful when the database is the source of truth and other systems need a near-real-time copy. The application writes to the database as usual. CDC watches those committed changes and passes them along.

This chapter covers what CDC captures, the main ways to implement it, and the problems to watch for in production.


1. What CDC Captures
CDC turns database changes into change events. The usual events are:

A row was inserted.
A row was updated.
A row was deleted.
Some CDC systems can also report table structure changes, such as adding or renaming a column.

A change event often contains:

Field	Purpose
Operation	Insert, update, or delete
Table	Source table or collection
Primary key	Identifies the changed row
Before value	Previous row state, if available
After value	New row state, if available
Commit position	Log sequence number, binlog position, or offset
Commit timestamp	When the database committed the change
Transaction details	Helps keep related changes in the right order
Example:

Json







123456789
The exact event format depends on the database and the CDC tool, but the idea is the same: describe what changed and where that change happened.


2. How CDC Works
At a high level, a CDC pipeline has four steps:




Database

Capture Changes

Build Change Events

Stream or Topic

Downstream Consumers

Capture: Detect committed changes from the source database.
Build events: Convert database-specific records into a change event format.
Publish: Write events to a stream, queue, topic, or destination system.
Consume: Other systems read the events and update their own data.
CDC should only publish changes that actually committed. If a database transaction rolls back, other systems should never see those changes.


3. CDC Approaches
There are three common ways to implement CDC.

Timestamp-Based CDC
Timestamp-based CDC uses a column such as updated_at to find changed rows.

Sql







123
This is simple, but it is not the strongest form of CDC. It is still polling. The timestamp just tells the polling job what to look for next.

It is easy to understand, works with almost any database, and is good enough for small systems and low-stakes sync jobs.

The downsides are easy to miss at first:

Hard deletes disappear unless you use soft deletes.
A missed updated_at update means the change may never be copied.
If a row changes three times between polls, you may only see the final value.
Clock precision matters. Two changes with the same timestamp can be tricky.
Without a good index on updated_at, every poll can become an expensive scan.
Use this approach when the job is simple and it is acceptable to be a little late or miss intermediate states.

Trigger-Based CDC
Trigger-based CDC uses database triggers to write changes into an audit or outbox table.

Sql







123456789101112131415
Triggers can capture deletes and before/after values. They can also be customized per table. This makes them useful for audit tables and for the transactional outbox pattern, especially when transaction logs are hard to access.

The cost is that the trigger runs during the normal database write. On a busy table, that extra work can slow the application down.

Triggers can also become hard to maintain. If the table changes, the trigger may need to change too. If the trigger has a bug, normal application writes can fail.

Triggers are useful, but they should be treated as production application code.

Log-Based CDC
Log-based CDC reads the database's transaction log. This is the log the database already uses to recover from crashes and replicate data.

Examples include the MySQL binlog, the PostgreSQL WAL through logical replication, the SQL Server transaction log through SQL Server CDC tables, and MongoDB oplog or change streams.




Application Writes

Database

Transaction Log

CDC Connector

Change Topic

This is the preferred approach for many high-volume systems because the database already writes these logs as part of normal operation.

Log-based CDC usually gives you:

Inserts, updates, and deletes.
Better ordering because it follows the commit log.
Less impact on application writes than triggers.
Better scaling for busy databases.
The ability to resume from a saved log position after an outage.
The trade-off is more setup and ongoing care. You need database-specific settings, safe permissions, enough log history for recovery, a plan for large initial snapshots, and a clear approach for table changes.

Log-based CDC is often the best production default, but it is not "set and forget."


4. CDC vs Events vs Event Sourcing
CDC is often confused with event-driven architecture and event sourcing.

They are related, but not the same.

Pattern	Source of Truth	What It Publishes
CDC	Database tables	Row-level changes after they commit
Domain events	Application business logic	Business facts such as OrderPlaced
Event sourcing	Append-only event log	Events are the primary record of state
CDC usually says, "This row changed."

A domain event says, "This business thing happened."

Those are not always the same thing. A single checkout may update an order row, a payment row, an inventory row, and an audit row. Another service may not care about those table details. It may only care that an OrderPlaced event happened.

The Outbox Pattern
The transactional outbox pattern combines business events with CDC.

Instead of writing to the database and publishing to Kafka in two separate steps, the application writes both records in one database transaction:

The normal business data.
A row in an outbox table that describes the event.
CDC then streams the outbox table.




single DB transaction

Application

Database

orders table

outbox table

CDC Connector

Event Topic

This avoids the classic dual-write problem: the database write succeeds, the event publish fails, and other systems never hear about the change.

With an outbox, the event row is committed in the same transaction as the business change. CDC publishes it later. If the transaction commits, the event will be published. If the transaction rolls back, no event is published.


5. Common Use Cases
CDC shows up whenever one system needs to react to changes in another system's database. Instead of each system polling the database separately, they can all read from the same stream of changes.

Search Indexing
When product, article, or user records change, CDC can update a search index such as Elasticsearch or OpenSearch.




Product DB

CDC

Product Changes

Search Indexer

Search Index

This keeps search up to date without forcing the application to write to both the database and the search index in the request path.

Data Warehousing
CDC can feed a lakehouse or warehouse faster than a nightly batch job.

Teams often use it to copy production data into analytics systems. Analysts can join, clean, and query the data there without putting heavy load on the production database.

Cache Invalidation
When a row changes, CDC can send a cache invalidation event for related cache keys.

This is often safer than trying to update cached values directly because consumers can delete stale entries and let the normal read path rebuild them.

Replication Between Services
CDC can help one service build a local read model from another service's data.

Use this carefully. Copying raw tables between services can create hidden coupling. If another service needs business meaning, prefer stable domain events or outbox events instead of exposing your table shape.

Audit and Compliance
CDC can preserve a history of changes for audit, debugging, or compliance needs.

If this is the goal, decide the rules up front:

How long should events be kept?
Who can read them?
Can records ever be changed or deleted?
How will personal or sensitive data be protected?

6. Debezium and Kafka Example
Debezium is a common open-source CDC platform. It reads database logs and publishes change events, often through Kafka Connect into Kafka topics.

A simplified modern MySQL connector configuration looks like this:

Json







12345678910111213141516
The key pieces are:

topic.prefix: Adds a prefix to the output topic names for this source.
table.include.list: Limits CDC to selected tables.
Stored offsets: Let the connector resume from the last processed log position.
Schema history: Helps Debezium understand old change events after table structures change.
In production, you also need a plan for snapshots, credentials, topic names, table changes, monitoring, and log retention.


7. Operational Challenges
CDC pipelines are production systems. They need the same care as APIs and databases.

Initial Snapshot
Most CDC tools need an initial snapshot before they can stream new changes. The snapshot copies the current table state so consumers have a starting point.

Large tables can make this expensive. Plan for how long the snapshot will take, how much load it will put on the database, and whether consumers can handle many old rows arriving at once.

Lag
CDC lag means other systems are behind the source database.

Useful signals include:

How far the connector is behind the database log.
How old the oldest unprocessed event is.
How far consumers are behind the stream.
How many writes to destination systems are failing.
How many events are landing in dead-letter queues.
Log Retention
If the connector is down longer than the database keeps its transaction logs, it may not be able to resume. At that point, you may need a new snapshot or a manual repair.

For PostgreSQL, unused replication slots can hold on to WAL files and grow disk usage. For MySQL, binlog retention must be long enough to cover expected connector outages.

Ordering
CDC follows database commit order at the source. But that order can change later if events are split across partitions, transformed, or processed by many workers.

Consumers should use primary keys, versions, commit positions, or timestamps to ignore older updates that arrive late.

Deletes
Deletes are easy to mishandle.

Other systems need to know what a delete means. Should they remove the record? Mark it inactive? Write a tombstone event? Keep it for legal or audit reasons?

Make the delete behavior clear before consumers depend on it.

Schema Evolution
Database table changes can break consumers if the CDC event format changes unexpectedly.

Prefer additive changes, such as adding a new optional field. Version event formats when needed, and test database migrations against CDC consumers before release.

Security and Privacy
CDC can expose everything in a table, including fields that were never meant to be widely shared.

Protect CDC streams by:

Giving the connector only the permissions it needs.
Masking or excluding sensitive columns.
Encrypting data in transit and at rest.
Controlling who can read CDC topics.
Applying retention rules for personal data.

8. Best Practices
A few habits keep a CDC pipeline reliable once it carries real traffic:

Prefer log-based CDC for high-volume production systems when the database supports it.
Use the outbox pattern for business events instead of asking consumers to guess meaning from raw table changes.
Make consumers safe to retry because CDC events can be replayed or duplicated.
Monitor lag and log retention so the connector can recover after outages.
Limit captured tables and columns to what other systems actually need.
Handle deletes deliberately with clear tombstone, hard-delete, or soft-delete behavior.
Version event formats and treat CDC streams as contracts.
Avoid leaking database details across service boundaries when domain events would be clearer.
Test snapshots, restarts, and replay before relying on CDC during an incident.

Summary
Change Data Capture streams committed database changes to other systems. It is useful for search indexing, data warehousing, cache invalidation, read-model replication, audit trails, and outbox-based event publishing.

Timestamp polling is simple but limited. Triggers are flexible but add write-path overhead. Log-based CDC is usually the strongest production approach, but it requires careful operation.

CDC is not the same as event sourcing, and raw row changes are not always good domain events. Used well, CDC reduces dual writes and keeps systems in sync. Used casually, it can leak database details, spread sensitive data, and create fragile coupling.