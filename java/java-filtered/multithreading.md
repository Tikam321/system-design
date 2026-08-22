# Java Multithreading & Thread Pooling — Complete Notes

A single, clean reference covering core threading concepts, thread pools, `ExecutorService`, synchronization, and common interview questions — each with the "what" **and** the "why".

---

## 1. What is a Thread?

A **thread** is the smallest unit of execution within a process. A single Java program (process) can run multiple threads concurrently, sharing the same memory space (heap) but each having its own stack, program counter, and registers.

**Why threads exist:** Modern CPUs have multiple cores. Without threads, a program can only do one thing at a time, wasting available hardware. Threads let a program do useful work (e.g., handle another user request) while one part is waiting (e.g., for a database response).

### Ways to create a thread in Java

```java
// 1. Extending Thread
class MyThread extends Thread {
    public void run() {
        System.out.println("Running in a thread");
    }
}
new MyThread().start();

// 2. Implementing Runnable (preferred — allows extending other classes too)
Runnable task = () -> System.out.println("Running in a thread");
new Thread(task).start();
```

**Why prefer `Runnable` over extending `Thread`:** Java doesn't support multiple inheritance. If your class extends `Thread`, it can't extend anything else. Implementing `Runnable` keeps the "task" separate from the "execution mechanism," which is also what `ExecutorService` expects.

---

## 2. Thread Lifecycle

A thread moves through the following states (`Thread.State` enum):

```
NEW → RUNNABLE → (BLOCKED / WAITING / TIMED_WAITING) → TERMINATED
```

| State | Meaning |
|---|---|
| NEW | Thread object created but `start()` not called yet |
| RUNNABLE | Eligible to run; may be running or waiting for CPU time |
| BLOCKED | Waiting to acquire a lock held by another thread |
| WAITING | Waiting indefinitely for another thread's signal (`wait()`, `join()`) |
| TIMED_WAITING | Waiting for a specified time (`sleep()`, `wait(timeout)`) |
| TERMINATED | Finished execution or exited due to exception |

**Why this matters:** Understanding lifecycle helps diagnose real problems — e.g., a thread stuck in `BLOCKED` usually indicates lock contention or a deadlock.

---

## 3. Why NOT create a new thread for every task?

Naively, you could do `new Thread(task).start()` for every incoming request. This works but is expensive at scale:

- **Thread creation cost** — Every OS thread consumes memory (default stack size ~512 KB–1 MB) and requires a system call to create.
- **Unbounded resource usage** — Under heavy load, you could spawn thousands of threads, exhausting memory and CPU, and ultimately crashing the JVM (`OutOfMemoryError: unable to create new native thread`).
- **Excessive context switching** — More threads than CPU cores means the OS spends time switching between threads instead of doing real work.
- **No reuse** — A thread is destroyed after its task finishes, so the creation cost is paid again for the next task.

This is exactly the problem **Thread Pooling** solves.

---

## 4. Thread Pooling

### What it is
A **thread pool** is a fixed or configurable set of worker threads created upfront and **reused** to execute many tasks. Instead of creating a thread per task, tasks are submitted to a queue, and idle worker threads pick them up.

### Why it helps
| Problem without pooling | How pooling fixes it |
|---|---|
| Thread creation overhead per task | Threads are created once and reused |
| Unbounded thread growth | Pool size is capped, protecting system resources |
| Poor throughput under load | A queue smooths out bursts of requests |
| Hard to control concurrency | Pool size directly controls how many tasks run in parallel |

**Analogy:** Instead of hiring and firing a new worker for every customer (expensive, slow), you keep a fixed team of workers on staff who pick up the next customer in line as soon as they're free.

In Java, thread pools are managed through the **`ExecutorService`** framework (`java.util.concurrent`).

---

## 5. ExecutorService

### What it is
`ExecutorService` is a high-level concurrency API that manages a pool of threads and executes asynchronous tasks. It abstracts away thread creation, scheduling, reuse, and lifecycle management.

### Why use it instead of raw threads
- You describe **what** to run (a `Runnable`/`Callable`), not **how** threads are managed.
- Built-in support for task queuing, result retrieval (`Future`), cancellation, and graceful shutdown.
- Centralizes concurrency configuration (pool size, queue, rejection policy) in one place instead of scattering `new Thread()` calls across the codebase.

