Compare-and-Swap (CAS)
High Priority
10 min read
Updated June 12, 2026

Listen to this chapter
Unlock Audio

Compare-and-Swap, or CAS, is the foundation of lock-free programming. It is a single atomic instruction provided by modern CPUs that allows threads to update shared memory without acquiring locks.

Instead of blocking other threads, CAS lets threads detect conflicts and retry. When contention is moderate, this approach often outperforms locks. When contention is extreme, the trade-offs become more nuanced.

This chapter covers how CAS works at the hardware level, how to use it correctly in your code, and the subtle bugs that can bite you if you are not careful.


What is Compare-and-Swap?
Real-World Analogy

Imagine you are updating a shared document with a colleague. Before making changes, you note the document's version number: version 5. You spend time writing your edits.

When you try to save, the system checks: is the document still on version 5? If yes, your changes are saved and the version becomes 6. If no, someone else edited it while you were working, and you must re-read, merge, and try again.

CAS works exactly like this, but at the CPU level and in nanoseconds.

In technical terms, CAS is an atomic operation that takes three arguments: a memory location, an expected value, and a new value. It atomically compares the current value at the memory location with the expected value. If they match, it replaces the current value with the new value and returns success. If they do not match, it leaves the memory unchanged and returns failure.

Compare And Swap
CAS swaps in a new value only if the current value still matches what the thread read. A failed CAS retries.

CAS retry
ABA problem

Thread 1
expected e
-
no CAS issued yet
shared memory
0
atomic check runs here
history
0
Thread 2
expected e
-
no CAS issued yet
execution order on the shared value
T1
T2
time
CAS attempts
0
rejected + retried
0
value
0
lock-free: a failed CAS retries, it never blocks
Two threads each want to increment the counter with CAS, no locks involved.



1 / 8
1x

The key insight is that the comparison and the swap happen as a single atomic operation. No other thread can intervene between the check and the update.




Yes

No

CAS Operation

Read current value
from memory location

Current value
== Expected?

Atomically write
new value

Return failure
Value unchanged

Return success

The diagram above shows the CAS decision flow. The operation reads the current value, compares it against what you expected, and either atomically swaps in the new value (success path in green) or leaves everything unchanged (failure path in red). The atomicity guarantee means no thread can see a half-completed CAS. Either the swap happened completely, or it did not happen at all.

The semantics in pseudocode look like this:








1234567
This simple primitive is powerful enough to build counters, stacks, queues, and complex concurrent data structures, all without traditional locks.


Benefits of CAS
Performance Benefits
Locks have overhead. Acquiring a mutex involves system calls, context switches, and thread scheduling. Even an uncontended lock takes 10-100 nanoseconds. Under contention, a thread might sleep for microseconds or milliseconds waiting for the lock holder.

CAS avoids all of this. A successful CAS is just a few CPU cycles. A failed CAS returns immediately, letting the thread retry without involving the operating system.

Scenario	Mutex Lock	CAS Operation
Uncontended	~25 ns	~5-10 ns
Low contention	~50-100 ns	~10-30 ns
High contention	1-10 μs (blocking)	50-500 ns (spinning)
These numbers vary by hardware, but the pattern holds: CAS is faster when you can afford to spin and retry.

No Blocking
With a mutex, if one thread crashes or gets stuck in an infinite loop while holding the lock, every other thread waiting for that lock is blocked forever. This is the fundamental vulnerability of lock-based programming.

With CAS, there is no lock to hold. A thread that crashes simply stops participating. Other threads continue their CAS operations without waiting. This property, where system progress does not depend on any single thread, is called lock-freedom, and CAS is the primitive that makes it possible.

Real-World Usage
CAS is not academic. It is everywhere in production systems:

Java ConcurrentHashMap: Uses CAS for lock-free bucket updates
Linux Kernel: Atomic reference counting, spinlocks built on CAS
Go runtime: Lock-free scheduler queues
Rust std::sync::atomic: All atomic types use CAS internally
Database engines: Lock-free buffer pools, transaction markers
When you call AtomicInteger.incrementAndGet() in Java or use std::atomic<int>::fetch_add() in C++, you are using CAS under the hood.


How CAS Works
Hardware-Level Implementation
Modern CPUs provide CAS as a single machine instruction. On x86/x64 processors, it is called CMPXCHG (compare and exchange). On ARM, the equivalent is a pair of instructions: LDREX (load exclusive) and STREX (store exclusive). These instructions coordinate with the CPU's cache coherence protocol to ensure atomicity across all cores.

When a core executes CMPXCHG:

The core signals the memory bus that it wants exclusive access to the cache line
Other cores invalidate their copies of that cache line (MESI protocol)
The comparison and conditional swap execute atomically
Other cores must reload the cache line if they need the value
This hardware coordination is what makes CAS truly atomic, not just "fast" but impossible to interrupt.




Compare and swapatomically
CMPXCHG request (exclusive access)
Invalidate cache line
Acknowledge invalidation
Exclusive access granted
Write new value
Read (cache miss, reload)
Return new value
Core 1
Cache/Memory
Core 2
Core 1
Cache/Memory
Core 2




8 / 8
The sequence diagram shows how cache coherence makes CAS atomic across cores. When Core 1 initiates a CAS, it first gains exclusive access to the relevant cache line. Core 2 must invalidate its cached copy. Only after Core 1 has exclusive access does the compare-and-swap execute. If Core 2 later reads the value, it gets a cache miss and reloads the updated value.

The CAS Loop Pattern
A single CAS can fail if another thread modified the value first. In practice, you almost always wrap CAS in a retry loop:








