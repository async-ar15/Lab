A thread doesn't just run from start to finish in one continuous burst. It moves through different states: waiting to start, actively running, blocked on I/O, waiting for a lock, and eventually terminating.

When your multi-threaded application hangs or behaves unexpectedly, understanding these states is often the key to diagnosing what went wrong.


What is Thread Lifecycle?
A thread's lifecycle is the sequence of states it passes through from creation to termination. Think of it like the lifecycle of an employee at a company: hired (created), onboarding (ready), actively working (running), waiting for resources or approvals (blocked/waiting), and eventually leaving the company (terminated).

At any given moment, a thread exists in exactly one state. External events and method calls cause transitions between states. The operating system's scheduler decides which runnable threads actually get CPU time.

Universal Thread States (OS Level)
At the operating system level, threads have these fundamental states:

State	Description
Ready	Thread can run, waiting for CPU time
Running	Actively executing on a CPU core
Blocked	Waiting for I/O, lock, or external event
Terminated	Finished execution, cannot run again
Every language's thread states map to these OS-level concepts, but the granularity of exposure varies widely. The diagram below shows the complete state model that we'll reference throughout this chapter:




Thread created

start() called

Scheduler assigns CPU

Yield / Time slice expired

Waiting for monitor lock

wait() / join() / park()

sleep() / wait(timeout)

run() completes or exception

Lock acquired

notify() / join completes

Timeout or notified

NEW

RUNNABLE

RUNNING

BLOCKED

WAITING

TIMED_WAITING

TERMINATED


How Different Languages Model Thread States
Languages expose thread states in very different ways, and those differences reflect deliberate design philosophies.

Java: Detailed State Introspection
Java provides Thread.State, an enum with six values: NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, and TERMINATED. You can query any thread's state at any time with thread.getState().

This makes Java excellent for debugging concurrency issues. You can programmatically detect deadlocks by finding threads in BLOCKED state waiting for each other's locks.

C#: Flags-Based Flexibility
C#'s ThreadState is a flags enum, meaning a thread can be in multiple states simultaneously. For example, a background thread that's sleeping might have ThreadState.WaitSleepJoin | ThreadState.Background. This is more flexible but requires bitwise operations to check states properly.

Python and C++: Minimal Exposure
Python's threading module only tells you if a thread is_alive(). C++ only tells you if a thread is joinable() (started but not yet joined). For detailed state analysis, you need external tools or platform-specific APIs.

Go: Deliberate Abstraction
Go takes a different approach: goroutine states aren't exposed at all. There's no getState() method and no state enum, and that is intentional.

Go's philosophy is that if you need to check a goroutine's state, you're probably doing something wrong. Instead of inspecting state, you communicate completion through channels, wait for goroutines with sync.WaitGroup, and handle cancellation with context.Context. This design pushes you toward patterns that are safer and more composable.


The Thread States
Each state is worth examining in detail, with code examples showing how to observe or trigger it in each language.

State 1: NEW (Created)
A thread in the NEW state has been created as an object in memory, but hasn't started executing yet. The operating system hasn't allocated resources for its execution.

In this state, the thread object exists in your program's heap, but no OS-level thread has been created. The thread cannot run any code yet, and calling most thread methods will fail or have no effect.


Java

Run







1234567891011121314
State 2: RUNNABLE (Ready to Run)
Once start() is called, the thread moves to the RUNNABLE state. The OS has created the thread and it's eligible to run, but it might not be running right now. The scheduler decides which runnable threads get CPU time.

In this state, the OS thread exists and is scheduled, but it may or may not be executing at any given moment. It competes with other threads for CPU time, and the scheduler can preempt it at any point.


Java

Run







1234567891011121314151617181920
State 3: RUNNING (Executing)
A thread is RUNNING when it's actively executing instructions on a CPU core. This is a sub-state of RUNNABLE in most models. The thread has been selected by the scheduler and is consuming CPU cycles.

A thread leaves the RUNNING state when its time slice expires (preemption), when it voluntarily yields, when it blocks on I/O or synchronization, or when it terminates.




Thread Scheduler
Runnable Queue
CPU Cores
Thread A
Thread B
Thread C
Thread D
Thread E
Scheduler
Core 1Thread X
Core 2Thread Y
Core 3(idle)
Core 4(idle)
The scheduler continuously moves threads between RUNNABLE and RUNNING. On a 4-core machine, at most 4 threads can be RUNNING simultaneously. Others wait in the runnable queue.

