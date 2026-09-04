Producer-Consumer Pattern
High Priority
7 min read
Updated June 5, 2026

Listen to this chapter
Unlock Audio

The producer-consumer pattern is one of the most common coordination problems in concurrent systems.

One or more producers generate work (events, messages, tasks) and place it into a shared buffer or queue, while one or more consumers remove that work and process it. The challenge is doing this safely and efficiently when producers and consumers run at different speeds.

In this chapter, we'll explore how producer-consumer works, implement it with proper synchronization, and understand the subtle issues around shutdown and backpressure.


What is Producer-Consumer?
Real-World Analogy

Imagine a bakery with a display shelf.

The bakers (producers) keep baking pastries and placing them on the shelf.
Customers (consumers) pick pastries from the shelf and take them away.
The shelf has limited space (bounded buffer).
If the shelf is full, bakers cannot place more pastries and have to wait until customers take some. If the shelf is empty, customers cannot take anything and have to wait until new pastries arrive.

That’s how the producer-consumer pattern works: producers generate items, consumers process them, and the shared buffer smooths out differences in speed.




Consumers

Producers

Bounded Buffer (capacity=5)

Item

Item

Item

Empty

Empty

Producer 1

Producer 2

Consumer 1

Consumer 2

In programming terms, producers and consumers are threads. The buffer is a thread-safe queue with a fixed capacity. Producers put items into the queue. Consumers take items from the queue. When the queue is full, producers block. When the queue is empty, consumers block. This blocking provides automatic flow control without explicit coordination.

The key properties of the pattern:

Decoupling: Producers and consumers don't know about each other
Buffering: Speed differences are absorbed by the queue
Backpressure: Full queue slows down producers naturally
Scalability: Multiple producers and consumers can work concurrently
The Problems Producer-Consumer Solves
In any pipeline, components have different speeds. A web scraper can fetch pages faster than a parser can process them. A sensor can generate readings faster than a network can transmit them. A user can click buttons faster than a database can record actions.

Without buffering, you have two choices:

Block the producer: The fast component waits for the slow one. You waste capacity.
Drop data: The fast component proceeds and data is lost. You lose correctness.
Producer-consumer gives you a third option: absorb temporary mismatches while maintaining both throughput and correctness.

