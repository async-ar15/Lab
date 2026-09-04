A client sends a request, the network drops the response, and now the client does not know whether the operation happened. Retrying feels natural, but a blind retry can charge a customer twice or create a duplicate order.

Idempotency is the property that makes retries safe: running an operation multiple times has the same intended effect as running it once.

Distributed systems need retries. Networks fail, clients time out, processes crash, message brokers deliver the same message again, and operators replay failed jobs. Without idempotency, those retries can charge a card twice, create duplicate orders, send the same email three times, or leave other systems in the wrong state.

Idempotency applies well beyond HTTP APIs. It matters anywhere work can be retried, replayed, or delivered more than once: payment APIs, order creation, webhook handlers, message consumers, background jobs, data pipelines, and model-run submission.

This chapter explains what idempotency means and how to make retries safe with idempotency keys.

Idempotency Keys, Safe Retries
A client retries a charge after a timeout. See how an idempotency key prevents a double charge.

No key
Idempotency key

Client
checkout page
API Server
POST /charge
key store
empty
Database
charges table
no rows yet
charges
0
customer paid
$0
keys stored
0
no dedupe
A client retries a request after a timeout. What happens to the charge?



1 / 6
1x


1. What Idempotency Means
An operation is idempotent when running it multiple times has the same intended effect as running it once.

Operation	Idempotent?	Reason
Set user status to ACTIVE	Yes	Repeating the write leaves the user active.
Delete session sess_123	Yes	The session is gone after the first delete.
Upload the same file to the same object key	Usually	Replacing the same file with the same content is safe. It is not safe if the storage system creates a new version or appends data each time.
Increment login count	No	Each retry increments the counter again.
Append an item to a list	No	Each retry appends another item.
Create payment without an operation ID	No	Each retry can create a new payment.
Idempotency is about effect, not identical responses.

For example, the first DELETE /sessions/sess_123 might return 204 No Content. A retry on a resource that is already gone often returns 404 Not Found, though some APIs choose to return 204 again to keep retries simple. The responses differ, but the final state is the same: the session no longer exists.

For a better client experience, many APIs return the original response for a duplicate retry. That way the caller does not have to treat a successful duplicate as an error.


2. Natural Idempotency vs Engineered Idempotency
Some operations are naturally idempotent because they set a final state.

Sql







123
Running this statement several times leaves the user in the same state.

Other operations are not naturally idempotent because they change the value again or create something new each time.

Sql







123
Each retry adds 10 more units. To make this safe, attach a stable operation ID:

Sql







123
The operation ID turns "add 10 units" into "apply shipment shipment_789 once."

This distinction matters:

Natural idempotency: The operation itself sets a final state.
Engineered idempotency: The system records a stable operation ID and uses it to detect duplicates.
Payments, order creation, email delivery, webhook processing, and job submission usually need engineered idempotency.


3. Idempotency Keys
An idempotency key is a stable ID for one logical operation.

Plaintext







12345
The payment details travel in the body below those headers:

Json







12345
The same logical operation must reuse the same key on every retry. A new key means a new operation.

Good keys are:

Client-generated: The key exists before the first request is sent.
Stable across retries: The same operation always uses the same key.
Unique for different operations: Two different payment attempts do not share a key.
Scoped: The server checks the key within a user, account, tenant, endpoint, or operation type.
Bound to the request: The server rejects the same key if the request body changes.
Stored durably: The key survives restarts and failover.
Do not generate the idempotency key only after the server receives the request. If the response is lost, the client will not know which key to use on retry.


4. Server-Side Implementation
A production system usually stores idempotency records in durable storage, such as a database.

Sql







12345678910111213
The scope prevents accidental collisions across tenants or endpoints. The request_hash catches client bugs where the same key is reused with a different request body. The status tells the server whether this is the first request, a completed retry, or a request that is still running.

Request Flow
Every incoming request follows the same basic path: check the key, reserve it if it is new, then decide what to do based on what the server found.




No

Yes

new key

completed

same key different body

in progress

Request Arrives

Key present?

Reject or process as non-idempotent

