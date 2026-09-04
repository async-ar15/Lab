Consider a money transfer between two bank accounts.

The transfer looks like one action to the user, but the database has to make at least two changes:

subtract money from account A
add money to account B
If the database subtracts the money from account A and then crashes before adding it to account B, money disappears. That is the kind of bug transactions are designed to prevent.

A transaction is a group of database operations that should succeed or fail as one unit. Either the whole group is saved, or none of it is.

ACID is the set of guarantees that makes transactions reliable:

Atomicity: all changes happen, or none happen
Consistency: saved data follows the rules you defined
Isolation: transactions running at the same time do not step on each other in unsafe ways
Durability: once a transaction is committed, the database can recover it after a crash
ACID does not make an application correct by itself. You still have to model the data well and write the right business logic. But ACID gives you a strong foundation for grouping related changes, protecting important rules, handling concurrency, and recovering after failures.


1. What a Transaction Looks Like
Here is a simple money transfer wrapped in a transaction.

Sql







12345678910111213
BEGIN starts the transaction. COMMIT asks the database to make the changes final.

If something goes wrong before COMMIT, the database can roll the transaction back. A rollback means the database throws away the changes made inside that transaction. If COMMIT succeeds, the database treats the transaction as saved according to its durability and isolation settings.


2. Atomicity
Atomicity means a transaction is all-or-nothing.

Either every operation in the transaction takes effect, or none of them do.

In the transfer example, atomicity prevents the broken state where account A is debited but account B never receives the credit.

How Atomicity Works
Databases track the state of each transaction. Changes made inside a transaction are not treated as final until the transaction commits.




yes

no

BEGIN

Run Statements

Success?

COMMIT

ROLLBACK

Inside the database, this requires bookkeeping. The database may keep log records, old row versions, undo information, redo information, locks, or conflict checks.

The details vary by database engine. The important idea is simple: if the transaction fails, the database must not leave behind a half-finished result.


3. Consistency
Consistency means a committed transaction must leave the database in a valid state.

In plain terms: after the transaction commits, the data should still follow the rules the database knows about.

Those rules can include:

primary key uniqueness
foreign key references
NOT NULL constraints
CHECK constraints
valid data types
triggers or stored procedures
Example schema:

Sql







1234
This CHECK constraint prevents the database from storing a negative balance.

But the database cannot know every business rule on its own.

For example, it will not automatically know that a payment processor, inventory system, and shipment service must all agree about the same order. Rules like that usually require application logic, background workflows, retries, and careful handling when multiple systems are involved.

So consistency is shared across layers:

the schema protects data types, keys, constraints, and relationships
the transaction groups related database changes together
the application handles business rules the database cannot express
external workflows handle retries, duplicate requests, and cross-system repair

4. Isolation
Isolation controls what transactions can see when they run at the same time.

This matters because production systems rarely do one thing at a time. Two users may check out at the same time. Two workers may process jobs at the same time. Two API requests may update the same account at the same time.

Without isolation, one transaction could read another transaction's half-finished work, or two transactions could both make decisions using old data.

Common concurrency problems:

Problem	What Happens
Dirty read	A transaction reads data another transaction has not committed
Non-repeatable read	A transaction reads the same row twice and sees different committed values
Phantom read	A repeated query returns a different set of matching rows
Lost update	Two transactions overwrite each other's changes
Write skew	Two transactions read overlapping data and make writes that violate a rule together
Isolation Levels
Most relational databases let you choose an isolation level.

Higher isolation protects you from more concurrency bugs. The tradeoff is that transactions may wait longer, conflict more often, or need retries.

Isolation Level	Protects Against	Tradeoff
Read uncommitted	Very little	Fast, but can read uncommitted changes
Read committed	Dirty reads	Still allows some surprises across repeated reads
Repeatable read	Many repeated-read problems	Exact behavior depends on the database
Serializable	Makes transactions behave as if they ran one at a time	More blocking, failed transactions, or retries
Database behavior differs. PostgreSQL REPEATABLE READ, MySQL/InnoDB REPEATABLE READ, and SQL Server isolation settings are not identical, even when the names look similar. Always check how your database actually behaves.

