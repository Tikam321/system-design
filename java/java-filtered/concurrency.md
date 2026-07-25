# Handling Concurrency Issues in Java

Concurrency issues occur when multiple threads access shared resources simultaneously, which can lead to:

- Race Conditions
- Deadlocks
- Data Inconsistency
- Thread Starvation
- Visibility Problems

Java provides multiple mechanisms to handle concurrency safely and efficiently.

---

# 1. Using synchronized

The `synchronized` keyword ensures that only one thread can access the critical section at a time.

## Example

```java
class Counter {
    int count = 0;

    synchronized void increment() {
        count++;
    }
}
```

## Benefits

- Prevents race conditions
- Easy to implement
- Built-in monitor locking

## Drawbacks

- Can reduce performance due to thread blocking

---

# 2. Synchronized Block

Instead of locking the whole method, we can lock only the critical section.

## Example

```java
class Counter {
    int count = 0;
    Object lock = new Object();

    void increment() {
        synchronized (lock) {
            count++;
        }
    }
}
```

## Benefits

- Better performance
- Smaller lock scope
- More control

---

# 3. Atomic Classes

Java provides atomic classes in the `java.util.concurrent.atomic` package.

## Example

```java
import java.util.concurrent.atomic.AtomicInteger;

class Counter {
    AtomicInteger count = new AtomicInteger(0);

    void increment() {
        count.incrementAndGet();
    }
}
```

## Benefits

- Thread-safe without explicit locking
- Faster than synchronized
- Uses CAS (Compare And Swap)

## Common Atomic Classes

- AtomicInteger
- AtomicLong
- AtomicReference

---

# 4. ReentrantLock

Provides advanced locking mechanisms compared to synchronized.

## Example

```java
import java.util.concurrent.locks.ReentrantLock;

class Counter {
    int count = 0;
    ReentrantLock lock = new ReentrantLock();

    void increment() {
        lock.lock();

        try {
            count++;
        } finally {
            lock.unlock();
        }
    }
}
```

## Benefits

- tryLock()
- Fair locking
- Interruptible locks
- Better flexibility

---

# 5. ReadWriteLock

Useful in read-heavy systems.

## Example

```java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

ReadWriteLock lock = new ReentrantReadWriteLock();
```

## Benefits

- Multiple readers allowed simultaneously
- Only one writer allowed
- Improves performance

## Common Use Cases

- Cache systems
- Configuration management systems

---

# 6. Concurrent Collections

Normal collections are not thread-safe.

## Non Thread-Safe Collections

```java
HashMap
ArrayList
```

## Thread-Safe Concurrent Collections

```java
ConcurrentHashMap
CopyOnWriteArrayList
BlockingQueue
```

## Example

```java
import java.util.concurrent.ConcurrentHashMap;

ConcurrentHashMap<String, Integer> map =
    new ConcurrentHashMap<>();
```

## Benefits

- Better scalability
- Internal fine-grained locking
- High performance under concurrency

---

# 7. ExecutorService (Thread Pooling)

Instead of manually creating threads, use thread pools.

## Example

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

ExecutorService executor =
    Executors.newFixedThreadPool(10);

executor.submit(() -> {
    System.out.println("Task Executed");
});
```

## Benefits

- Reuses threads
- Reduces thread creation overhead
- Better resource management

---

# 8. volatile Keyword

Ensures visibility of shared variables across threads.

## Example

```java
volatile boolean running = true;
```

## Benefits

- Latest value visible to all threads
- Prevents CPU cache inconsistency

## Use Cases

- Status flags
- Feature toggles
- Shutdown signals

## Limitation

`volatile` does NOT provide atomicity.

---

# 9. Deadlock Prevention

Deadlock occurs when threads wait indefinitely for each other’s locks.

## Prevention Techniques

- Maintain proper lock ordering
- Avoid nested locks
- Use tryLock()
- Keep lock scope small

---

# 10. Immutability

Immutable objects are naturally thread-safe.

## Example

```java
final class User {

    private final String name;

    User(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

## Benefits

- No synchronization required
- Thread-safe by design
- Easier to maintain

---

# Real Production Approach

In production-grade microservices:

- Use `ConcurrentHashMap` for caching
- Use `ExecutorService` for async tasks
- Use `AtomicInteger` for counters
- Avoid excessive synchronized blocks
- Prefer stateless services
- Use distributed locking for multi-instance systems

---

# Interview Ready Answer

> To handle concurrency issues in Java, I use synchronization techniques like `synchronized`, `ReentrantLock`, and atomic classes depending on the use case. For shared collections, I prefer concurrent collections like `ConcurrentHashMap`. I also use `ExecutorService` for thread pooling and asynchronous task execution. To improve scalability, I minimize shared mutable state using immutability and stateless design principles. Additionally, I avoid deadlocks through proper lock ordering and use `volatile` when only visibility is required.
