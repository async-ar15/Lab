# Synchronization Primitives - Theory Super

## Global Mind Map: Synchronization Primitives

```mermaid
graph TB
    SP[Synchronization Primitives] --> EXCLUSION[Mutual Exclusion]
    SP --> COORDINATION[Coordination & Limits]
    SP --> ADV_LOCKING[Advanced Locking Patterns]
    SP --> LOCK_FREE[Lock-Free (Hardware)]

    EXCLUSION --> MUTEX[Mutex]
    EXCLUSION --> GRANULARITY[Lock Granularity]
    
    COORDINATION --> SEMAPHORE[Semaphores]
    COORDINATION --> CV[Condition Variables]
    
    ADV_LOCKING --> REENTRANT[Reentrant Locks]
    ADV_LOCKING --> TRY_LOCK[Try-Lock / Timed]
    
    LOCK_FREE --> CAS[Compare-and-Swap]

    MUTEX -.->|Only one thread at a time| EXCLUSION
    GRANULARITY -.->|Coarse vs Striped vs Fine| EXCLUSION
    SEMAPHORE -.->|N threads at a time| COORDINATION
    CV -.->|Wait for custom predicate| COORDINATION
    REENTRANT -.->|Same thread acquires safely| ADV_LOCKING
    TRY_LOCK -.->|Non-blocking acquisition| ADV_LOCKING
    CAS -.->|Atomic hardware instruction| LOCK_FREE
```

---

# 1. Mutex (Mutual Exclusion)

## The Problem - Lost Updates
If two threads both run `counter++`, they can simultaneously read `0`, calculate `1`, and write `1`. The second update is lost because the threads overwrote each other without knowing.

## The Core Idea
A **Mutex** is the most fundamental synchronization primitive. It ensures that only *one* thread can access a critical section of code at a time. The thread that locks it must be the one to unlock it.

## How It Works
1. **Acquire**: Thread checks if mutex is free. If yes, it locks it and proceeds. If no, the thread blocks (sleeps in a wait queue).
2. **Execute**: The thread runs the critical section safely.
3. **Release**: The thread unlocks the mutex and wakes up one waiting thread.

### Tradeoffs
- **Pros**: Safe, simple, guarantees no race conditions.
- **Cons**: High overhead under contention. If 100 threads hit the same mutex, 99 of them go to sleep, serializing your multi-core system into a single-core bottleneck.

---

# 2. Semaphores

## The Problem - Resource Pools
A mutex only allows *one* thread. What if you have a database connection pool of 20 connections? You want to allow up to 20 threads to work concurrently, and block the 21st. A mutex can't do this.

## The Core Idea
A **Semaphore** maintains a counter of available "permits". It answers the question: "How many threads are allowed to proceed right now?"

## How It Works
1. **Acquire (P/wait)**: Decrements the permit counter. If counter > 0, proceed. If counter == 0, block.
2. **Release (V/signal)**: Increments the counter and wakes up one waiting thread.

### Binary vs Counting
- **Binary (Permits = 1)**: Looks like a mutex, but any thread can release it. Perfect for signaling between threads (e.g., Producer signals Consumer).
- **Counting (Permits = N)**: Perfect for rate limiting, connection pools, and bounded buffers.

> **Crucial Difference from Mutex**: A mutex has "ownership" (the locking thread must unlock). A semaphore does not (Thread A can acquire, Thread B can release).

---

# 3. Condition Variables

## The Problem - Busy Waiting
If a consumer thread needs to wait for data, it could write `while(!dataReady) {}`. This burns 100% of a CPU core doing absolutely nothing (busy-waiting).

## The Core Idea
A **Condition Variable** lets a thread go to sleep and consume zero CPU until another thread explicitly wakes it up, signaling that a specific condition has changed.

## How It Works (The Wait-Notify Pattern)
A Condition Variable **must always be paired with a Mutex** to prevent the "Lost Wakeup" problem (where a signal arrives in the microsecond before a thread fully goes to sleep).

1. Thread acquires Mutex.
2. Thread checks condition in a `while` loop (e.g., `while(buffer.isEmpty())`).
3. If false, thread calls `wait()`. This **atomically releases the mutex and puts the thread to sleep**.
4. Producer adds data, calls `signal()`.
5. Consumer wakes up, **reacquires the mutex**, rechecks the `while` loop, and proceeds.

