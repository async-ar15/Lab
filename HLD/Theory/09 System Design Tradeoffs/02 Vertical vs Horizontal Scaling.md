Scaling starts with a bottleneck.

Maybe the CPU is maxed out. Maybe memory is full. Maybe disk, network, database locks, GPU memory, or a shared service is the real limit.

Once you know what is limiting the system, there are two basic ways to add capacity.

Vertical scaling makes an existing machine or runtime bigger: more CPU, more memory, faster storage, or a larger instance type.

Horizontal scaling adds more machines, containers, processes, or nodes and spreads work across them.




Measured bottleneck
CPU, memory, I/O, network, GPU, database

Vertical scaling
bigger instance or larger resource limits

Horizontal scaling
more instances, workers, or nodes

Same basic shape
more capacity per unit

More units
load spread across them

Neither option is always better. The right choice depends on where the load is, how much state the component owns, how quickly demand changes, and what kind of failure the business can tolerate.

Vertical scaling is simpler but has a hard upper limit. Horizontal scaling can go much further, but it adds coordination, data splitting, and more moving parts.

Most production systems use both.

This chapter covers vertical and horizontal scaling, their trade-offs, and how systems use both.


1. Vertical Scaling
Vertical scaling, or scaling up, means giving an existing unit more resources.

That unit might be a physical server, cloud VM, database instance, Kubernetes pod, cache node, or GPU worker.

The idea is the same in every case: keep the system shape mostly unchanged, but give one unit more room to work.




After

Before

scale up

Service or database
8 vCPU
32 GB RAM
standard disk

Same service or database
32 vCPU
256 GB RAM
faster storage

Common examples include moving a database from 8 vCPU and 32 GB RAM to 32 vCPU and 256 GB RAM, or adding memory so the hot data fits in cache.

A write-heavy database might benefit from faster NVMe storage or higher disk I/O. A Kubernetes workload may need higher CPU and memory limits.

An inference service might move to a larger GPU instance because the model no longer fits comfortably in memory.

Vertical scaling is often the first practical move because it avoids redesigning the application.

Pros of Vertical Scaling
Simple operating model, since a larger node needs fewer application changes than splitting work across nodes
Keeps related data and compute close together, which helps databases, caches, search indexes, and in-memory analytics
Less coordination, avoiding distributed locks, cross-node joins, replica lag, and complex request routing
A larger machine can buy time quickly for a genuinely CPU-bound or memory-bound service
A natural fit for stateful systems, which are usually easier to scale up before they are scaled out
Cons of Vertical Scaling
A hard ceiling, since every platform has a largest practical instance size
Poor failure isolation, since one large node owning the whole workload can take down a lot when it fails
Cost often jumps sharply for very large machines, high-memory instances, faster I/O, and GPU instances
Upgrades often need a process restart, VM replacement, pod restart, or database failover
More hardware does not fix every bottleneck, such as a slow external API, hot database row, missing index, or code path that only runs one thing at a time
Vertical scaling is not a beginner-only strategy. Many serious production systems run for years on larger databases or search clusters because the simpler design is more reliable than spreading things out too early.


2. Horizontal Scaling
Horizontal scaling, or scaling out, means adding more units and spreading work across them.

For an application tier, this usually means running more instances behind a load balancer. For workers, it means adding more consumers to a queue.

For storage, it may mean replicas, partitions, or shards. For AI systems, it may mean more model-serving replicas, more embedding workers, or more vector database nodes.




Clients

Load balancer

App instance 1

App instance 2

App instance 3

App instance N

Shared database

Shared cache / object store

Horizontal scaling works best when each unit can handle work independently.

A stateless HTTP service is a clean example. Any instance can serve any request, and shared state lives outside the instance in a database, cache, object store, or token.

Add more instances and the load balancer has more places to send traffic.

Stateful systems are harder. If data is tied to a specific node, the system needs routing, replication, agreement between nodes, rebalancing, and recovery logic.

Pros of Horizontal Scaling
A higher capacity ceiling, adding nodes as demand grows when shared systems can handle the extra traffic
Better availability, since other nodes keep serving when one fails
Easier scaling up and down on cloud platforms, based on load, queue length, or custom metrics
Better failure isolation from smaller units, plus placement closer to users or data when delay, compliance, or data location rules require it
Cons of Horizontal Scaling
More complexity from running many nodes: partial failure, retries, timeouts, load balancing, coordinated deploys, and monitoring
Shared dependencies become a risk, since more application servers can overload the database, cache, message broker, or another service they call
Data consistency gets harder, as replication, sharding, and caching introduce stale reads, conflicts, and edge cases
Slowest-dependency delay appears when one request calls many services, because the slowest call holds up the whole request
New nodes are not instant, needing time to start, load models, fill caches, establish connections, or join clusters
Horizontal scaling can add a lot of capacity, but that capacity is not free.

If the database primary can handle 20,000 writes per second, adding 50 more web servers does not raise that limit. It may only make the database fail faster.


3. Scaling Depends on the Layer
Different components scale differently. A good design does not apply one rule everywhere.

Stateless API servers and background workers usually scale out cleanly, as long as sessions, files, and changing state live outside the instance. Workers should also be safe to retry, because failed jobs often run again.

The number of waiting jobs is a useful scaling signal for worker systems. If the queue keeps growing, you may need more workers.

Relational databases are a different story. Read-only copies can help with read traffic, but write traffic usually needs harder changes such as partitioning or sharding.

Caches sit in a similar middle ground. Copying or splitting cache data can help, but one very popular key can still overload one node.

Search indexes and vector databases can both scale horizontally, but there are costs. Shards add capacity, but they also make rebalancing and ranking harder. A single query may also need to ask many shards before it can return an answer.