Reserve key atomically

Reserve result

Perform operation

Return stored response

Reject conflict

Return retryable response

Store response and resource id

Return response

The reserve step must be atomic, meaning two requests cannot both claim the same key at the same time. This pattern is unsafe:

Plaintext







1234
Two retries running at the same time can both pass step 2 and both process the payment. Use a unique constraint, transaction, lock, compare-and-set operation, or a business table keyed by the operation ID.

Reserving a Key
A single atomic insert makes the reservation safe when retries arrive at the same time. The query below tries to claim the key and does nothing if another request already holds it.

Sql







123
If the insert succeeds, this request owns the operation. If it does not, the server loads the existing record and decides whether to return a stored response, reject a changed request body, or ask the client to retry later.

Status and Lock Lifecycle
The status and locked_until columns keep the record safe while the operation is running, even if the server crashes. A record moves through a small set of states:

Reserved as IN_PROGRESS. The successful insert sets status = IN_PROGRESS and locked_until = now() + timeout. This claims the operation for this request. The locked_until value is a lease, which means "this request owns the work until this time."
Completed. When the operation finishes, the owner updates the row to status = COMPLETED, stores response_status and response_body, and sets completed_at. A later duplicate that sees COMPLETED returns the stored response without running the operation again.
The tricky case is when the owner crashes after reserving the key but before finishing the operation. Without a lease, the row would stay at IN_PROGRESS forever, and every retry would be told "still in progress." locked_until fixes this. When a duplicate finds an IN_PROGRESS record, it checks the lease:

If locked_until is still in the future, the original request is probably still running, so the server returns a retry-later response.
If locked_until has passed, the original owner is probably gone, so the duplicate claims the lease by extending locked_until and runs the operation.
This is why the work behind an idempotency key must also be safe to retry. If the first owner crashes, another request may take over the same key. The lease should be long enough for a normal slow operation, but short enough that a real crash does not block retries for too long.


5. Handling In-Progress Requests
Retries can arrive at the same time. A user double-clicks, a mobile client retries aggressively, or two workers process the same job.

When a duplicate request arrives while the first request is still running, the server has a few options:

Strategy	Behavior	Tradeoff
Return 409 Conflict	Tell the client the operation is in progress	Simple, client must retry
Return 202 Accepted	Expose an operation resource to poll	Good for long-running work
Wait briefly	Block until the first request completes	Easier for clients, ties up server resources
Return stored partial state	Useful for workflows with clear states	Requires careful state modeling
For long-running operations, returning an operation resource is often cleaner than holding the connection open:

Json







12345
The idempotency key identifies the operation. The operation resource describes its current state.


6. External Side Effects
The hardest idempotency bugs happen when the operation calls another system.

Example payment flow:

Reserve idempotency key.
Create local payment attempt.
Call payment provider.
Provider charges the card.
Local service crashes before saving the provider result.
On retry, the service must not charge again. Better designs reduce this risk:

Pass an idempotency key to the external provider when the provider supports it.
Store a local payment attempt before calling the provider.
Store the provider's charge ID as soon as it is known.
Recover by querying the provider using its request ID or saved call details.
Use a workflow engine for long-running, multi-step operations when appropriate.
Keep side effects behind database uniqueness checks where possible.
No database transaction can include every external API. The design needs a recovery path for this case: "the external action happened, but our local state did not finish updating."


7. Idempotency in Messaging
Message systems often provide at-least-once delivery, which means a consumer may receive the same message more than once. This can happen after a crash, rebalance, visibility timeout, or manual replay.

Consumers should treat duplicate delivery as normal.




operation_id exists

Message
operation_id=ship_123

Transaction

Apply business change

Record operation_id

ACK / commit offset

Duplicate later?

Skip work and ACK

Durable duplicate detection can take several shapes: a processed-message table with a unique message ID, a business table keyed by operation ID, a state machine that ignores old transitions, or a durable key-value store.

The business write is often the best place to detect duplicates. A shipments table with a unique shipment_id is stronger than an in-memory set that says a message was processed.

