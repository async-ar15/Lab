When an order is placed, several independent systems may need to react. Inventory may reserve stock, notifications may send a confirmation email, analytics may record the event, fraud detection may evaluate the order, and shipping may prepare fulfillment.

The order service should not have to call all of those services one by one. That would make checkout slower, and one slow service could hurt the whole order path.

Publish-subscribe, usually called pub/sub, solves this by letting a publisher announce an event to a topic. Subscribers that care about that topic receive their own copy and react independently.

The publisher does not need to know who is listening or what each subscriber will do.

This chapter covers how publish-subscribe works, how it differs from queues, and where it fits.

Pub/Sub System
Publish messages to topics and watch every subscribed service receive them independently.


Publish Message

#orders
Enter message...

Topics
Add
orders
2

notifications
2

analytics
1

Subscribers
Add
O
Order Service

#orders
#notifications
#analytics
No messages yet
E
Email Service

#orders
#notifications
#analytics
No messages yet
D
Dashboard

#orders
#notifications
#analytics
No messages yet
3 Topics
3 Subscribers
0 Messages

Fullscreen

1. What Is Pub/Sub?
Pub/sub is a messaging pattern with three core ideas:

A publisher writes an event.
A topic receives the event.
Each subscription attached to that topic gets its own stream of messages.



OrderPlaced

Order Service

orders.events

Inventory Subscription

Email Subscription

Fraud Subscription

Inventory Service

Email Service

Fraud Service

The subscription is important. In many systems, the topic is where events are published, but each subscription tracks its own delivery details: what has been acknowledged, what is waiting, what should be filtered, how retries work, and where failed messages go.


2. Queue vs Pub/Sub
Queues and pub/sub often work together, but they solve different problems.

Question	Work Queue	Pub/Sub
Who receives a message?	One worker from a pool	Every interested subscription
Main purpose	Spread work across workers	Send one event to many listeners
Producer intent	"Someone should do this task"	"This thing happened"
Consumer ownership	Workers usually share one job type	Each subscriber owns its own reaction
Example	Resize this image	User registered
A useful rule of thumb: use a work queue when one worker should do a task, and use pub/sub when something happened and several services may care.


3. Core Concepts
Pub/sub rests on a small set of building blocks. Publishers create events, topics receive them, subscriptions connect services to topics, and messages carry enough detail for subscribers to act safely.

Publisher
A publisher creates events and sends them to a topic. It should publish facts, not instructions for every other service.

Good event:

Json







1234567
Less useful event:

Json







123
The first event lets subscribers decide what to do. The second forces the publisher to know too much about other services.

Topic
A topic is a named stream or channel where events are published. Topic design should be boring and predictable, with names like orders.events, payments.events, users.events, and inventory.events.

Avoid creating a new topic for every tiny variation unless your broker and team can manage that many topics without confusion.

Subscription
A subscription connects a service to a topic. Each subscription usually has its own waiting messages, acknowledgments, retry rules, dead-letter rules, filters, and worker instances.




payments.events

ledger-sub

email-sub

Ledger Worker 1

Ledger Worker 2

Email Worker 1

Here, the ledger service and email service each receive payment events. Within ledger-sub, multiple ledger workers share the work.

Message
A pub/sub message should include enough information for subscribers to process it safely. That usually means an event ID, event type, entity ID such as orderId, timestamp, version, trace ID, the event data, and any fields used for filtering.

Subscribers should not have to guess what happened or how to handle duplicates.


4. How Pub/Sub Works
The basic flow:




Publish OrderPlaced
Publish acknowledged
Copy for Subscription A
Copy for Subscription B
Deliver message
Deliver message
ACK
ACK
Publisher
Topic
Subscription A
Subscription B
Consumer A
Consumer B
Publisher
Topic
Subscription A
Subscription B
Consumer A
Consumer B




8 / 8
Each subscription processes independently. If the email subscriber is down, the inventory subscriber can continue. If analytics falls behind, it should not block fraud detection.

That independence is the real value of pub/sub.


