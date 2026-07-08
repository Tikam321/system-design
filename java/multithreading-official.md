# Multithreading Interview Questions & Answers

A complete guide covering conceptual, Java-specific, and coding-based multithreading interview questions — from basics to advanced.

---

## Table of Contents
1. [Basic Concepts](#basic-concepts)
2. [Thread Creation & Lifecycle](#thread-creation--lifecycle)
3. [Synchronization](#synchronization)
4. [Inter-Thread Communication](#inter-thread-communication)
5. [Concurrency Problems](#concurrency-problems)
6. [Executor Framework & Thread Pools](#executor-framework--thread-pools)
7. [Locks, Atomic Variables & java.util.concurrent](#locks-atomic-variables--javautilconcurrent)
8. [Java Memory Model](#java-memory-model)
9. [Common Coding Questions](#common-coding-questions)
10. [Advanced / Design Questions](#advanced--design-questions)

---

## Basic Concepts

### 1. What is a thread?
A thread is the smallest unit of execution within a process. Multiple threads within the same process share the same memory space (heap, code, data) but have their own stack, program counter, and registers.

### 2. What is multithreading?
Multithreading is the ability of a CPU (or a program) to execute multiple threads concurrently, allowing better utilization of CPU resources and improved application responsiveness.

### 3. Difference between a process and a thread?

| Process | Thread |
|---|---|
| Independent execution unit with its own memory space | Executes within a process, shares memory with other threads |
| Heavyweight (more overhead to create/switch) | Lightweight (less overhead) |
| Inter-process communication is complex (sockets, pipes, shared memory) | Inter-thread communication is easier (shared variables) |
| A crash in one process doesn't usually affect another | A crash in one thread can bring down the whole process |

### 4. What is multitasking vs multithreading vs multiprocessing?
- **Multitasking**: OS runs multiple processes concurrently (time-sliced on a single core or truly parallel on multiple cores).
- **Multithreading**: A single process runs multiple threads concurrently.
- **Multiprocessing**: Multiple processes run in parallel, typically across multiple CPU cores.

### 5. What are the advantages of multithreading?
- Better CPU utilization (overlap I/O wait with computation)
- Improved application responsiveness (e.g., UI thread stays responsive)
- Simplified modeling of concurrent real-world tasks
- Resource sharing (memory) is cheaper than IPC

### 6. What are the disadvantages/challenges of multithreading?
- Race conditions and data corruption if not synchronized properly
- Deadlocks, livelocks, starvation
- Debugging is harder (non-deterministic behavior)
- Context-switching overhead
- Increased code complexity

### 7. What is context switching?
The process of storing the state of a currently running thread/process and loading the state of another so execution can resume later. It's necessary for time-sharing CPUs among multiple threads but has performance overhead.

### 8. What is concurrency vs parallelism?
- **Concurrency**: Multiple tasks make progress during overlapping time periods (may or may not run at the exact same instant — e.g., single-core time slicing).
- **Parallelism**: Multiple tasks literally execute at the same instant, requiring multiple cores/processors.

---

## Thread Creation & Lifecycle

### 9. What are the ways to create a thread in Java?
1. Extending the `Thread` class and overriding `run()`.
2. Implementing the `Runnable` interface and passing it to a `Thread` object.
3. Implementing `Callable` (returns a result, can throw checked exceptions) and using it with `ExecutorService`.
4. Using the `ExecutorService`/thread pool framework.

```java
// 1. Extending Thread
class MyThread extends Thread {
    public void run() { System.out.println("Running..."); }
}

// 2. Implementing Runnable
class MyRunnable implements Runnable {
    public void run() { System.out.println("Running..."); }
}

Thread t1 = new MyThread();
Thread t2 = new Thread(new MyRunnable());
t1.start();
t2.start();
```

### 10. Why is `Runnable` generally preferred over extending `Thread`?
- Java doesn't support multiple inheritance, so extending `Thread` uses up your only superclass slot.
- `Runnable` promotes better design — separates the "task" from the "thread of execution."
- `Runnable` objects can be reused and submitted to thread pools (`ExecutorService`).

### 11. What is the difference between `run()` and `start()`?
- `start()` creates a new thread of execution and the JVM internally calls `run()` on that new thread.
- Calling `run()` directly just executes the method on the **current** thread — no new thread is created.

### 12. What are the states in a thread's lifecycle?
1. **New** — thread created but `start()` not yet called.
2. **Runnable** — eligible to run, waiting for CPU scheduling.
3. **Running** — actively executing.
4. **Blocked/Waiting** — waiting for a lock (`Blocked`), or waiting indefinitely for another thread's signal (`Waiting`), or waiting for a fixed time (`Timed Waiting`).
5. **Terminated** — execution completed or thread was stopped due to an exception.

### 13. What is a daemon thread?
A background thread that doesn't prevent the JVM from exiting when all user (non-daemon) threads finish. Example: Garbage Collector thread. Set via `thread.setDaemon(true)` before calling `start()`.

### 14. What does `Thread.sleep()` do? Does it release the lock?
`sleep()` pauses the current thread for a specified time. It does **not** release any locks it holds — it just gives up the CPU.

### 15. What is thread priority? Does it guarantee execution order?
Threads have priority (1–10 in Java, default 5) that hints to the scheduler which thread should be preferred. It's only a **hint** — actual behavior depends on the OS scheduler and is not guaranteed.

### 16. What is the difference between `join()` and `sleep()`?
- `join()` makes the calling thread wait until the thread it's called on finishes execution.
- `sleep()` pauses the current thread for a fixed duration, unrelated to any other thread's completion.

---

## Synchronization

### 17. What is a race condition?
A race condition occurs when two or more threads access shared data concurrently, and the final outcome depends on the unpredictable timing/order of their execution — leading to inconsistent or incorrect results.

### 18. What is the `synchronized` keyword? How does it work?
`synchronized` ensures that only one thread can execute a block/method at a time by acquiring an intrinsic lock (monitor) associated with an object (or class, for static methods).

```java
public synchronized void increment() { count++; }

public void increment() {
    synchronized (this) { count++; }
}
```

### 19. Difference between synchronized method and synchronized block?
- **Synchronized method**: locks the entire method using the object's (or class's) monitor — less granular.
- **Synchronized block**: locks only a specific critical section, allowing finer-grained control and better performance since less code is locked.

### 20. What is a monitor lock (intrinsic lock)?
Every Java object has an associated intrinsic lock (monitor). When a thread enters a `synchronized` method/block, it acquires this lock; other threads attempting to enter must wait until it's released.

### 21. What is the `volatile` keyword?
`volatile` guarantees visibility of changes to variables across threads — every read of a volatile variable is from main memory, and every write is flushed to main memory immediately. It does **not** guarantee atomicity of compound operations (like `count++`).

### 22. Difference between `volatile` and `synchronized`?

| `volatile` | `synchronized` |
|---|---|
| Ensures visibility only | Ensures visibility **and** atomicity/mutual exclusion |
| No locking, non-blocking | Locking involved, can block threads |
| Works on variables | Works on methods/blocks |
| Cheaper performance cost | Higher overhead |

### 23. What is a reentrant lock (in the context of intrinsic locks)?
Java's intrinsic locks are reentrant — if a thread already holds a lock on an object, it can re-acquire that same lock (e.g., calling another synchronized method on the same object) without deadlocking itself.

### 24. What is deadlock? How do you prevent it?
Deadlock occurs when two or more threads are blocked forever, each waiting for a resource held by another.

**Prevention strategies:**
- Always acquire locks in a consistent, global order.
- Use timeouts (`tryLock(timeout)`).
- Avoid nested locks where possible.
- Use a single lock for related operations instead of multiple fine-grained locks.
- Use higher-level concurrency utilities instead of manual locking.

### 25. What is livelock?
Threads are not blocked but keep responding to each other in a way that prevents progress — e.g., two threads repeatedly yielding to each other to avoid a deadlock, but as a result neither makes progress.

### 26. What is starvation?
A thread is perpetually denied access to resources it needs because other "greedy" threads keep grabbing them first — often due to poor thread priority management.

### 27. What are the four necessary conditions for deadlock (Coffman conditions)?
1. **Mutual Exclusion** — resource can't be shared.
2. **Hold and Wait** — a thread holds one resource while waiting for another.
3. **No Preemption** — resources can't be forcibly taken from a thread.
4. **Circular Wait** — a cycle of threads each waiting on the next.

---

## Inter-Thread Communication

### 28. What are `wait()`, `notify()`, and `notifyAll()`?
Methods on `Object` used for inter-thread communication:
- `wait()` — releases the lock and makes the current thread wait until notified.
- `notify()` — wakes up a single waiting thread.
- `notifyAll()` — wakes up all waiting threads.

These **must** be called from within a synchronized block/method, otherwise `IllegalMonitorStateException` is thrown.

```java
synchronized (lock) {
    while (!condition) {
        lock.wait();
    }
    // proceed
}

synchronized (lock) {
    condition = true;
    lock.notifyAll();
}
```

### 29. Why should `wait()` be called in a loop (`while`) instead of an `if`?
Because of **spurious wakeups** — a thread may wake up without an explicit `notify()`. Checking the condition in a loop ensures the thread re-verifies the condition before proceeding.

### 30. Difference between `notify()` and `notifyAll()`?
`notify()` wakes exactly one waiting thread (chosen arbitrarily by JVM), which can cause missed signals if the wrong thread wakes up. `notifyAll()` wakes all waiting threads, letting each re-check its condition — safer, though slightly less efficient.

### 31. What is the difference between `wait()` and `sleep()`?

| `wait()` | `sleep()` |
|---|---|
| Defined in `Object` class | Defined in `Thread` class |
| Releases the lock | Does NOT release the lock |
| Must be called within synchronized context | Can be called anywhere |
| Resumed via `notify()`/`notifyAll()` | Resumes automatically after timeout |

---

## Concurrency Problems

### 32. What is the Producer-Consumer problem?
A classic synchronization problem where "producer" threads generate data into a shared buffer and "consumer" threads take data out. The buffer has limited size, so producers must wait when it's full and consumers must wait when it's empty. Typically solved using `wait()`/`notify()`, `BlockingQueue`, or semaphores.

### 33. What is the Dining Philosophers problem?
A classic illustration of deadlock and resource contention: philosophers sit at a table with one fork between each pair, needing both forks to eat. If everyone picks up their left fork simultaneously, all are stuck waiting for the right fork — a deadlock. Solutions include resource ordering, a waiter/arbitrator, or allowing only N-1 philosophers to sit at once.

### 34. What is false sharing?
A performance issue (not correctness) where independent variables used by different threads happen to sit on the same CPU cache line. Updates by one thread invalidate the cache line for other threads even though they access different variables, causing unnecessary cache coherence traffic.

---

## Executor Framework & Thread Pools

### 35. Why use thread pools instead of creating threads manually?
- Creating/destroying threads is expensive.
- Thread pools reuse a fixed set of worker threads, reducing overhead.
- They give you control over resource limits (avoiding thread explosion under load).
- Provide task queuing, scheduling, and lifecycle management.

### 36. What is `ExecutorService`?
A higher-level replacement for manually managing threads, providing methods to submit tasks (`Runnable`/`Callable`), manage a thread pool, and retrieve results via `Future`.

```java
ExecutorService executor = Executors.newFixedThreadPool(4);
Future<Integer> future = executor.submit(() -> 42);
Integer result = future.get();
executor.shutdown();
```

### 37. What are the common types of thread pools in `Executors`?
- `newFixedThreadPool(n)` — fixed number of threads.
- `newCachedThreadPool()` — creates threads as needed, reuses idle ones, unbounded.
- `newSingleThreadExecutor()` — single worker thread, tasks executed sequentially.
- `newScheduledThreadPool(n)` — supports delayed/periodic task execution.
- `newWorkStealingPool()` — uses a `ForkJoinPool` internally, good for parallel workloads.

*(Note: `Executors` factory methods are often discouraged in production in favor of directly configuring `ThreadPoolExecutor` to avoid unbounded queues/thread counts.)*

### 38. What are the core parameters of `ThreadPoolExecutor`?
```java
new ThreadPoolExecutor(
    corePoolSize,      // threads kept alive even when idle
    maximumPoolSize,   // max threads allowed
    keepAliveTime,     // idle time before excess threads are terminated
    TimeUnit unit,
    BlockingQueue<Runnable> workQueue, // holds tasks before execution
    RejectedExecutionHandler handler   // policy when pool/queue is full
);
```

### 39. What is `Future` and `CompletableFuture`?
- `Future` represents the result of an asynchronous computation; `get()` blocks until it's done.
- `CompletableFuture` (Java 8+) extends this with composability — chaining callbacks (`thenApply`, `thenCompose`, `thenCombine`), combining multiple futures, and handling exceptions without blocking.

```java
CompletableFuture.supplyAsync(() -> fetchData())
    .thenApply(data -> process(data))
    .thenAccept(result -> System.out.println(result));
```

### 40. Difference between `Runnable` and `Callable`?

| `Runnable` | `Callable` |
|---|---|
| `void run()` — no return value | `V call()` — returns a value |
| Cannot throw checked exceptions | Can throw checked exceptions |
| Used with `Thread` or `Executor` | Used with `ExecutorService` (returns `Future`) |

### 41. What happens if a task submitted to a thread pool throws an exception?
- If submitted via `execute()`, the exception propagates to the thread's uncaught exception handler (usually just prints stack trace), and the worker thread is replaced.
- If submitted via `submit()`, the exception is captured inside the `Future` and re-thrown (wrapped in `ExecutionException`) when you call `future.get()`.

### 42. What is `ForkJoinPool`?
A specialized executor for divide-and-conquer, recursive tasks (`RecursiveTask`/`RecursiveAction`) that uses **work-stealing** — idle threads "steal" tasks from busy threads' queues, improving load balancing for parallel computations. Powers Java's parallel streams.

---

## Locks, Atomic Variables & java.util.concurrent

### 43. What is `ReentrantLock` and how does it differ from `synchronized`?

| `synchronized` | `ReentrantLock` |
|---|---|
| Implicit lock/unlock, JVM handles it | Explicit `lock()`/`unlock()` calls needed |
| Can't interrupt a waiting thread | Supports `lockInterruptibly()` |
| No timeout support | Supports `tryLock(timeout)` |
| No fairness policy control | Can construct with `fair=true` for FIFO fairness |
| Simpler, less error-prone | More flexible but must remember to unlock (use `try/finally`) |

```java
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    // critical section
} finally {
    lock.unlock();
}
```

### 44. What is `ReadWriteLock`?
Allows multiple threads to read concurrently but ensures exclusive access for writes — useful when reads vastly outnumber writes, improving throughput compared to a single mutual-exclusion lock.

### 45. What are atomic classes (`AtomicInteger`, `AtomicLong`, `AtomicReference`)?
Classes in `java.util.concurrent.atomic` that provide lock-free, thread-safe operations on single variables using low-level CAS (Compare-And-Swap) CPU instructions — faster than `synchronized` for simple counters/flags.

```java
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet(); // atomic, thread-safe
```

### 46. What is Compare-And-Swap (CAS)?
A CPU-level atomic instruction: it compares a memory location's current value to an expected value, and only if they match, swaps in a new value — all as one atomic operation. It's the foundation for most lock-free data structures.

### 47. What is a `Semaphore`?
A concurrency utility that maintains a set of permits. Threads call `acquire()` to take a permit (blocking if none available) and `release()` to return one — commonly used to limit the number of threads accessing a resource concurrently (e.g., connection pool limiting).

### 48. What is `CountDownLatch`?
A synchronization aid that lets one or more threads wait until a set of operations in other threads completes. Initialized with a count; each `countDown()` decrements it; threads calling `await()` block until it reaches zero. **One-time use** — cannot be reset.

### 49. What is `CyclicBarrier`?
Similar to `CountDownLatch`, but it allows a set of threads to wait for each other to reach a common barrier point, and **can be reused** (cyclic) after all threads arrive, optionally running a barrier action.

### 50. Difference between `CountDownLatch` and `CyclicBarrier`?

| `CountDownLatch` | `CyclicBarrier` |
|---|---|
| One-time use, cannot reset | Reusable across multiple phases |
| Threads counting down need not wait themselves | All participating threads must wait at the barrier |
| No action triggered automatically | Can run a `Runnable` action when barrier trips |

### 51. What are concurrent collections? Give examples.
Thread-safe collection implementations designed for high concurrency (better than legacy synchronized collections like `Vector`/`Hashtable`):
- `ConcurrentHashMap` — segmented/bucket-level locking (Java 8+ uses CAS + synchronized on bins) instead of locking the whole map.
- `CopyOnWriteArrayList` — writes create a new copy of the underlying array; ideal for read-heavy, write-rare scenarios.
- `BlockingQueue` (`ArrayBlockingQueue`, `LinkedBlockingQueue`) — supports blocking `put()`/`take()`, ideal for producer-consumer patterns.
- `ConcurrentLinkedQueue` — non-blocking, lock-free queue using CAS.

### 52. How does `ConcurrentHashMap` achieve thread safety without locking the entire map?
Older versions (Java 7) used **lock striping** — dividing the map into segments, each with its own lock. Java 8+ uses finer-grained locking at the bucket/bin level combined with CAS operations, allowing much higher concurrent throughput than `Hashtable` or `Collections.synchronizedMap()`.

### 53. What is `ThreadLocal`?
Provides thread-confined variables — each thread accessing a `ThreadLocal` variable gets its own independently initialized copy, eliminating the need for synchronization. Commonly used for per-thread contexts like database connections, user sessions, or `SimpleDateFormat` instances (which aren't thread-safe).

```java
ThreadLocal<SimpleDateFormat> formatter =
    ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));
```
⚠️ Common pitfall: not calling `remove()` in thread-pool environments causes memory leaks since threads are reused.

---

## Java Memory Model

### 54. What is the Java Memory Model (JMM)?
A specification that defines how threads interact through memory — what visibility and ordering guarantees are provided for shared variable access, and how/when changes made by one thread become visible to others.

### 55. What is "happens-before" relationship?
A guarantee in JMM that if action A happens-before action B, then the effects of A (including memory writes) are visible to B. Examples of happens-before relationships:
- A `synchronized` block's exit happens-before the next thread's entry into a block synchronized on the same monitor.
- A write to a `volatile` field happens-before every subsequent read of that field.
- A call to `Thread.start()` happens-before any action in the started thread.
- All actions in a thread happen-before another thread successfully returns from `join()` on it.

### 56. Why can a variable modified by one thread appear "stale" to another thread without synchronization?
Due to CPU caching and compiler/JIT reordering optimizations, a thread may read a cached/stale value instead of the latest value in main memory, unless proper synchronization (`volatile`, `synchronized`, locks) establishes a happens-before relationship.

### 57. What is instruction reordering, and how does synchronization prevent unwanted reordering?
Compilers and CPUs may reorder instructions for optimization as long as it doesn't affect single-threaded program correctness — but this can break multithreaded assumptions. `volatile` and synchronized blocks act as memory barriers, preventing certain reorderings across their boundary.

---

## Common Coding Questions

### 58. Implement a simple thread-safe counter.
```java
class Counter {
    private int count = 0;
    public synchronized void increment() { count++; }
    public synchronized int getCount() { return count; }
}
```

### 59. Print numbers alternately using two threads (odd/even).
```java
class NumberPrinter {
    private int number = 1;
    private final int max;
    private final Object lock = new Object();

    NumberPrinter(int max) { this.max = max; }

    void printOdd() {
        synchronized (lock) {
            while (number <= max) {
                while (number % 2 == 0) {
                    try { lock.wait(); } catch (InterruptedException e) {}
                }
                System.out.println("Odd: " + number++);
                lock.notifyAll();
            }
        }
    }

    void printEven() {
        synchronized (lock) {
            while (number <= max) {
                while (number % 2 != 0) {
                    try { lock.wait(); } catch (InterruptedException e) {}
                }
                System.out.println("Even: " + number++);
                lock.notifyAll();
            }
        }
    }
}
```

### 60. Implement a Producer-Consumer using `BlockingQueue`.
```java
BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(10);

// Producer
new Thread(() -> {
    int val = 0;
    while (true) {
        try {
            queue.put(val++);
        } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
    }
}).start();

// Consumer
new Thread(() -> {
    while (true) {
        try {
            int val = queue.take();
            System.out.println("Consumed: " + val);
        } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
    }
}).start();
```

### 61. Detect a deadlock scenario (illustrative example).
```java
Object lockA = new Object();
Object lockB = new Object();

Thread t1 = new Thread(() -> {
    synchronized (lockA) {
        try { Thread.sleep(100); } catch (InterruptedException e) {}
        synchronized (lockB) { System.out.println("T1 acquired both locks"); }
    }
});

Thread t2 = new Thread(() -> {
    synchronized (lockB) {
        try { Thread.sleep(100); } catch (InterruptedException e) {}
        synchronized (lockA) { System.out.println("T2 acquired both locks"); }
    }
});
// t1 and t2 will deadlock: each holds one lock while waiting for the other
```
**Fix**: always acquire `lockA` before `lockB` in both threads (consistent lock ordering).

### 62. Implement a thread-safe Singleton (double-checked locking).
```java
class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
`volatile` is essential here to prevent another thread from seeing a partially constructed object due to instruction reordering during object creation.

---

## Advanced / Design Questions

### 63. How would you design a thread-safe LRU cache?
Use a `LinkedHashMap` with access-order enabled, wrapped with synchronization (or `ReentrantReadWriteLock` for better read concurrency), overriding `removeEldestEntry()` to evict the least recently used entry when capacity is exceeded.

### 64. How does `synchronized` differ in performance from `ReentrantLock` and atomic variables?
Roughly, from lowest to highest concurrency/throughput for simple operations: `synchronized` (JVM-managed, has been heavily optimized with biased/lightweight locking) → `ReentrantLock` (more flexible, similar or better under contention) → Atomic/CAS-based classes (fastest for simple counters, no blocking at all under low contention).

### 65. What is thread starvation caused by an unfair lock, and how can `ReentrantLock` fairness help?
An unfair lock may repeatedly let recently-active threads re-acquire the lock, starving others. `new ReentrantLock(true)` creates a fair lock, granting access in roughly FIFO order — at some throughput cost.

### 66. How do virtual threads (Project Loom, Java 21+) change multithreading?
Virtual threads are lightweight threads managed by the JVM (not the OS) that make blocking-style code scale to millions of concurrent threads without the memory/context-switch overhead of OS threads — greatly simplifying highly concurrent I/O-bound applications without needing complex async/reactive code.

### 67. What is the difference between synchronous, asynchronous, blocking, and non-blocking?
- **Synchronous**: caller waits for the operation to complete before continuing.
- **Asynchronous**: caller continues without waiting; result delivered later (callback/future).
- **Blocking**: a thread is suspended until the operation completes.
- **Non-blocking**: the thread is never suspended; it can immediately proceed or poll for completion.
(Note: sync/async and blocking/non-blocking are related but distinct axes — e.g., you can have async blocking or sync non-blocking designs.)

### 68. How would you debug a multithreading issue in production (e.g., a hang or high CPU)?
- Take a **thread dump** (`jstack`, or `kill -3` on the JVM) to inspect thread states and detect deadlocks/blocked threads.
- Use profilers (VisualVM, JFR/Java Flight Recorder, async-profiler) to identify hot locks or contention.
- Check logs for patterns correlating with the issue's timing.
- Reproduce with stress/load testing tools, and add fine-grained logging around suspected critical sections.

### 69. What are best practices for writing multithreaded code?
- Minimize shared mutable state — prefer immutability where possible.
- Keep synchronized blocks as small as possible.
- Prefer higher-level concurrency utilities (`java.util.concurrent`) over raw `wait()/notify()`.
- Always release locks in `finally` blocks.
- Avoid nested locks; if unavoidable, enforce a consistent acquisition order.
- Use thread pools instead of creating raw threads.
- Document thread-safety guarantees of your classes clearly.
- Write tests that stress concurrent paths (though non-determinism makes this inherently hard).

---

## Quick Cheat-Sheet Summary

| Concept | Key Point |
|---|---|
| Race condition | Unsynchronized access to shared mutable state |
| `synchronized` | Mutual exclusion + visibility |
| `volatile` | Visibility only, not atomicity |
| Deadlock | Circular wait for locks |
| `wait/notify` | Must hold lock; use in a `while` loop |
| `ExecutorService` | Manage thread pools, submit tasks |
| `Future`/`CompletableFuture` | Async result handling/composition |
| `CountDownLatch` | One-time wait for N events |
| `CyclicBarrier` | Reusable rendezvous point for N threads |
| `ConcurrentHashMap` | Fine-grained locking for concurrent access |
| `ThreadLocal` | Per-thread isolated variable |
| CAS | Lock-free atomic update primitive |
| Happens-before | Guarantees visibility ordering across threads |

---

*Good luck with your interview! Practice explaining these concepts out loud and be ready to write quick code snippets on a whiteboard or shared editor.*