> **Signal vs Broadcast**: Use `signal()` to wake one thread (prevents the "Thundering Herd" problem). Use `broadcast()` to wake all threads when multiple threads can proceed.

---

# 4. Lock Granularity (Coarse vs Fine-Grained)

## The Problem - The Contention Bottleneck
If you protect a massive Hash Table with a single global mutex, thread A writing to bucket 0 blocks thread B writing to bucket 999. Parallelism is destroyed.

## The Core Idea
**Lock Granularity** is the spectrum of how much data a single lock protects.

## How It Works
- **Coarse-Grained**: One lock protects everything. Easy to write, impossible to deadlock, but scales horribly under load.
- **Medium (Lock Striping)**: Divide data into segments (stripes). E.g., 16 locks for 1000 buckets. Thread hashes the key to find which lock to acquire. High parallelism, reasonable memory overhead. (This is how Java's `ConcurrentHashMap` originally worked).
- **Fine-Grained**: A lock per element/node. Maximum parallelism, but uses massive memory and risks deadlocks if multiple locks are acquired out of order.

---

# 5. Reentrant Locks (Recursive Locks)

## The Problem - Self-Deadlock
If a thread holds a lock, and calls a function (or recurses) that tries to acquire that *exact same lock*, a normal mutex will block forever. The thread is waiting for itself.

## The Core Idea
A **Reentrant Lock** tracks *which* thread owns it and *how many times* they acquired it. If the owning thread asks for it again, it says yes.

## How It Works
1. Thread A acquires lock. Owner = Thread A, Count = 1.
2. Thread A recurses, acquires lock again. Owner = Thread A, Count = 2.
3. Thread A returns, calls unlock. Count = 1 (Lock is still held!).
4. Thread A calls final unlock. Count = 0. Lock is finally free for Thread B.

> **Tradeoff**: ~25% overhead compared to non-reentrant locks due to tracking the owner and count, but saves you from architectural nightmares in recursive algorithms and event callbacks.

---

# 6. Try-Lock and Timed Locking

## The Problem - Indefinite Deadlocks
If Thread A holds Lock 1 and waits for Lock 2, while Thread B holds Lock 2 and waits for Lock 1, they wait forever. Standard locks offer no escape hatch.

## The Core Idea
**Try-Lock** attempts to acquire a lock, but if it fails, it returns `false` *immediately* instead of blocking. **Timed-Lock** waits, but only for a maximum duration before giving up.

## How It Works
Instead of `lock()`, use `tryLock()`.
If it returns `false`, the thread can release any locks it currently holds, sleep for a random backoff period, and try the whole process again. This breaks the "hold and wait" condition of deadlocks.

> **Real-world anchor**: Use timed locks in web servers. If a request handler can't get a lock within 100ms, it's better to fail fast with a 503 error than to hang the user's browser indefinitely.

---

# 7. Compare-and-Swap (CAS)

## The Problem - Lock Overhead
Locks require OS context switches. If you just want to increment a counter, going to sleep and waking up via the OS is horribly slow (microseconds).

## The Core Idea
**CAS (Compare-and-Swap)** is a hardware-level CPU instruction that updates a value atomically *without locks*. It detects conflicts and retries. This is the foundation of Lock-Free programming.

## How It Works
CAS takes 3 arguments: `Location`, `ExpectedValue`, `NewValue`.
1. Read current value (e.g., `5`).
2. Calculate new value (e.g., `6`).
3. Fire the CAS instruction: "If value is still `5`, make it `6`."
4. If another thread snuck in and made it `7`, the CAS fails.
5. The thread immediately loops, reads `7`, calculates `8`, and tries again.

### The ABA Problem
Thread 1 reads `A`. Thread 2 changes `A` to `B`, then changes it back to `A`. Thread 1 fires CAS(expected: `A`). It succeeds, but the state underneath was actually corrupted (e.g., memory was freed and reallocated).
**Fix**: Use Tagged Pointers / Version numbers (`A_v1` vs `A_v3`) so the CAS detects the hidden change.
