Consider an e-commerce system that uses a payment provider such as Stripe. When a payment succeeds, your system needs to mark the order as paid, send a receipt, and notify fulfillment. The catch is that you do not control when the bank confirms the payment or when a subscription invoice is paid.

One option is polling: repeatedly ask the provider for new events.




loop
[Every N seconds]
Payment succeeds
Latency = up to one poll interval
GET /payments?since=...
No new events
GET /payments?since=...
payment_intent.succeeded
Your System
Payment Provider
Your System
Payment Provider




6 / 6
Polling wastes work when most checks return "nothing changed." It also adds delay because your system learns about the event only on the next check.

A webhook flips the direction. Instead of your system repeatedly asking for updates, the provider sends an HTTP request to your system when an event occurs.

This chapter covers what a webhook is, how delivery works, what a webhook request contains, how to build a safe receiver, and how to process webhooks reliably in production.


1. What Is a Webhook?
A webhook is an HTTP request one system sends to another when something happens.

The sending system is usually called the provider. The receiving system exposes an endpoint and is often called the consumer or receiver.




HTTP POST
event body + signature

Event occurs
e.g. payment succeeded

Provider

Receiver Endpoint

Your Application

Common examples include a payment provider sending payment_intent.succeeded, GitHub sending pull_request.opened, a CI system sending build.failed, an AI platform sending fine_tuning.job.completed, a document processing service sending extraction.completed, or a vector database sending indexing.failed.

Webhooks are a push-based integration pattern. They are useful when your system needs to react to a change that happens in another system.

Webhooks do not make networks reliable. A webhook is still an HTTP request crossing the internet. It can be delayed, sent more than once, arrive out of order, or fail until the provider retries it. Good webhook design starts with those assumptions.


2. How Webhooks Work
Webhooks, Register and Deliver
Subscriber registers a callback URL. Later, the publisher fires an event and POSTs a signed payload to that URL.

Register & Deliver
Retry Backoff
Signature

10 steps ready
plain HTTP, one POST per eventdeliveries
←
acks and registration
Service A
publisher
webhook table
no hooks stored
Service B
app.subscriber.example.com/webhooks/incoming
verify and log
no events yet
delivery timeline
ABBA
Delivery request
request not started
Response
awaiting response
Press Play to walk through the scenario.
hooks stored
0
events fired
0
delivered
0
Idle, press Play to begin



0 / 10
1x

At a high level, webhook delivery has four steps: registration, event creation, delivery, and confirmation. In plain terms: you give the provider a URL, something happens, the provider calls that URL, and your system confirms that it received the event.

Example: GitHub Webhook to Your App
Consider an internal developer platform. When someone opens a pull request in GitHub, the platform should start checks, post a Slack notification, and update a deployment dashboard.

Here is the flow.

Step 1: Register an Endpoint
You configure a webhook in GitHub:

Endpoint URL: https://platform.example.com/webhooks/github
Events: pull_request, push, issue_comment
Secret: a strong random value shared between GitHub and your receiver
The endpoint must be reachable by the provider. For public SaaS providers, that usually means a public HTTPS URL. For private integrations, it may mean a relay, a private network path, or an agent inside your network that sends events out.

Step 2: The Provider Records an Event
When a pull request is opened, GitHub records an event. The event usually includes the event type, the action, a delivery ID, repository details, user details, and either the resource data or a reference you can use to fetch it.

Step 3: The Provider Sends an HTTP Request
The provider sends a POST request to your endpoint. The request usually contains a JSON body and provider-specific headers.

The provider may also sign the request body with a shared secret. Your receiver checks that signature to confirm the request really came from the provider and was not changed on the way.

Step 4: Your Receiver Acknowledges Receipt
Your receiver should do only the minimum required work before responding: read the raw request body, verify the signature, validate the event type, save the event or put it on a queue, and return a 2xx response.

Heavy work should happen in the background. Do not call half a dozen other services while the provider is waiting for the webhook response.


3. Anatomy of a Webhook Request
Most webhooks use POST because event data belongs in the request body. Some systems support other methods, but POST is the standard shape for webhook delivery.

A webhook request usually has three parts: the endpoint URL, request headers, and request body.