1234567
This pattern is called a CAS loop or a lock-free update loop. You read the current value, compute what you want to change it to, and attempt the CAS. If another thread beat you to it, your expected value no longer matches, the CAS fails, and you retry with the new current value.




Success

Failure

Start

Read current
value

Compute
new value

CAS
attempt

Done

The retry loop (shown as the arrow from CAS failure back to Read) is what distinguishes lock-free algorithms from lock-based ones. A thread might retry multiple times under contention, but it will never block indefinitely waiting for another thread.

Weak vs Strong CAS
Some platforms distinguish between strong CAS and weak CAS:

Strong CAS: Only fails if the value actually changed. If the current value equals expected, the swap always succeeds.
Weak CAS: May fail spuriously even if the values match. The hardware might report failure due to cache-related events, not actual value changes.
CAS Type	Spurious Failure?	Use Case	Languages
Strong	No	When single-attempt matters	Java compareAndSet
Weak	Yes	In loops where retry is cheap	C++ compare_exchange_weak
Weak CAS exists because on some architectures (like ARM), implementing a spurious-failure-free CAS requires extra synchronization that is expensive. If you are already in a retry loop, weak CAS is often faster because you are going to retry anyway.

In Java, AtomicInteger.compareAndSet() is always strong. In C++, you have both compare_exchange_strong and compare_exchange_weak. Use weak CAS in loops, strong CAS when you are making a one-shot attempt and need a definitive answer.


CAS in Practice
Basic Usage

Java
Java provides CAS through the java.util.concurrent.atomic package. The compareAndSet method performs the CAS operation.

Java







12345678910111213141516171819202122232425262728293031
The tryIncrement method shows a single CAS attempt that may fail. The incrementAndGet method shows the CAS loop pattern where we retry until success. The built-in incrementAndGet() does the same thing internally but is optimized.


Example: Lock-Free Counter
Let us build a complete lock-free counter and test it under contention:


Java







12345678910111213141516171819202122232425262728293031
The output shows that even though 10 threads performed 1,000,000 increments in total, the counter correctly reached 1,000,000. The CAS failures count shows how many times threads had to retry. Under high contention, failures are common, but correctness is maintained.


The ABA Problem
The ABA problem is a subtle bug that can occur with CAS-based algorithms. Here is the scenario:

Thread 1 reads value A from a location
Thread 1 is preempted (paused by OS)
Thread 2 changes the value from A to B
Thread 2 changes the value from B back to A
Thread 1 resumes and performs CAS, expecting A, seeing A
CAS succeeds, but the state has actually changed!
The problem is that CAS only checks the value, not whether the value changed and changed back. Thread 1 thinks nothing happened because the value is still A, but Thread 2 might have done significant work in between.




Preempted...
Memory = B
Memory = A (but state changed!)
Resumes
CAS succeeds!
But intermediate state changemay have corrupted linked data
Read A (expected = A)
Change A → B
Change B → A
CAS(A, newValue)
Thread 1
Memory [A]
Thread 2
Thread 1
Memory [A]
Thread 2




10 / 10
The diagram shows the classic ABA sequence. Thread 1 reads A and gets preempted. Thread 2 performs a complete A→B→A cycle. When Thread 1 resumes, it sees A and thinks nothing changed. But if A was a pointer to a node in a linked list, that node might have been freed and reallocated during the B phase. Thread 1's CAS succeeds, but it is now pointing to garbage or a completely different node.

When It Matters
The ABA problem is most dangerous with pointer-based structures:

Lock-free stacks: Pop returns a node, another thread pops and pushes, reusing the same memory address
Lock-free queues: Similar issues with node recycling
Memory pools: Reused memory addresses look identical to CAS
For simple integer counters, ABA is usually not a problem. If a counter goes 5→6→5, the CAS might succeed unexpectedly, but the value is still valid.

Solutions
1. Tagged Pointers / Version Numbers
Add a version counter that increments on every modification. Instead of comparing just the pointer, compare pointer + version:




Tagged Pointer (64-bit)

Pointer: 0x1234

Version: 42

Regular Pointer

Pointer: 0x1234

Even if the pointer returns to the same address, the version number will be different, causing the CAS to fail.

2. AtomicStampedReference (Java)
Java provides AtomicStampedReference specifically for solving the ABA problem:

Java







12345678910111213141516171819202122232425262728293031
The stamp (version number) increments on every operation. Even if the reference returns to the same value, the stamp will be different.

3. Hazard Pointers
A more sophisticated technique where threads publish pointers they are currently using. Other threads check these "hazard pointers" before freeing memory. This prevents the problematic memory reuse entirely.

4. Epoch-Based Reclamation
Memory is not freed immediately. Instead, it is retired to a list. Only when all threads have advanced past the retirement epoch is the memory actually freed. This ensures no thread is using the memory during the ABA-vulnerable window.


Performance Considerations
CAS vs Lock Overhead
Operation	Uncontended	High Contention (8 threads)
Mutex lock/unlock	~25 ns	~1-10 μs
CAS (success)	~5-10 ns	~10-30 ns
CAS (failure + retry)	N/A	~50-200 ns average
CAS (extreme contention)	N/A	May exceed lock
When CAS Wins
Short critical sections: Less time for contention to build
Read-heavy workloads: Readers do not conflict with each other
Low to moderate contention: Retries are rare
Latency-sensitive applications: No blocking means no tail latency spikes
When Locks Win
High contention: CAS spinning wastes CPU
Long critical sections: Blocking is cheaper than burning cycles
Priority scheduling needs: Locks can support priority inheritance
Simpler correctness reasoning: Easier to verify