5. Fan-Out and Filtering
Two features make pub/sub flexible: fan-out and filtering. Fan-out sends one event to many subscriptions. Filtering lets each subscription receive only the messages it cares about. Together, they let you add or narrow subscribers without changing the publisher.

Fan-Out
Fan-out means one event is sent to multiple subscriptions.




UserRegistered

Publisher

users.events

Welcome Email

CRM Sync

Audit Log

Product Metrics

This keeps the publisher simple. New subscribers can be added without changing the publisher.

Filtering
Filtering lets a subscription receive only matching messages.




eventType = OrderPlaced

eventType = OrderCancelled

region = EU

all events

orders.events

Inventory Subscription

Refund Subscription

EU Compliance Subscription

Analytics Subscription

Prefer filtering on message attributes when your broker supports it. If every consumer has to parse a large message just to decide whether it cares, you waste work and spread filtering logic across the codebase.


6. Push vs Pull Delivery
Pub/sub systems usually deliver messages in one of two ways.

Push
The broker calls a subscriber endpoint.




HTTP push

Topic

Subscription

Subscriber Endpoint

Push is convenient for webhooks and quick delivery. The subscriber must expose a reliable endpoint and be ready for bursts.

Pull
Subscribers fetch messages when they are ready.




pull batch

messages

ack

Subscriber Worker

Subscription

Pull gives consumers more control over batch size, number of workers, and how fast they take messages.

Delivery Mode	Strength	Trade-off
Push	Simple for web-facing endpoints and quick callbacks	Slowing down is harder; endpoint must stay reachable
Pull	Consumer controls rate, batching, and worker count	Consumer must run polling workers
The decision comes down to whether the broker or the consumer should set the pace of delivery.


7. Consumer Groups and Shared Subscriptions
Pub/sub fan-out and load balancing are different ideas.

Fan-out sends a copy to each subscription. Load balancing spreads one subscription's messages across multiple worker instances.




orders.events

email-sub

analytics-sub

Email Worker 1

Email Worker 2

Analytics Worker 1

Analytics Worker 2

Analytics Worker 3

Kafka calls this a consumer group. Other systems may call it a shared subscription or competing consumers. The idea is the same: each service gets its own subscription, and that service's workers share the work.


8. Durability and Replay
Pub/sub systems differ a lot in what they store.

Some are temporary: messages are delivered only to subscribers that are connected right now. If a subscriber is offline, it misses the message.

Others are durable: messages are kept for a period of time, so subscribers can catch up from their backlog.

Mode	Behavior	Good For
Temporary	Only online subscribers receive messages	Presence, cache invalidation, low-value live updates
Durable subscription	Messages wait until the subscriber acknowledges them or the keep time expires	Business events, background processing
Replayable log	Consumers can go back to older messages while they are still kept	Data pipelines, reprocessing, stream processing
Replay is useful, but it is not free. Replaying old events can repeat work, so consumers must be safe to run the same event more than once.


9. Ordering
Most pub/sub systems do not provide one perfect order for every event. If they provide ordering, it is usually limited to a partition, message group, ordering key, or entity key.

Design for this from the start. Put related events on the same key, such as orderId or accountId. Include fields like eventId, eventTime, and version so consumers can ignore old events when a newer one has already been applied. Avoid workflows that require every event in the company to arrive in perfect order.




orders.events

key=order_1

key=order_2

Placed -> Paid -> Shipped

Placed -> Cancelled

Per-order ordering is usually realistic. Ordering every event in the whole company is not.


10. Delivery Failures
Most durable pub/sub systems use at-least-once delivery. In plain terms, a subscriber may receive the same event more than once.

Subscribers should be safe to retry. Treat short-lived failures differently from permanent failures. Retry short-lived failures more slowly, move repeatedly failing messages to a dead-letter topic after a limit, alert when backlogs grow, and avoid hammering another service that is already unhealthy.




success

transient failure

permanent failure

Subscription

Subscriber

ACK

Retry with backoff

Dead Letter Topic