```java
ExecutorService executor = Executors.newFixedThreadPool(5);

executor.submit(() -> {
    System.out.println("Hello from a pooled thread");
});

executor.shutdown();
```

### Common factory methods (`Executors` class)
| Method | Behavior | When to use / caution |
|---|---|---|
| `newFixedThreadPool(n)` | Fixed number of threads, unbounded queue | Predictable load; **beware**: unbounded queue can grow indefinitely under sustained overload |
| `newCachedThreadPool()` | Creates threads as needed, reuses idle ones, no upper bound | Short-lived bursty tasks; **risky** in production — can spawn unlimited threads |
| `newSingleThreadExecutor()` | One worker thread, tasks run sequentially | When tasks must run in strict order, one at a time |
| `newScheduledThreadPool(n)` | Supports delayed/periodic task execution | Cron-like or recurring jobs |

**Why avoid `Executors` factory methods in production:** Several (`newFixedThreadPool`, `newCachedThreadPool`) use unbounded queues or unbounded thread creation internally, which can silently exhaust memory. Most style guides (and tools like SonarQube) recommend constructing `ThreadPoolExecutor` directly so every parameter is explicit and bounded.

### Common methods
```java
execute(Runnable task)       // fire-and-forget, no result
submit(Runnable/Callable)    // returns a Future
shutdown()                   // graceful shutdown
shutdownNow()                // forceful shutdown
awaitTermination(timeout)    // block until shutdown completes or timeout
```

---

## 6. `execute()` vs `submit()`

| Aspect | `execute()` | `submit()` |
|---|---|---|
| Return type | `void` | `Future<T>` |
| Accepts | Only `Runnable` | `Runnable` and `Callable` |
| Result | Cannot return a result | Can return a result via `Future.get()` |
| Exception handling | Propagated immediately to the thread's `UncaughtExceptionHandler` | Captured and stored inside the `Future`; only surfaces when you call `get()` |

**Why the exception behavior differs (and why it matters):** `execute()` has no way to hand you an exception back later — there's no return value — so the JVM reports it immediately via the uncaught exception handler (usually printed to console). `submit()` gives you a `Future`, so instead of crashing/logging immediately, the exception is safely wrapped and only thrown when *you* call `get()`. This means **a silently swallowed exception in `submit()` is a common bug** — if you never call `get()`, you'll never know the task failed.

```java
// execute(): exception goes straight to console
executor.execute(() -> { throw new RuntimeException("Boom!"); });

// submit(): exception is hidden until get() is called
Future<?> future = executor.submit(() -> { throw new RuntimeException("Boom!"); });
try {
    future.get();
} catch (ExecutionException e) {
    System.out.println("Caught: " + e.getCause());
}
```

**Interview one-liner:** *"Use `execute()` when you don't need a result or a handle on the task. Use `submit()` when you need a return value, want to cancel the task, or need to inspect exceptions via `Future`."*

---

## 7. ThreadPoolExecutor — the engine behind ExecutorService

### What it is
`ThreadPoolExecutor` is the concrete implementation behind most `ExecutorService`s, giving full control over pool behavior.

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    2,                              // corePoolSize
    5,                              // maximumPoolSize
    60, TimeUnit.SECONDS,           // keepAliveTime for idle non-core threads
    new LinkedBlockingQueue<>(10),  // work queue
    new ThreadPoolExecutor.CallerRunsPolicy() // rejection policy
);
```

### Parameters and why each exists
| Parameter | Purpose |
|---|---|
| `corePoolSize` | Minimum number of threads kept alive even when idle — avoids re-creating threads for a steady baseline of work |
| `maximumPoolSize` | Hard ceiling on threads — protects the system from unbounded resource usage during spikes |
| `keepAliveTime` | How long extra (non-core) threads stay idle before being terminated — balances responsiveness to bursts vs. wasting resources |
| `workQueue` | Buffers tasks when all core threads are busy — smooths out bursts without immediately creating new threads or rejecting work |
| `threadFactory` | Customizes thread creation (naming, daemon status, priority) — useful for debugging (named threads show up clearly in thread dumps) |
| `RejectedExecutionHandler` | Defines what happens when both threads and queue are exhausted — prevents unbounded queue growth from crashing the app |

### How a task actually flows through it (and why)

```
New Task Submitted
        │
        ▼