Real Systems Using This Pattern
System	How It Uses Producer-Consumer
Apache Kafka	The entire system is producer-consumer. Producers write to partitions, consumers read from them. Partitions are the buffer.
RabbitMQ	Message queues between publishers and subscribers. Queue depth provides buffering.
Go Channels	Buffered channels are exactly this pattern. make(chan int, 100) creates a buffer of 100 items.
Java BlockingQueue	LinkedBlockingQueue, ArrayBlockingQueue implement the buffer. Thread pool task queues use this.
Unix Pipes	`cat file

Core Components
The pattern has four essential components. Each one has design decisions that affect correctness and performance.

Producer(s)
Producers generate data and put it into the buffer. They could be threads reading from a network, parsing files, or generating synthetic data.

Key Responsibilities
Generate items at their natural pace
Put items into the buffer (blocking if full)
Handle buffer-full conditions gracefully
Signal completion when done producing
Design Decisions
Should producers block when the buffer is full, or drop items?
How do producers signal that they're done?
Should there be a timeout on blocking?



Start

Buffer full

Space available

No more items

Producing

WaitingForSpace

Done

Consumer(s)
Consumers take items from the buffer and process them. They could be threads writing to a database, sending network requests, or computing results.

Key Responsibilities
Take items from the buffer (blocking if empty)
Process each item to completion
Handle errors without dying
Detect shutdown signals and exit gracefully
Design Decisions
How do consumers know when to stop?
Should consumers process items in batches?
What happens if processing fails?
Bounded Buffer
The buffer is the core synchronization point. It must be thread-safe and support blocking operations.

Key Responsibilities
Store items in FIFO order
Block producers when full
Block consumers when empty
Support concurrent access from multiple threads
Capacity Sizing
Buffer Size	Pros	Cons
Small (10-100)	Low memory, fast backpressure	Frequent blocking under burst
Medium (100-10000)	Good burst handling	More memory, delayed backpressure
Large (10000+)	Handles huge bursts	High memory, masks overload
The right size depends on burst patterns and memory constraints. Start small and increase if you see frequent producer blocking without consumer saturation.




Bounded Buffer States

produce()

produce()

consume()

consume()

Empty
Consumers block

Partially Full
No blocking

Full
Producers block

Synchronization Mechanism
Producers and consumers need to coordinate. This is typically done with condition variables or blocking queue implementations.

Key Responsibilities
Wake consumers when items are added
Wake producers when space becomes available
Handle spurious wakeups correctly
Support shutdown signaling
The synchronization must ensure:

No item is consumed before it's produced
No item is consumed twice
No producer waits forever when there's space
No consumer waits forever when there are items

How It Works
Let's trace through the lifecycle of an item from production to consumption.

Step 1: Producer Creates Item
The producer generates or receives an item to be processed. This happens at the producer's natural pace, independent of consumers.

Step 2: Producer Attempts to Enqueue
The producer calls buffer.put(item). If the buffer has space, the item is added and the method returns immediately. If the buffer is full, the producer blocks.

Step 3: Buffer Signals Consumer
When an item is added to a previously empty buffer, a waiting consumer is notified. This is typically done via a condition variable signal.

Step 4: Consumer Wakes and Dequeues
A waiting consumer wakes up, checks that the buffer is indeed non-empty (to handle spurious wakeups), and removes the item.

Step 5: Buffer Signals Producer
If the buffer was previously full, a waiting producer is notified that space is now available.

Step 6: Consumer Processes Item
The consumer processes the item. This happens at the consumer's natural pace, independent of producers.




Consumer
Buffer
Producer
Consumer
Buffer
Producer
Waiting (buffer empty)
Buffer has 3 items
Process item1
Buffer full (capacity=5)
Blocked (buffer full)
Process item2
Unblocked
put(item1)
signal (not empty)
put(item2)
put(item3)
take()
item1
put(item4)
put(item5)
put(item6)
take()
item2
signal (space available)
put(item6)
Example Trace







12345678910111213141516171819202122232425262728293031

Implementation
Let's implement producer-consumer from scratch, then show how to use standard library implementations.


Java
Java







12345678910111213141516171819202122232425262728293031
Key Points:

Line 12-15: while loop handles spurious wakeups (never use if)
Line 17: notifyAll() wakes all waiters since both producers and consumers might be waiting
Line 21-23: Same wait pattern for consumers
synchronized ensures only one thread accesses the queue at a time
Using Standard Library Implementations
In practice, use the standard library's blocking queue implementations.


Java







12345678910111213141516171819202122232425262728293031

Practical Example
Scenario: Log Processing Pipeline
You're building a centralized logging system. Multiple application servers generate logs. The logs need to be parsed, enriched with metadata, and stored in Elasticsearch.

Requirements:
Handle bursts of logs without dropping data
Decouple log generation from storage (don't slow down apps)
Scale consumers independently of producers
Graceful shutdown without losing buffered logs



Processing Pipeline

Application Servers

App 1

App 2

App 3

Raw Log Buffer

Parser Workers

Parsed Buffer

Storage Workers

Elasticsearch


Java







12345678910111213141516171819202122232425262728293031

When to Use / When Not to Use
Use When	Avoid When
Producer and consumer have different speeds	Both run at exactly the same speed
You need to decouple components	Tight coupling is acceptable
You want to absorb bursts of activity	Bursts don't happen in your workload
Multiple producers or consumers are needed	Single thread is sufficient
You need graceful degradation under load	Immediate failure is preferred
Consider Instead
Direct call: When producer and consumer are fast and in sync
Async/await: When consumer is I/O-bound and you want non-blocking code
Actor model: When you need stateful consumers with complex interaction patterns
Stream processing: When you need operators like map, filter, window over data flow