Vector databases gain storage and more work per second from extra nodes, but recall, indexing cost, and data placement become important design choices.

Model inference often scales by running more copies of the model. Large models may also need bigger machines, batching, model compression, or splitting one model across multiple GPUs.

This is why "just scale horizontally" is incomplete advice. The web tier may scale out easily while the database, embedding pipeline, or GPU inference tier becomes the real limit.


4. When to Choose Vertical vs Horizontal Scaling
Start with measurement. Look at resource use, what is maxed out, p95 and p99 response time, queue length, error rates, database waits, and cost per request. Scaling before measuring often hides the real issue.




yes

no

yes

no

yes

no

yes

no

Measure the bottleneck

Is one node's resource maxed out?

Does the component own hard-to-split state?

Can requests or jobs run independently?

Do you need failure isolation or traffic bursts?

Fix dependency, query, lock, retry, or external limit

Prefer vertical scaling first

Prefer horizontal scaling

Choose Vertical Scaling When
Vertical scaling is usually a good first move when one machine is the clear bottleneck. For example, CPU, memory, disk I/O, network bandwidth, or GPU memory may be maxed out on that machine.

It also fits stateful workloads. Databases, caches, and search nodes often benefit from larger machines before they benefit from being split across many machines.

When the traffic forecast does not justify sharding, multi-node coordination, or more things for the team to run, vertical scaling keeps things simple.

There are two more situations where scaling up wins. If the hot data almost fits, more RAM can keep it in memory and avoid expensive disk reads.

If the code is difficult to distribute, as with legacy systems, monoliths, or tightly coupled services, a larger machine may be the only safe option until the architecture catches up.

For example, a PostgreSQL primary may be slow because the hot indexes no longer fit in memory. Moving to a larger instance can be the right next step while you optimize queries and plan longer-term partitioning.

Choose Horizontal Scaling When
Horizontal scaling is usually the better direction when the service must survive machine failures. One node should not take down the whole service.

It is also a natural fit when traffic comes in bursts, because more instances can absorb spikes. This works especially well for stateless services and queue workers.

Workloads with independent requests, jobs, tenants, files, embeddings, or messages can often be split across workers without much coordination.

Scaling out also becomes necessary when one machine is near its practical limit and the next larger machine is unavailable, too expensive, or still not enough.

Serving users from nearby regions is another driver, especially when users or data are spread around the world and delay or data-location rules require local serving.

For example, an API service is CPU-bound during peak traffic, stores no local session state, and depends on a database that still has extra room. Adding more API instances behind a load balancer is the natural move.


5. Combining Vertical and Horizontal Scaling
Most production systems combine both.

A horizontally scaled service still needs each node to be sized well. A database shard may still need to be a large machine.

A model-serving cluster may run several replicas, each on a GPU instance large enough to hold the model and serve batches efficiently.




Horizontal application tier

Traffic

Load balancer

Well-sized app node

Well-sized app node

Well-sized app node

Larger database primary

Read replica

Queue

Worker replica

Worker replica

A common combination is a bigger database paired with more application servers, where the stateless tier scales out while the database stays on a larger primary.

Read-heavy systems often add read-only copies on top of a larger primary database. Reads move to the copies while writes stay on the stronger primary.

Sharded databases follow a similar mixed pattern: split data across shards, but size each shard for its hot data and write load.

The same logic applies to background work and AI workloads. Queue workers scale out to do more work per second, and each worker instance grows when individual jobs need more CPU, memory, or GPU capacity.

Inference systems add more model-serving instances to survive failures and do more work per second. Then each instance is tuned for GPU memory, batch size, and delay targets, often with batching.

Automatic scaling does not remove the need for design. Horizontal scaling can add instances when metrics rise, and vertical scaling can adjust CPU and memory settings or recommend larger sizes.

Both still depend on good metrics, startup time, shared-dependency capacity, and safe deploy behavior.


6. Practical Decision Framework
Use this order when deciding how to scale:

Find the bottleneck. Use metrics and traces, not guesses.
Remove obvious waste. Fix slow queries, missing indexes, runaway retries, huge request or response bodies, and inefficient code.
Scale vertically when it buys simple extra room. This is often the fastest safe move for stateful components.
Scale horizontally when you need to survive failures, handle bursts, or add more total capacity. Make the service stateless or split state deliberately.
Protect shared dependencies. Add connection limits, backpressure, rate limits, queues, and load shedding where needed. In plain terms, slow down, queue, or reject extra work before one overloaded dependency takes down everything.
Re-test under realistic load. The bottleneck will move after every meaningful scaling change.
The best scaling strategy is the one that increases capacity without creating a system the team cannot safely run.

Vertical scaling gives you simplicity and keeps related data and compute close together. Horizontal scaling helps you handle bursts and survive failures. Mature systems use both, but they apply each one where it matches the actual workload.


Summary
Scaling starts from a measured bottleneck, whether that is CPU, memory, disk I/O, network, GPU memory, or an overloaded shared dependency. Vertical scaling makes an existing machine or runtime bigger. Horizontal scaling adds more machines, containers, or nodes.

Vertical scaling is simple and often the fastest safe move for stateful components, but a single machine has a ceiling and can still be a single point of failure.

Horizontal scaling helps with bursts and failures, but it usually requires stateless services or carefully split state. It also requires protecting shared dependencies with connection limits, queues, rate limits, backpressure, or load shedding.

The practical order is to find the bottleneck with metrics, remove obvious waste first, scale vertically for simple extra room, and scale horizontally when failure handling or total capacity demands it. The bottleneck moves after every meaningful change, so re-test under realistic load.

Mature systems use both and apply each where it matches the workload.