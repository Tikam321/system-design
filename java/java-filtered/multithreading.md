#  what is threading
- Thread Pooling is a technique where a fixed or configurable set of worker threads is created in advance and reused to execute multiple tasks.
-  Instead of creating a new thread for every request, tasks are submitted to a queue, and an available thread processes them.
-  This reduces thread creation overhead, improves performance, limits resource usage, and enables applications to handle many concurrent tasks efficiently.
-  In Java, thread pools are managed using the ExecutorService framework.

# 2. what is executor service 
- ExecutorService is a high-level Java concurrency framework that manages a pool of threads and executes asynchronous tasks.
- ExecutorService is a Java concurrency framework that manages thread pools and executes asynchronous tasks. It abstracts thread creation, scheduling, reuse, and lifecycle management, making concurrent programming more efficient and scalable.

- # Thread Pooling - Interview Questions & Answers

## 1. What is Thread Pooling?

### Answer

A **Thread Pool** is a collection of pre-created, reusable worker threads that execute multiple tasks. Instead of creating a new thread for every task, tasks are submitted to the pool and executed by available threads.

### Benefits

* Reuses threads instead of creating new ones.
* Reduces thread creation overhead.
* Improves application performance.
* Controls resource utilization.
* Prevents excessive thread creation.

---

# 2. What is ExecutorService?

### Answer

`ExecutorService` is a Java concurrency framework that manages thread pools and executes asynchronous tasks. It abstracts thread creation, scheduling, execution, and lifecycle management.

Instead of manually creating threads:

```java
new Thread(() -> {
    // task
}).start();
```

Use:

```java
ExecutorService executor = Executors.newFixedThreadPool(5);

executor.submit(() -> {
    System.out.println("Hello");
});
```

### Common Methods

```java
execute(Runnable task)
submit(Runnable task)
submit(Callable<T> task)
shutdown()
shutdownNow()
awaitTermination()
```

### Interview Answer

> ExecutorService is a high-level concurrency framework that manages a pool of reusable threads and executes asynchronous tasks efficiently.

---

# 3. Difference between execute() and submit()

| execute()                        | submit()                        |
| -------------------------------- | ------------------------------- |
| Returns `void`                   | Returns `Future<T>`             |
| Accepts only `Runnable`          | Accepts `Runnable` & `Callable` |
| Cannot return result             | Can return result               |
| Exception immediately propagated | Exception stored in `Future`    |

### execute()

```java
executor.execute(() -> {
    System.out.println("Hello");
});
```

### submit()

```java
Future<Integer> future = executor.submit(() -> 100);

System.out.println(future.get());
```

### Interview Answer

> Use `execute()` when you don't need a result. Use `submit()` when you need a result, want to cancel the task, or handle exceptions through `Future`.

# Exception Handling: `execute()` vs `submit()`

## `execute()`

* Returns **void**.
* If the task throws an exception, it is **immediately propagated** to the thread's `UncaughtExceptionHandler`.
* The exception is typically printed to the console.

### Example

```java
ExecutorService executor = Executors.newSingleThreadExecutor();

executor.execute(() -> {
    throw new RuntimeException("Something went wrong!");
});

executor.shutdown();
```

**Output**

```text
Exception in thread "pool-1-thread-1"
java.lang.RuntimeException: Something went wrong!
```

---

## `submit()`

* Returns a **Future**.
* If the task throws an exception, it is **captured and stored inside the `Future`**.
* The exception is thrown only when `future.get()` is called.

### Example

```java
ExecutorService executor = Executors.newSingleThreadExecutor();

Future<?> future = executor.submit(() -> {
    throw new RuntimeException("Something went wrong!");
});

try {
    future.get();
} catch (ExecutionException e) {
    System.out.println(e.getCause());
}

executor.shutdown();
```

**Output**

```text
java.lang.RuntimeException: Something went wrong!
```

---

## Execution Flow

### `execute()`

```text
Task
  ↓
Exception Thrown
  ↓
UncaughtExceptionHandler
  ↓
Stack Trace Printed
```

### `submit()`

```text
Task
  ↓
Exception Thrown
  ↓
Stored in Future
  ↓
future.get()
  ↓
ExecutionException
```

---

## Interview Answer