Example: Overselling Inventory
Suppose one item is left in stock and two buyers check out at the same time.

Sql







12345678910111213
If two transactions both read stock = 1 before either one commits, both may think the item is available.

A safer design usually needs one of these:

a conditional update, such as WHERE stock > 0
a row lock, such as SELECT ... FOR UPDATE
a unique reservation record so the same item cannot be reserved twice
serializable isolation plus retry logic
a reservation workflow that handles duplicate requests safely
Choosing an isolation level is part of the design. It is not just a database setting you pick once and forget.

How Databases Implement Isolation
Databases use a few common techniques to enforce isolation:

Locks make one transaction wait when another transaction is changing the same data.
MVCC lets readers see a stable snapshot while writers create newer versions of rows.
Range locks protect a group of rows, such as "all seats in this row" or "all products below this price."
Conflict detection lets the database stop a transaction when it cannot safely run alongside another one.
Each technique has a cost. Some reduce concurrency. Some use more memory. Some make applications retry failed transactions. There is no free version of isolation.


5. Durability
Durability means that once a transaction commits, the database can recover it after a failure, as long as the failure is within the cases the database was set up to handle.

This is an important distinction. Durability does not mean data can never be lost under any possible disaster. It means the database has written enough recovery information to keep committed transactions safe for the failure cases it promises to handle.

Write-Ahead Logging
Many databases use a write-ahead log, or WAL.

The rule is simple:

Write the recovery record before relying on the changed data page.




Change Data

Append WAL

Flush Commit Record

Acknowledge Commit

Write Data Pages Later

The database may not immediately write every changed table page to its main data files. That would be slow. Instead, it first writes enough information to the WAL.

If the database crashes after commit but before the changed pages reach the main files, recovery can replay the WAL and restore the committed changes.

Durability Settings Matter
Some databases let teams trade durability for speed. For example, a database may acknowledge commits before every log flush and rely on periodic flushing instead.

That can be acceptable for caches, analytics buffers, or data that can be rebuilt. It is usually not acceptable for payments, orders, identity records, or compliance data.

Replication can improve durability if a machine fails, but asynchronous replication can still lose recently acknowledged writes during failover. Backups protect against deletion, corruption, and human mistakes, but they are not part of the normal commit path.


6. Putting ACID Together
Now put the four ideas together with an order placed inside one database:

Sql







123456789101112
In this example, ACID means:

Property	What It Protects
Atomicity	The stock update and order insert succeed or fail together
Consistency	Constraints help prevent invalid rows, such as negative stock if modeled correctly
Isolation	Buyers checking out at the same time do not rely on half-finished changes
Durability	Once committed, the database can recover the order after a crash
This example stays inside one database on purpose.

Charging a credit card, sending an email, or calling a warehouse API cannot be rolled back by the database. Once that external action happens, the database transaction cannot magically undo it.

For those cases, systems usually use patterns such as duplicate-safe request keys (idempotency keys), outbox tables, retries, and sagas. The names sound advanced, but the goal is practical: make external work safe to retry and easier to repair if one step fails.


7. Common Mistakes
Most transaction bugs come from expecting transactions to do more than they actually promise.

Watch for these mistakes:

Treating ACID as full application correctness. ACID helps, but the transaction still has to contain the right operations and constraints.
Ignoring isolation level. A transaction at READ COMMITTED can still have concurrency bugs.
Doing slow external calls inside transactions. Long transactions hold locks or row versions longer and can slow down other work.
Assuming rollback undoes external side effects. A database rollback cannot unsend an email or undo a payment API call.
Using transactions instead of duplicate-safe retries. Retries still need safe request handling so the same request does not create duplicate work.
Forgetting durability settings. Some databases expose faster but weaker durability options.

Summary
ACID transactions let a database group related changes so they succeed or fail as one unit.

Atomicity means all operations in a transaction commit, or none do.
Consistency means committed data follows the rules defined in the schema and database logic.
Isolation means transactions running at the same time are controlled so they do not interfere in unsafe ways.
Durability means committed changes can be recovered after failures, within the database's durability settings.
Use transactions for data that must change together. Keep them short, define constraints clearly, choose the right isolation level, and treat external side effects with extra care.