Is a core thread free? ──Yes──▶ Run immediately
        │No
        ▼
Is the queue full? ──No──▶ Add task to queue (wait for a free thread)
        │Yes
        ▼
Can we create a new thread (below maximumPoolSize)? ──Yes──▶ Create thread & run task
        │No
        ▼
Reject the task (per RejectedExecutionHandler)
```

**Why this specific order (core threads → queue → extra threads → reject):** It prioritizes reusing existing threads (cheapest), then buffering in the queue (cheap, but delays execution), then creating new threads only as a last resort before core capacity is exceeded (expensive), and only rejects when truly overloaded. This ordering keeps steady-state performance efficient while still handling bursts gracefully.

---

## 8. Choosing the Right Thread Pool Size

This is one of the most common — and most misunderstood — interview questions.

### CPU-bound tasks (e.g., image processing, encryption, compression, math-heavy computation)

```
Optimal threads ≈ Number of CPU cores  (sometimes cores + 1)
```

**Why:** A CPU core can truly execute only one thread at a time (ignoring hyper-threading). If you create more threads than cores, the OS must **context-switch** between them — saving/restoring registers, program counters, and flushing CPU caches. This costs CPU cycles that produce zero useful work. So for pure computation, matching threads to cores keeps every core saturated with real work and avoids wasted switching overhead.

### I/O-bound tasks (e.g., DB calls, REST APIs, file/network I/O)

```
Threads ≈ CPU Cores × (1 + Wait Time / Compute Time)
```

**Why:** During I/O, a thread is blocked waiting (for the disk, network, or database) and doesn't use the CPU at all. That idle time can be "given" to other threads doing their own work. So you can safely run far more threads than cores, since most of them are asleep at any given moment rather than competing for CPU.

**Example:**
```
CPU Cores = 8
Wait Time = 900ms
Compute Time = 100ms

Threads = 8 × (1 + 900/100) = 8 × 10 = 80
```

**Rule of thumb summary:**
| Task type | Sizing guidance | Reason |
|---|---|---|
| CPU-bound | ≈ number of cores | Only N threads can truly run in parallel on N cores |
| I/O-bound | Much larger than cores | Threads spend most time waiting, not computing |

---

## 9. What Happens When a Thread Pool Is "Full"?

"Full" means: all threads are busy **and** the queue has no room **and** `maximumPoolSize` has been reached. At that point, `ThreadPoolExecutor` invokes a **Rejection Policy** to decide the fate of the new task.

### The four built-in rejection policies

| Policy | Behavior | Why you'd choose it |
|---|---|---|
| `AbortPolicy` (default) | Throws `RejectedExecutionException` | Fail fast and loud — forces the caller to notice and handle overload explicitly |
| `CallerRunsPolicy` | The submitting thread itself runs the task | Provides natural **backpressure**: the caller (e.g., a web server thread) is slowed down instead of overwhelming the pool, which indirectly throttles incoming requests |
| `DiscardPolicy` | Silently drops the new task | Acceptable only when occasionally losing a task is tolerable (e.g., non-critical logging/metrics) |
| `DiscardOldestPolicy` | Drops the oldest queued task, then queues the new one | Prioritizes fresh/recent work over stale queued work (e.g., latest sensor reading matters more than an old one) |

**Why rejection policies exist at all:** Without one, an overloaded system would either queue tasks forever (eventually running out of memory) or silently create unlimited threads (crashing the JVM). A rejection policy is the deliberate, explicit "circuit breaker" for overload.

---

## 10. Runnable vs Callable

| Runnable | Callable |
|---|---|
| `run()` returns nothing | `call()` returns a value |
| Cannot throw checked exceptions | Can throw checked exceptions |
| Used with `execute()` or `submit()` | Used only with `submit()` |

```java
Runnable task = () -> System.out.println("No result, no checked exceptions");
Callable<Integer> callableTask = () -> 100; // can also throw checked exceptions
```

**Why both exist:** `Runnable` predates Java's concurrency utilities (it's tied to `Thread` itself) and was designed for fire-and-forget work. `Callable` was introduced later specifically to support tasks that produce a result or need to signal failure via checked exceptions — something `Runnable`'s `void run()` signature can't express.

---

## 11. Future

### What it is
`Future<T>` represents the eventual result of an asynchronous computation submitted via `submit()`.

```java
Future<Integer> future = executor.submit(() -> 10 + 20);
Integer result = future.get(); // blocks until the task completes
```

### Key methods
```java
future.get();          // blocks until result is available (or throws)
future.get(timeout, unit); // blocks with a timeout
future.cancel(true);   // attempt to cancel the task
future.isDone();
future.isCancelled();
```

### Limitations (why `CompletableFuture` was introduced)
- `get()` **blocks** the calling thread — defeats some of the purpose of asynchrony if used carelessly.
- No built-in way to **chain** ("when this finishes, then do that") or **combine** multiple futures.
- No way to react to completion without blocking (no callbacks).

---

## 12. CompletableFuture

| Future | CompletableFuture |
|---|---|
| Blocking (`get()`) | Supports non-blocking callbacks |
| No chaining | `thenApply`, `thenAccept`, `thenCompose`, etc. |
| Manual combination of results | Built-in combinators (`thenCombine`, `allOf`, `anyOf`) |
| Cannot be manually completed | Can be completed manually (`complete()`) |

```java
CompletableFuture
    .supplyAsync(() -> "Hello")
    .thenApply(String::toUpperCase)
    .thenAccept(System.out::println);