> **`execute()`** immediately propagates unchecked exceptions to the executing thread's `UncaughtExceptionHandler`.
> **`submit()`** captures exceptions inside the returned `Future`, and they are rethrown as an `ExecutionException` only when `future.get()` is invoked.

---

# 4. What happens when the Thread Pool is Full?

When:

* All worker threads are busy.
* The task queue is full.
* Maximum pool size has been reached.

The executor applies a **Rejection Policy**.

Execution Flow:

```
New Task
   |
Core Thread Available?
   |
  No
   |
Queue Full?
   |
  No → Add to Queue
   |
Queue Full?
   |
Can create Max Thread?
   |
Yes → Create New Thread
   |
Already at Maximum?
   |
Reject Task
```

---

# 5. What is ThreadPoolExecutor?

### Answer

`ThreadPoolExecutor` is the core implementation of `ExecutorService` and provides complete control over thread pool behavior.

```java
ThreadPoolExecutor executor =
new ThreadPoolExecutor(
    2,
    5,
    60,
    TimeUnit.SECONDS,
    new LinkedBlockingQueue<>(10)
);
```

### Constructor Parameters

* `corePoolSize`
* `maximumPoolSize`
* `keepAliveTime`
* `TimeUnit`
* `BlockingQueue`
* `ThreadFactory`
* `RejectedExecutionHandler`

### Interview Answer

> ThreadPoolExecutor provides fine-grained control over thread creation, queue management, thread lifetime, and rejection policies.

---

# 6. How do you decide the optimal thread pool size?

### CPU-Bound Tasks

Examples:

* Image processing
* Encryption
* Compression

Formula:

```
Thread Pool Size ≈ Number of CPU Cores
```

### IO-Bound Tasks

Examples:

* Database calls
* REST APIs
* Kafka
* File operations

Formula:

```
Threads = CPU Cores × (1 + Wait Time / Compute Time)
```

### Example

```
CPU Cores = 8
Wait Time = 900ms
Compute Time = 100ms

Threads = 8 × (1 + 9)
        = 80
```

### Interview Answer

> CPU-bound tasks typically use a thread pool close to the number of CPU cores, while I/O-bound tasks can benefit from a larger pool because threads spend much of their time waiting.

# Why is Thread Pool Size ≈ Number of CPU Cores for CPU-Bound Tasks?

## What are CPU-Bound Tasks?

CPU-bound tasks spend most of their time performing computations rather than waiting for I/O.

**Examples:**

* Image processing
* Encryption/Decryption
* Data compression
* Mathematical calculations

---

## Why choose the number of CPU cores?

A CPU can execute only **one thread per core at a time** (ignoring technologies like Hyper-Threading).

For example, on a **4-core CPU**:

```text
Core 1 → Thread 1
Core 2 → Thread 2
Core 3 → Thread 3
Core 4 → Thread 4
```

All cores remain busy, maximizing CPU utilization.

---

## What if we create more threads?

Example: **4 CPU cores, 20 threads**

```text
20 Threads
    │
    ▼
Scheduler
    │
    ▼
4 CPU Cores
```

Only **4 threads** can run simultaneously.

The remaining **16 threads** wait in the ready queue.

The operating system repeatedly switches between threads (**context switching**), which introduces overhead without increasing throughput.

---

## Why is Context Switching Expensive?

When switching from one thread to another, the CPU must:

* Save the current thread's state (registers, program counter).
* Load the next thread's state.
* Flush/reload CPU caches if necessary.

This overhead consumes CPU cycles that could have been used for actual computation.

---

## Rule of Thumb

```text
CPU-Bound Tasks

Thread Pool Size ≈ Number of CPU Cores
```

This keeps all cores busy while minimizing unnecessary context switching.

---

## Interview Answer

> For CPU-bound tasks, the thread pool size is typically set close to the number of CPU cores because these tasks continuously use the CPU. Creating more threads than available cores doesn't improve performance, as only one thread can execute per core at a time. Extra threads simply increase context switching overhead, reducing overall efficiency.

---

# 7. What is a Rejection Policy?

When the thread pool and queue are both full, `ThreadPoolExecutor` decides how to handle new tasks using a rejection policy.

## AbortPolicy (Default)

```java
new ThreadPoolExecutor.AbortPolicy();
```

Throws:

```
RejectedExecutionException
```

---

## CallerRunsPolicy

The thread submitting the task executes it.