Request Headers
Headers carry delivery details and security information. The exact names vary by provider.

Common headers include:

Header	Purpose
Content-Type	Usually application/json
Event type header	Event name, such as pull_request or payment_intent.succeeded
Delivery ID	Unique ID for this delivery attempt or event
Signature	HMAC or similar signature used to verify who sent the request
Timestamp	Helps detect replayed requests when used with a signature
User agent	Identifies the sender, such as GitHub or Stripe
Do not build your design around generic header names like X-Event-Type. Real providers use their own names. GitHub uses headers such as X-GitHub-Event, X-GitHub-Delivery, and X-Hub-Signature-256. Stripe uses Stripe-Signature.

Request Body
The body usually contains an event envelope and resource data.

The envelope tells you what happened. The resource data gives details about the object involved.

Example: GitHub Pull Request Event
GitHub sends a body like this when a pull request opens. The action is near the top, and the pull request details are nested under it.

Json







12345678910111213141516171819
Example: AI Job Completed Event
A batch inference provider might report a finished job with a body like this. It includes the job status and the output file the work produced.

Json







12345678910111213
This style is common in modern AI systems. Long-running work such as batch inference, file parsing, fine-tuning, data labeling, and vector indexing often finishes after the original request has returned. A webhook lets the provider tell your system when the job is done or has failed.


4. Building a Webhook Receiver
A webhook receiver looks simple from the outside: accept a request and return a response.

In production, the hard parts are reliability and trust. Events can be duplicated, arrive out of order, or be retried after a failed delivery. Attackers can send fake requests, other services may be slow or down, and your receiver may restart in the middle of a delivery.

Dedicated Endpoint
Expose a narrow endpoint for each provider or integration:

Plaintext







123
The endpoint should accept only the expected HTTP method and content types, use HTTPS in production, set request size limits, avoid browser-form security rules such as CSRF checks, and keep each provider's parsing and verification separate.

Provider-specific endpoints are usually easier to run than one generic /webhook route because each provider has different signature rules, headers, body shapes, retry behavior, and event names.

Verify Before Parsing Business Data
Verify the webhook before you trust the body.

Most providers use HMAC signatures. The provider creates a signature from the raw request body and a shared secret. Your receiver creates the same signature and compares it with the signature header.

A few details matter. Verify the raw request body, not JSON that your code has parsed and printed again. Use constant-time comparison so timing differences do not leak the secret. Include the timestamp when the provider supports it, and reject old timestamps to reduce replay risk. Store secrets in a secrets manager and support rotating them without downtime.


This example shows the shape of the check. In real code, use the provider's official library when one exists. Signature formats vary: Stripe signs a timestamped body and sends Stripe-Signature: t=..,v1=..; GitHub prefixes its hash with sha256= in X-Hub-Signature-256; others use different formats and may keep multiple active secrets during rotation.

Make Processing Safe to Repeat
Most webhook providers use at-least-once delivery. That means your receiver must assume duplicates.

Do not treat "we returned 200 last time" as proof that the business operation ran exactly once. The response may have been lost. The provider may redeliver manually. Operators may replay old events during recovery.

Use a stable key to recognize repeats, such as the provider event ID (evt_123), the provider delivery ID, or a resource ID plus event type. Store processed IDs in reliable storage with a unique constraint.

Sql







12345678910
When a duplicate arrives, return success if the original event was already accepted.


This does not create true exactly-once delivery. It makes repeated deliveries safe, which is the practical goal.

Do Not Depend on Delivery Order
Providers often do not guarantee event order. Even when events are created in the right order, retries can make them arrive in a different order.

For example, retries or network delays may cause your receiver to see events in an unexpected sequence:

Plaintext







123
Even when the provider creates these events in the correct order, the order you see can differ. Your handler should not assume that created always arrives before paid.

A few strategies help here. Before applying an important change, fetch the latest resource state from the provider instead of trusting the event alone. Use resource versions or timestamps when the provider exposes them.

Make state changes explicit so the handler can reject an invalid step backward instead of silently applying it. Also run periodic checks against the provider to find records that look missed or inconsistent.