Commit broker offsets or acknowledge messages only after the business operation is stored durably. If the process crashes after the write but before the ack, the broker may deliver the message again. Idempotency makes that safe.


8. HTTP Method Behavior
HTTP defines idempotency at the method level. The important part is the intended effect of the request.

Method	Idempotent by HTTP Meaning?	Practical Meaning
GET	Yes	Reads data. Logging and metrics may still happen.
HEAD	Yes	Same idea as GET, but without a response body.
PUT	Yes	Replaces or creates a resource at a known URI.
DELETE	Yes	Leaves the resource deleted after the first successful delete.
POST	No by default	Often creates a new child resource or starts processing.
PATCH	Depends	Setting a field can be idempotent. Incrementing a field is not.
Examples:

PUT /users/123/status with { "status": "ACTIVE" } is idempotent.
PATCH /users/123 with { "status": "ACTIVE" } can be idempotent if defined as a set operation.
PATCH /inventory/1 with { "increment_by": 10 } is not idempotent.
POST /payments without an idempotency key is not idempotent.
POST /payments with an idempotency key can be engineered to be idempotent.
Idempotent does not mean "nothing else happens." Logging, metrics, and audit records may still happen. It means repeated identical requests have the same intended effect on the resource.


9. Retention and Scope
Idempotency records cannot be kept forever in every system. They contain request hashes, response bodies, resource IDs, timestamps, and sometimes tenant-specific details.

Keep records long enough to cover realistic retry windows. That includes client retries after network failures, mobile apps coming back online, broker redelivery windows, provider retry schedules for webhooks, operator replay windows, and audit needs.

If a key expires, the server may treat a late retry as a new operation. Document the retention window as part of the API contract.

Scope matters too. The same key string may appear in different tenants or endpoints. Store keys under a scope such as:

Plaintext







1
This prevents one tenant or operation type from colliding with another.


10. Common Pitfalls
Idempotency designs usually break in a few predictable ways, often around key handling or storage.

New Key on Every Retry
If each retry gets a new key, the server sees each attempt as a new operation. The key must represent the logical operation, not the individual HTTP attempt.

Payload Mismatch
The same key with a different request body should be rejected. Returning the old result for a different request can hide client bugs and break workflows.

Non-Atomic Reservation
Check-then-insert logic can race. Reserve the key atomically, or let a unique constraint protect the business operation.

Dedupe in Memory
An in-memory set disappears on restart and is not shared across replicas. Use durable storage in production.

Side Effects Before Reservation
Calling an external provider before reserving the key leaves no durable record for retries to find.

Confusing Idempotency with Exactly-Once
Idempotency does not prove that the code ran once. It makes repeated attempts end with one intended effect.

Exactly-once promises, where available, are usually limited to one system boundary. A broker may provide exactly-once processing for topic-to-topic workflows, but that does not automatically make calls to an external payment provider exactly once.


11. Best Practices
Use this checklist when building retry-safe endpoints, consumers, and calls to other services.

Use stable operation IDs for operations with side effects.
Require idempotency keys for retryable endpoints that can create side effects, such as payment creation and job submission.
Scope keys by tenant, caller, endpoint, or operation type.
Store request hashes and reject mismatches.
Reserve keys atomically.
Store the original response when client consistency matters.
Keep business state and duplicate-detection state in the same transaction when possible.
Pass idempotency keys to providers that support them.
Treat message consumers as duplicate-tolerant by default.
Document key retention windows.
Test concurrent retries, timeouts, process crashes, provider failures, and broker redelivery.

Summary
Idempotency makes retries safe by letting a system receive the same logical operation more than once without creating duplicate side effects. Natural idempotency comes from setting a final state. Engineered idempotency comes from stable operation IDs, durable records, request hashes, atomic reservation, and unique constraints.

HTTP method behavior helps, but it is not enough for payment creation, order submission, webhook processing, background jobs, and message consumers. Those workflows need explicit idempotency design.

The practical goal is simple: clients, brokers, and operators should be able to retry after uncertainty without charging twice, creating duplicate orders, sending duplicate irreversible actions, or corrupting state.