```
Main Thread
     |
Executes Task
```

Useful for applying backpressure.

---

## DiscardPolicy

Silently discards the new task.

```
Task Lost
```

---

## DiscardOldestPolicy

Removes the oldest queued task and inserts the new task.

```
Queue

A
B
C

↓

Remove A

↓

Insert D
```

### Interview Answer

> A rejection policy defines how a ThreadPoolExecutor handles new tasks when the thread pool and queue are both full.

---

# 8. Runnable vs Callable

| Runnable                        | Callable                     |
| ------------------------------- | ---------------------------- |
| No return value                 | Returns a value              |
| Cannot throw checked exceptions | Can throw checked exceptions |
| execute() / submit()            | submit()                     |

Runnable:

```java
Runnable task = () -> {
    System.out.println("Running...");
};
```

Callable:

```java
Callable<Integer> task = () -> 100;
```

---

# 9. What is Future?

`Future` represents the result of an asynchronous computation.

Example:

```java
Future<Integer> future = executor.submit(() -> 10);

Integer result = future.get();
```

### Common Methods

```java
future.get();
future.cancel(true);
future.isDone();
future.isCancelled();
```

### Limitation

* `get()` blocks the calling thread.
* Cannot easily chain multiple asynchronous tasks.

---

# 10. CompletableFuture vs Future

| Future                | CompletableFuture      |
| --------------------- | ---------------------- |
| Blocking              | Non-blocking           |
| Cannot chain tasks    | Supports task chaining |
| Manual combination    | Built-in composition   |
| Limited functionality | Rich asynchronous API  |

Example:

```java
CompletableFuture
    .supplyAsync(() -> "Hello")
    .thenApply(String::toUpperCase)
    .thenAccept(System.out::println);
```

---

# 11. What is a BlockingQueue?

A `BlockingQueue` stores tasks waiting to be executed.

Worker threads retrieve tasks from the queue.

Common implementations:

* LinkedBlockingQueue
* ArrayBlockingQueue
* PriorityBlockingQueue
* SynchronousQueue

---

# 12. shutdown() vs shutdownNow()

| shutdown()                | shutdownNow()                  |
| ------------------------- | ------------------------------ |
| Stops accepting new tasks | Stops accepting new tasks      |
| Completes queued tasks    | Attempts to stop running tasks |
| Graceful shutdown         | Immediate shutdown             |

Example:

```java
executor.shutdown();
```

```java
executor.shutdownNow();
```

---

# 13. How does ThreadPoolExecutor work internally?

Execution flow:

```
Submit Task
      |
Core Thread Available?
      |
Yes → Execute
      |
No
      |
Queue Available?
      |
Yes → Queue Task
      |
No
      |
Maximum Thread Available?
      |
Yes → Create Thread
      |
No
      |
Reject Task
```

---

# 14. How do you monitor Thread Pool health?

Common metrics:

* Active Thread Count
* Pool Size
* Queue Size
* Completed Task Count
* Largest Pool Size
* Task Rejection Count

Useful with:

* Spring Boot Actuator
* Micrometer
* Prometheus
* Grafana

---

# 15. What happens if a task throws an exception?

### execute()

```java
executor.execute(() -> {
    throw new RuntimeException();
});
```

Exception is propagated to the thread's uncaught exception handler.

---

### submit()

```java
Future<?> future = executor.submit(() -> {
    throw new RuntimeException();
});

future.get();
```

The exception is wrapped inside an `ExecutionException` and is thrown when `get()` is called.

---

# 16. How would you design a Thread Pool for a REST API?

### Best Practices

* Use a fixed-size `ThreadPoolExecutor`.
* Size the pool based on CPU and I/O characteristics.
* Use a bounded queue.
* Configure a suitable rejection policy (e.g., `CallerRunsPolicy`).
* Monitor pool metrics.
* Avoid creating threads manually.
* Gracefully shut down the executor during application shutdown.

---

# Interview Summary

## Key Takeaways

* Use `ExecutorService` instead of manually creating threads.
* Prefer `ThreadPoolExecutor` for production applications.
* Choose thread pool size based on workload (CPU-bound vs I/O-bound).
* Understand the difference between `execute()` and `submit()`.
* Know all four rejection policies.
* Understand `Future` and `CompletableFuture`.
* Monitor thread pools in production to avoid bottlenecks.
