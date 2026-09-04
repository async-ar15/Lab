Two threads both increment a shared counter. Each runs counter++ a thousand times. The final value should be 2,000.

But when you run it, you get 1,847. Run it again: 1,923. Again: 1,756. The result is different every time, and always wrong.

This is a race condition.

Race conditions don't crash your program or throw exceptions. They corrupt your data with no error or warning, and they only appear under specific timing conditions that are nearly impossible to reproduce in testing.

This chapter explains what race conditions are, why they happen, and how to prevent them using critical sections.


What is a Race Condition?
A race condition occurs when the behavior of a program depends on the relative timing of events, such as the order in which threads are scheduled. When two or more threads access shared data concurrently, and at least one modifies it, the final result depends on who "wins the race" to access the data first.

Think of a bank transfer. You have $500 in your account. Two transactions happen simultaneously: a $100 withdrawal and a $200 deposit. The correct final balance should be $600. But if both transactions read the balance ($500) before either writes, you might end up with $300 (withdrawal wrote last) or $700 (deposit wrote last). Neither is correct.

The formal definition

A race condition exists when the program's correctness depends on the interleaving of operations from multiple threads, and at least one of those operations is a write.




Both threads try to increment counter
Final value: 1 (should be 2!)
Read counter (0)
Read counter (0)
Add 1 (result: 1)
Add 1 (result: 1)
Write 1
Write 1
Thread 1
counter = 0
Thread 2
Thread 1
counter = 0
Thread 2




8 / 8
In this diagram, both threads read 0, both add 1, and both write 1. The second increment is lost entirely.

Race conditions are dangerous because they produce no error and are non-deterministic. Unlike a null pointer exception that crashes immediately, a race condition corrupts data without any crash or warning.

Real-World Examples
Inventory overselling: Two customers buy the last item simultaneously. Both threads check stock (1 available), both proceed with purchase, both decrement stock. Result: stock is -1, and you've sold an item you don't have.

Lost database updates: Two users edit the same record. Both load the current version, both make changes, both save. The second save overwrites the first user's changes completely.

Double spending: A user submits a payment twice quickly. Both requests check the balance, both see sufficient funds, both deduct the amount. The user pays once but the account is debited twice, or vice versa.


How Race Conditions Happen
Race conditions require three ingredients:

Shared state: Multiple threads access the same variable or data structure
Mutability: At least one thread modifies the shared state
Concurrent access: Access happens without proper synchronization
Remove any one of these, and you eliminate the race condition. Immutable data can't have races. Data that isn't shared can't have races. Properly synchronized access prevents races.

The Non-Atomic Operation Problem
The classic race condition involves operations that look atomic but aren't. Consider count++. Conceptually, this is one operation: increment the counter. But at the machine level, it's three operations:

Read: Load the current value from memory into a register
Modify: Add 1 to the register
Write: Store the new value back to memory
Any other thread can execute between any of these steps. This is called a read-modify-write race.




Race Condition: count++ is not atomic
Another thread can interleave anywhere
count++ is actually 3 operations
Thread 2 reads here
Thread 2 reads here
Thread 2 reads here
1. Read count from memory
2. Add 1 in CPU register
3. Write result to memory
The Check-Then-Act Problem
Another common pattern is check-then-act: you check a condition, then act based on the result. But between the check and the act, another thread can change the condition.


Java
Java







1234567891011
Two threads call getInstance() simultaneously. Both see instance == null, both create a new object. Now you have two instances of what should be a singleton, and one is orphaned.


Critical Sections Explained
A critical section is a region of code that reads or writes shared state (for example: a counter, a map, a queue, a file, or a database row). Because multiple threads can reach that code at the same time, we must ensure the shared state is updated atomically and consistently.




Mutex: Mutual Exclusion
Thread 1
Thread 2
Thread 3
Mutex
Critical SectionsharedCounter++
lock
lock blocked
lock blocked
unlock
A critical section solution provides: mutual exclusion. At any instant, at most one thread is allowed to execute that protected region. Everyone else must wait until the current thread leaves.

Properties of a Good Critical Section Solution
Mutual exclusion: Only one thread executes the critical section at a time
Progress: If no thread is in the critical section, a waiting thread can enter
Bounded waiting: No thread waits forever (no starvation)
No assumptions about speed: Works regardless of how fast threads run
Race Condition

A Race Condition occurs when threads access shared data concurrently without synchronization, causing unpredictable results.

No Sync
Synchronized
Shared Counter
0
T1
Thread 1
Idle
Read
Increment
Write
T2
Thread 2
Idle
Read
Increment
Write
Operation Log
Click "Run Increment" to start...
Runs counter++ on both threads


No Sync: Both threads may read the same value, increment it, and write back—losing one update. This is a race condition.

counter++ is actually 3 operations: read → increment → write

Without synchronization, threads can interleave these operations

Fix: Use mutex, synchronized block, or atomic operations


Solutions: Protecting Critical Sections
Solution 1: Mutex/Locks
The most common solution is a mutex (mutual exclusion lock). Before entering a critical section, acquire the lock. After leaving, release it. If another thread holds the lock, you wait.


Java







12345678910111213141516171819202122232425262728293031
Solution 2: Atomic Operations
For simple variables like counters and flags, atomic operations provide lock-free thread safety. The hardware guarantees the operation completes without interruption.


Java







12345678910111213
Solution 3: Immutability
If data never changes, it can't have race conditions. Immutable objects are thread-safe by nature.


Java







123456789101112131415161718