```

**Why it matters:** It enables truly asynchronous pipelines — e.g., "fetch user, then fetch their orders, then combine with pricing, then send response" — without any thread ever blocking on `get()`, which is essential for scalable, non-blocking server architectures.

---

## 13. BlockingQueue

### What it is
A thread-safe queue used internally by thread pools to hold tasks waiting for execution. Worker threads pull tasks from it.

### Why "blocking" specifically
A regular queue would throw an exception or return `null`/`false` if you try to remove from an empty queue or add to a full one. A `BlockingQueue` instead makes the calling thread **wait** until the queue has data (for removal) or space (for insertion) — exactly the behavior a producer-consumer system like a thread pool needs, without manual `wait()`/`notify()` code.

### Common implementations
| Implementation | Characteristic | Typical use |
|---|---|---|
| `LinkedBlockingQueue` | Optionally bounded, linked-node based | General-purpose pools |
| `ArrayBlockingQueue` | Fixed-size, array-based | When you want a hard, predictable capacity |
| `PriorityBlockingQueue` | Orders elements by priority, not FIFO | When some tasks must run before others |
| `SynchronousQueue` | Zero capacity — a handoff directly from producer to consumer | `newCachedThreadPool()`; forces immediate thread creation instead of queuing |

---

## 14. shutdown() vs shutdownNow()

| `shutdown()` | `shutdownNow()` |
|---|---|
| Stops accepting new tasks | Stops accepting new tasks |
| Lets already-queued and running tasks finish | Attempts to stop actively running tasks (via interrupt) and returns unstarted queued tasks |
| Graceful | Immediate/forceful |

```java
executor.shutdown();      // finish what's running/queued, then stop
executor.shutdownNow();   // stop ASAP, return List<Runnable> of tasks that never started
```

**Why both exist:** Graceful shutdown (`shutdown()`) is the right default — it avoids losing in-flight work. `shutdownNow()` is for emergencies (e.g., app is force-closing, or tasks are hung) where you'd rather cut losses than wait. A common production pattern combines both:

```java
executor.shutdown();
try {
    if (!executor.awaitTermination(30, TimeUnit.SECONDS)) {
        executor.shutdownNow();
    }
} catch (InterruptedException e) {
    executor.shutdownNow();
    Thread.currentThread().interrupt();
}
```

---

## 15. Monitoring Thread Pool Health

Key metrics `ThreadPoolExecutor` exposes:

| Metric | What it tells you |
|---|---|
| `getActiveCount()` | Threads currently executing tasks |
| `getPoolSize()` | Current number of threads in the pool |
| `getQueue().size()` | Tasks waiting to be picked up |
| `getCompletedTaskCount()` | Total tasks finished so far |
| `getLargestPoolSize()` | Peak number of threads ever active — helps right-size `maximumPoolSize` |
| Rejection count (via a custom handler) | How often tasks are being dropped/rejected — signals under-provisioning |

**Why monitor this at all:** A pool that looks "fine" from the outside can be silently accumulating a huge backlog in its queue (growing latency) or constantly hitting its rejection policy (dropped work). Tools like **Micrometer**, **Spring Boot Actuator**, **Prometheus**, and **Grafana** are commonly wired up to alert on these metrics before they become outages.

---

## 16. Thread Safety Essentials (often paired with pooling questions)

Since pooled threads share the same JVM memory, running tasks concurrently reintroduces classic concurrency hazards. Key concepts:

| Concept | What it is | Why it matters |
|---|---|---|
| **Race condition** | Two threads access/modify shared state without coordination, producing incorrect results depending on timing | The most common source of subtle, hard-to-reproduce bugs in multithreaded code |
| **`synchronized`** | Keyword that lets only one thread execute a block/method on a given object at a time | Simplest way to enforce mutual exclusion around shared state |
| **`volatile`** | Ensures a variable's writes are immediately visible to other threads (no caching in CPU registers/local cache) | Solves visibility issues, but **not** atomicity — doesn't replace locks for compound operations like `i++` |
| **`java.util.concurrent.atomic` (e.g., `AtomicInteger`)** | Lock-free, thread-safe operations on single variables using CPU-level compare-and-swap | Faster than `synchronized` for simple counters/flags since it avoids blocking |
| **Deadlock** | Two or more threads wait on each other's locks forever | Understanding lock ordering avoids it — always acquire multiple locks in a consistent global order |
| **`ReentrantLock`** | Explicit lock object with more flexibility than `synchronized` (tryLock, fairness, interruptible locking) | Needed when you require timeouts, fairness policies, or non-block-structured locking |

**Why this belongs alongside thread pooling:** Pooling controls *how many* threads run concurrently, but doesn't automatically make your task code safe. If two pooled tasks mutate a shared object without synchronization, you'll get race conditions regardless of pool size.

---

## 17. Designing a Thread Pool for a Real REST API (Best Practices)

1. **Use `ThreadPoolExecutor` explicitly**, not `Executors.newFixedThreadPool()` — so core size, max size, and queue capacity are all deliberate and bounded.
2. **Size the pool based on workload type** — CPU-bound vs. I/O-bound (see Section 8).
3. **Use a bounded queue** (e.g., `ArrayBlockingQueue`) — an unbounded queue can hide overload until memory runs out.
4. **Pick a sensible rejection policy** — `CallerRunsPolicy` is popular for APIs because it naturally throttles the caller instead of dropping requests.
5. **Name your threads** via a custom `ThreadFactory` — makes thread dumps and logs far easier to debug ("payment-pool-3" vs. "pool-1-thread-7").
6. **Monitor pool metrics in production** — active threads, queue size, rejection count.
7. **Shut down gracefully on application stop** — use `shutdown()` + `awaitTermination()`, falling back to `shutdownNow()`.
8. **Avoid unbounded thread creation** (`newCachedThreadPool()` in high-traffic services) — it can spawn a thread per request under load spikes.

---

## Quick-Reference Interview Summary

- Use `ExecutorService`/`ThreadPoolExecutor` instead of manually creating threads — reuse beats recreation.
- `execute()` = no result, exceptions surface immediately; `submit()` = returns `Future`, exceptions are deferred until `get()`.
- CPU-bound → threads ≈ cores; I/O-bound → threads ≈ cores × (1 + wait/compute).
- Rejection policies (`Abort`, `CallerRuns`, `Discard`, `DiscardOldest`) exist to handle overload predictably instead of crashing or silently dropping work forever.
- `Future` blocks and can't chain; `CompletableFuture` is non-blocking and composable.
- `shutdown()` is graceful; `shutdownNow()` is forceful — combine both for robust app shutdown.
- Pooling controls *concurrency level*; it does **not** make shared-state access thread-safe — you still need `synchronized`, locks, or atomics for that.
- Always monitor pool metrics in production; a "working" pool can still be silently overloaded.