In this example, threads X and Y are RUNNING (assigned to cores, shown in green), threads A-E are RUNNABLE (waiting in the queue, shown in orange), and cores 3-4 are idle (shown in gray).

Why can't most languages distinguish RUNNING from RUNNABLE?

The distinction between "ready to run" and "actually running" exists at the OS level, but user-space programs typically can't observe it reliably.

By the time you query a thread's state and receive the answer, the thread may have been preempted or resumed multiple times. Java, C#, Python, and Go all combine these into a single "alive and not blocked" concept.

State 4: BLOCKED (Waiting for Lock)
A thread enters the BLOCKED state when it tries to acquire a lock (monitor) that another thread holds. It cannot proceed until the lock becomes available.

This is different from WAITING. BLOCKED specifically means "waiting to enter a synchronized block or method." The thread isn't waiting for a signal; it's waiting for exclusive access to a resource.


Java

Run







7891011121314151617181920212223242526272829303132333435363738146711public class BlockedStateDemo {    public static void main(String[] args) throws InterruptedException {        Thread holder = new Thread(() -> {            synchronized (lock) {                } catch (InterruptedException e) {
State 5: WAITING (Indefinite Wait)
A thread enters the WAITING state when it explicitly waits for another thread to perform an action. Unlike BLOCKED, the thread isn't competing for a lock; it's parked until signaled.

Common triggers for WAITING:

Java: Object.wait(), Thread.join(), LockSupport.park()
C#: Monitor.Wait(), Thread.Join(), ManualResetEvent.WaitOne()
Python: Condition.wait(), Thread.join(), Event.wait()
C++: condition_variable.wait(), thread.join()
Go: Blocking channel receive, sync.WaitGroup.Wait()
The thread will stay in WAITING forever unless another thread wakes it up.


Java

Run







123456789101112131415161718192021222324252627282930
State 6: TIMED_WAITING (Bounded Wait)
TIMED_WAITING is similar to WAITING, but with a timeout. The thread will wake up either when signaled or when the timeout expires, whichever comes first.

Common triggers:

Java: Thread.sleep(millis), Object.wait(millis), Thread.join(millis)
C#: Thread.Sleep(millis), Monitor.Wait(obj, millis), Thread.Join(millis)
Python: time.sleep(secs), Condition.wait(timeout), Thread.join(timeout)
C++: sleep_for(), wait_for(), join() doesn't have native timeout
Go: time.Sleep(), select with timeout, context.WithTimeout()

Java

Run







12345678910111213141516171819202122232425
State 7: TERMINATED (Dead)
A thread enters the TERMINATED state when its execution completes, either normally or by throwing an uncaught exception. The thread can never run again. Its resources are released.

In this state, the thread has finished executing and the OS thread no longer exists. isAlive() returns false, though the Thread object may still exist in memory, and calling start() again throws an exception.


Java

Run







1234567891011121314151617181920212223242526

State Transitions Summary
From State	To State	Trigger
NEW	RUNNABLE	start() / Start() called
RUNNABLE	RUNNING	Scheduler assigns CPU
RUNNING	RUNNABLE	Time slice expires, yield()
RUNNING	BLOCKED	Attempt to acquire held lock
RUNNING	WAITING	Indefinite wait call (wait(), join(), channel receive)
RUNNING	TIMED_WAITING	Bounded wait call (sleep(), wait(timeout))
RUNNING	TERMINATED	Execution completes or exception
BLOCKED	RUNNABLE	Lock acquired
WAITING	RUNNABLE	Signaled (notify(), channel send, Pulse())
TIMED_WAITING	RUNNABLE	Timeout expires or signaled



alt
[Lock available]
[Lock held by other]
NEW / Unstarted
RUNNABLE / Running
RUNNING
RUNNING
BLOCKED / WaitSleepJoin
RUNNABLE
WAITING / WaitSleepJoin
RUNNABLE
TIMED_WAITING / WaitSleepJoin
RUNNABLE
TERMINATED / Stopped
new Thread() / Thread()
start() / Start() / go
Assign CPU
synchronized / lock / Lock()
Acquired
Blocked
Lock released
wait() / Wait() / <-channel
notify() / Pulse() / channel send
sleep() / Sleep()
Timeout
Execution completes
User Code
Thread
OS Scheduler
Lock/Monitor
User Code
Thread
OS Scheduler
Lock/Monitor

