Livelock
High Priority
11 min read
Updated June 5, 2026

Listen to this chapter
Unlock Audio

A livelock happens when threads are not blocked, but the system still makes no progress. Instead of waiting, the threads keep reacting to each other in a way that prevents either from completing work. CPU usage can be high, logs can be noisy, and yet nothing useful finishes.

Real-World Analogy

Two people meet in a narrow hallway. Both step aside to let the other pass. They step in the same direction. They step aside again, same direction. And again.

Both are actively trying to be polite, both are moving, but neither makes any progress down the hallway.

In many ways, livelock is more insidious than deadlock. With deadlock, you notice immediately because everything stops. With livelock, the system looks busy. Threads are executing, network requests are flying, logs are filling up with retry messages. It might take hours before someone realizes that despite all this activity, no actual work is completing.


What is Livelock?
Livelock occurs when threads are not blocked but continuously change state in response to each other without making progress. Unlike deadlock where threads are stuck waiting, in livelock threads are actively running but trapped in an unproductive loop.

The key distinction:

Aspect	Deadlock	Livelock
Thread state	BLOCKED (waiting)	RUNNING (executing)
CPU usage	Near zero	High (often 100%)
Activity	None	Lots of retries, state changes
Detection	Thread dumps show blocked threads	Thread dumps show running threads
Root cause	Circular wait for locks	Overly reactive retry logic

Why Livelock Happens
Livelock typically emerges from code designed to be helpful. Retry logic, conflict resolution, and polite backoff can all cause livelock when they interact badly.

1. Overly Polite Algorithms
Both participants back off under the same conditions, using the same strategy.








12345
The problem: both threads are too accommodating. Neither asserts priority or breaks the symmetry.

2. Retry Without Randomization
When multiple components retry at fixed intervals, they stay synchronized.








123456789
Without randomization, the retry storm repeats forever at the same interval.

3. Collision Resolution Without Jitter
Network protocols like Ethernet's CSMA/CD and distributed systems often use backoff after collisions. Without randomness, collisions keep happening.








123
4. Thundering Herd
When many threads wait for a resource and all wake up simultaneously when it becomes available, they all compete, most fail, and all go back to waiting. The cycle repeats.








12345

How to Detect Livelock
Livelock is harder to detect than deadlock because the system looks active. Here are the symptoms:

Symptoms
High CPU with no throughput: CPU pegged at 100%, but requests per second is zero or near zero. Work is being done, but it's all overhead.

Retry logs exploding: Log files fill with retry messages. You see patterns like "Retry attempt 1... Retry attempt 2... Retry attempt 3..." repeating infinitely.

Metrics show activity but no completion: Queue depth grows, in-flight requests increase, but completed requests flatline.

Patterns in timing: If you plot retry attempts, you might see synchronization: many retries at exactly the same timestamps.

Detection Approach
Compare CPU usage vs. throughput: High CPU with low throughput is suspicious.
Check retry counts: If retries are unbounded and growing, investigate.
Analyze thread dumps: Unlike deadlock (BLOCKED threads), livelock shows RUNNABLE threads all executing similar retry logic.
Profile hotspots: Profilers will show all time spent in retry/backoff code, not in actual business logic.

Java







123456789101112131415161718192021

Backoff Strategies
The key to preventing livelock is breaking synchronization between competing threads. Backoff strategies determine how long to wait before retrying.

Linear Backoff







12345
Problem: If two threads start at the same time, they stay synchronized. Thread A waits 100ms, Thread B waits 100ms. Both retry at the same time. Both wait 200ms. Same problem.

Exponential Backoff







12345
Problem: Slightly better because waits grow faster, but still synchronized if threads start together. Also, waits can grow very large very quickly.

Exponential Backoff with Jitter (Recommended)







12345
Why it works: The randomness breaks synchronization. Even if two threads start at exactly the same time and fail at exactly the same time, their random waits will differ. One will retry first and likely succeed before the other interferes.




Exponential + Jitter

random(0-100ms)

random(0-200ms)

random(0-400ms)

Exponential Backoff

100ms

200ms

400ms

Linear Backoff

100ms

200ms

300ms

Stays synchronized

Still synchronized

Breaks synchronization

Jitter Variations
There are several ways to add jitter:

Full Jitter: wait = random(0, base * 2^attempt)

Most aggressive randomization. Wait time is completely random within the range.

Equal Jitter: wait = base * 2^attempt / 2 + random(0, base * 2^attempt / 2)

Half deterministic, half random. Provides a minimum wait while still randomizing.

Decorrelated Jitter: wait = min(cap, random(base, previous_wait * 3))

Each wait depends on the previous wait, creating more variation over time.


Java







1234567891011121314151617181920212223242526272829

How to Prevent Livelock
Strategy 1: Add Randomization to Backoff
The simplest fix: add random jitter to all retry delays.


Java







12345678910111213141516171819202122232425262728293031
Strategy 2: Asymmetric Behavior
Give different threads different roles or priorities. Not everyone should back off equally.


Java







12345678910111213141516171819202122232425262728293031
Threads with higher priority back off less, making them more likely to acquire locks first. This breaks the symmetry that causes livelock.

Strategy 3: Limit Retry Attempts
Don't retry forever. Set a maximum number of retries and fail gracefully.

Java







12345678910111213141516171819
Strategy 4: Circuit Breaker Pattern
If operations keep failing, stop trying for a while. Let the system recover.




Normal operation

Failures exceed threshold

Timeout expires

Probe succeeds

Probe fails

Closed

Open

HalfOpen

When the circuit is Open, all operations fail immediately without trying. This gives the system time to recover and prevents retry storms from making things worse.


Livelock in Practice
The Bug: Synchronized Retry Loop

Java







12345678910111213141516171819202122232425262728293031
The Fix: Random Backoff

Java







12345678910111213141516171819202122232425262728293031