Pub/sub does not remove failure handling. It moves failure handling to each subscriber.


11. Changing Events Safely
Events become contracts. Once several services consume an event, changing it casually can break production.

Prefer adding fields instead of changing or removing existing ones. Keep old fields until consumers stop using them. Include a version when it helps. For very busy or strongly typed event streams, a schema registry can help teams agree on what each event should look like. Treat event removal as a breaking change, and document who owns each event.

Avoid publishing database rows directly as events. A table is an internal storage detail, not a stable contract for other services.


12. Common Patterns
A few common designs show up when teams build on pub/sub. Some events are tiny notifications. Others carry enough data for subscribers to update themselves. The right choice depends on how much each subscriber needs to know.

Event Notification
Publish a small event that says something happened. Subscribers fetch more data if they need it.




OrderPlaced

Order Service

orders.events

Email Service

Inventory Service

Analytics Service

This is a good default when services need to react independently.

Fan-Out to Queues
Use pub/sub to send one event to many places, then give each subscriber its own queue for buffering and worker scaling.




Publisher

Event Topic

Email Queue

Inventory Queue

Analytics Queue

Email Workers

Inventory Workers

Analytics Workers

This is common with systems such as SNS to SQS. The topic sends the event to each subscriber. Each queue stores work for one subscriber.

Events That Carry State
The event contains enough state for subscribers to update themselves without calling the publisher.

Example: ProductPriceChanged includes product ID, old price, new price, currency, and version.

This reduces direct calls back to the publisher, but it makes event design more important.

Event Sourcing
Event sourcing is a separate design where the source of truth is an append-only event log. Pub/sub can distribute those events, but publishing events does not automatically mean the system is doing event sourcing.

That distinction matters. Many systems use pub/sub for notifications while still storing current state in normal databases.


13. Popular Implementations
Several systems implement pub/sub, and they differ in how they store and deliver messages. The table below gives the practical shape of each option.

System	Model	Notes
Apache Kafka	Durable partitioned log	Great for event streams, replay, consumer groups, and high-volume pipelines. Ordering is per partition.
Amazon SNS	Managed fan-out service	Publishes to SQS, Lambda, HTTP/S, email, SMS, and more. Failed delivery behavior depends on the target; DLQs can be attached to subscriptions.
Google Cloud Pub/Sub	Managed topics and subscriptions	Supports push and pull, at-least-once delivery, ack deadlines, replay while messages are retained, ordering keys, filters, and dead-letter topics.
Azure Service Bus Topics	Managed topics and subscriptions	Good for enterprise messaging with filters, sessions, scheduled delivery, and dead-letter queues.
Redis Pub/Sub	Temporary in-memory pub/sub	Very fast, but messages are not saved for offline subscribers. Use Redis Streams when persistence and consumer groups matter.
Do not choose only by brand name. Choose based on what the workload needs: saved messages, replay, filtering, ordering, volume, cloud integration, or very fast temporary delivery.


14. When to Use Pub/Sub
Use pub/sub when:

Multiple services need to react to the same event.
The publisher should not know all consumers.
Subscribers can process independently.
It is acceptable for different services to update a little later.
New consumers may be added over time.
Avoid pub/sub when:

The caller needs an immediate answer from the consumer.
There is exactly one handler and no fan-out.
The workflow needs strict step-by-step control.
Subscribers cannot tolerate duplicate or out-of-order events.
Nobody owns monitoring the subscriptions and backlogs.
For multi-step business workflows, a workflow engine may be clearer than a long chain of loosely connected events.


Summary
Pub/sub lets publishers announce events without knowing who will consume them. Each subscription gets its own copy and can process on its own.

The pattern is excellent when many services need to react to the same event. It also brings real responsibilities: duplicate delivery, backlog monitoring, safe event changes, ordering limits, retries, and dead-letter handling.

Use pub/sub for events, not as a way to hide every service call. Design events as stable contracts, make subscribers safe to retry, and choose a broker whose storage and replay behavior match the workload.