For payments, the webhook is often best treated as a signal that says, "something changed." The final truth may still need to be fetched from the provider API before fulfilling an order or granting access.

Return the Right Status Code
Provider retry rules differ, but the broad pattern is simple: 2xx means accepted. Non-2xx usually means failed and may be retried.

Use status codes deliberately:

Response	Meaning
200 OK or 204 No Content	Event was accepted or already handled
400 Bad Request	Request body is malformed or unsupported
401 Unauthorized / 403 Forbidden	Signature or authorization check failed
404 Not Found	Endpoint is wrong or no longer exists
429 Too Many Requests	Receiver is overloaded; treated as a retryable failure by most providers
500 / 503	Temporary receiver failure; provider should retry
Do not return 200 OK before the event is safely saved or queued. If your process crashes after returning success but before saving the event, the provider has no way to know it was lost and will not retry.


5. Webhook Security
Webhook endpoints are public attack surfaces. Treat them like internet-facing APIs that anyone can hit until you have verified the request.

Verify Signatures
Signature verification is the main defense against fake webhooks.

Without it, anyone who discovers your endpoint could send fake events such as payment_succeeded, subscription_active, admin_user_created, or batch.completed. That can lead to free access, bad financial records, leaked data, or unauthorized workflows.

Protect Against Replay
A valid webhook can still be sent again later by an attacker or by an operator using redelivery tools.

Reject old timestamps when supported, detect duplicates by event or delivery ID, record received and processed events, and make follow-up actions safe to repeat.

Replay protection is especially important for events that grant access, move money, send email, or trigger external jobs.

Use IP Allow Lists Carefully
Some providers publish IP ranges for webhook delivery. Allow lists can reduce noise and block obvious spoofing attempts.

They should not replace signature verification.

IP ranges change, traffic may come through relays, and private network paths can be reconfigured. If you use allow lists, automate updates and monitor rejected traffic.

Avoid Sensitive Data Leaks
Be disciplined with logging and responses. Do not log secrets, tokens, authorization headers, or full payment bodies. Do not put secrets in webhook URLs, and never return stack traces or internal error messages to the provider. Redact personal data before sending logs to third-party tools, and keep test and production webhook secrets separate.


6. Designing Scalable Webhook Infrastructure
A simple synchronous handler can work at low volume. It will struggle when traffic spikes, other services slow down, or multiple providers retry at the same time.

A production webhook pipeline separates receiving from processing.




HTTP POST

failed after retries

failed after retries

Provider

Receiver Endpoint
verify signature
detect duplicates + save

Event Store

Queue / Outbox

Worker

Worker

Other Services
DB writes, notifications,
billing, fulfillment

Dead Letter Queue

Keep Receiving Fast
The receiver endpoint should verify the signature, validate the basic event shape, detect duplicates or reserve the event ID, save the raw event, enqueue a processing job in the same database transaction when possible, and return a 2xx response.

This keeps the provider-facing path short and predictable. The save-and-enqueue boundary matters: saving to the database and sending to a separate queue as two independent writes can create a dual-write bug. If the process crashes between those writes, the event may be dropped or processed twice.

Either commit both together, often with an outbox table that a relay forwards to the queue, or skip the queue and let workers poll the events table.

For queues, common choices include:

SQS for managed queueing that is easier to run
RabbitMQ for flexible routing and traditional work queues
Kafka for high-volume event streams and replay
Cloud Pub/Sub or Azure Service Bus for managed queueing on those clouds
The queue choice matters less than the boundary: the webhook endpoint should not be responsible for expensive business processing.

Some providers can also deliver events directly to managed event buses such as Amazon EventBridge or Azure Event Grid. That can reduce the work of running a public endpoint, but the same design rules still apply: verify trust, save important events, make processing safe to repeat, and monitor failures.

Store Events for Audit and Replay
Save incoming events before processing them.

For each event, record the provider, event ID, event type, and delivery ID when it is separate from the event ID. Keep the headers you need for debugging, along with the raw or redacted body and the result of signature verification. Track the status and timestamps for received, queued, processed, failed, and replayed events.

This event store gives you a practical history of what happened. It lets you answer questions like:

Did we receive the provider event?
Did signature verification fail?
Did we put it on the queue?
Which worker processed it?
Which follow-up call failed?
Can we replay it safely?
For sensitive domains, store the raw body only if you have a clear policy for how long to keep it and what to redact.

Process with Workers
Workers perform the actual business logic: fetch the latest provider state when needed, apply safe state changes, update internal databases, call other services, trigger notifications or workflows, and mark the event as processed or failed.

Workers let you control how much work runs at once. That matters for expensive workloads such as AI batch output processing, embedding generation, document parsing, and analytics fan-out. You can add more workers, apply rate limits per provider, and pause a bad event type without taking down the public webhook endpoint.

Retry with Backoff and Jitter
Some failures are temporary: a database failover, provider API timeout, another service being deployed, rate limiting, or a short network issue. Retry these more slowly over time, and add a little randomness so every retry does not happen at once.

Do not retry all errors the same way. A malformed body or unsupported event type will not become valid after 10 attempts. Classify failures as retryable or final.

Good retry design includes a maximum attempt count, slower retries with randomness, rate limits per provider and tenant, visibility into the next retry time, worker logic that is safe to repeat, and manual replay tools for operators.

Use a Dead Letter Queue
After repeated failures, move the event to a dead letter queue or failed-event table.

A DLQ is a place to investigate failed events, not a place to throw them away.

For each DLQ event, operators should be able to see:

Provider and event type
Body or redacted body
Error message and stack trace
Attempt count
First failure time and last failure time
Related internal resource IDs
After the underlying issue is fixed, replay the event through the same safe-to-repeat processing path.

Add Observability
Webhook failures are easy to miss. The provider may retry quietly, and your users may notice only the symptom: an order stuck in PENDING, a fine-tuning job never marked complete, or an invoice email not sent.

Track metrics at each stage:

Events received by provider and event type
Signature verification failures
Accepted vs rejected events
Queue depth and oldest event age
Processing delay
Retry count
DLQ count
Follow-up API failure rate
End-to-end time from provider event creation to local processing
Useful alerts:

Sudden drop to zero events from an active provider
Spike in signature failures
Growing queue backlog
Increasing event age
DLQ count above zero for critical event types
High webhook endpoint delay or non-2xx rate
Logs should include correlation IDs, provider event IDs, and internal resource IDs. Traces are useful when webhook processing calls several services.


7. Common Mistakes
Avoid these mistakes in production systems:

Doing too much work in the HTTP handler: This increases timeout risk and makes provider retries more likely.
Skipping signature verification: A public endpoint without verification is an open workflow trigger.
Parsing JSON before preserving the raw body: Many signature checks require the exact bytes that were sent.
Assuming exactly-once delivery: Webhooks are commonly delivered at least once. Design for duplicates.
Assuming event order: Retries and provider internals can change delivery order.
Returning success before saving the event: A crash after 200 OK can lose the event permanently.
Subscribing to every event type: Extra events add load, cost, and noise.
No reconciliation job: Webhooks are delivery signals. Critical systems still need a way to compare local state with provider state.

8. When to Use Webhooks
Use webhooks when:

Another system owns the event
Your system needs to react soon after the event occurs
Polling would add unnecessary delay or cost
The receiver can expose or operate a reliable endpoint
You can handle retries, duplicates, and delayed delivery
Avoid webhooks as the only mechanism when:

The receiver cannot be reached reliably
Strict ordering is required
The consumer must control the processing rate tightly
Events are critical and there is no replay or reconciliation path
The provider cannot sign requests or identify events reliably
In many production systems, webhooks and polling are used together. The webhook gives fast notification. A reconciliation job occasionally polls the provider to catch missed, delayed, or inconsistent events.


Summary
Webhooks let systems notify each other about events without constant polling. The simple version is just an HTTP request when something happens, but the production version needs care. The receiver should verify the sender, save the event before acknowledging it, and process it in the background.

It should assume duplicates, avoid depending on ordering, retry safely, reconcile critical state separately, and make the whole path easy to monitor.

Handled this way, webhooks become a reliable boundary for payments, developer workflows, SaaS automation, AI jobs, and many other systems that need to react to outside events.