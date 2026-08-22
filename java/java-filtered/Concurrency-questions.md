# Hands-On Java Concurrency — Implementation Practice

For each problem: **don't read the solution first.** Read the problem + constraints, try coding it yourself for 15–20 min, then check against the solution and the "why" notes. This mirrors how Round 2 will actually feel.

---

## Problem 1: Producer-Consumer (using `BlockingQueue`)

### Requirements
- One or more producer threads generate items and put them in a shared buffer.
- One or more consumer threads take items from the buffer and process them.
- If the buffer is full, producers must wait. If empty, consumers must wait.

### Steps to implement
1. Create a shared `BlockingQueue<Integer>` with a fixed capacity (e.g., `ArrayBlockingQueue(10)`).
2. Producer: loop, generate a value, call `queue.put(value)` (this blocks automatically if full).
3. Consumer: loop, call `queue.take()` (blocks automatically if empty), process the value.
4. Run producer(s) and consumer(s) as separate threads via `ExecutorService`.
5. Add a "poison pill" mechanism to signal consumers to stop cleanly.

### Solution
```java
import java.util.concurrent.*;

public class ProducerConsumerBQ {
    private static final int POISON_PILL = Integer.MIN_VALUE;

    public static void main(String[] args) throws InterruptedException {
        BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(5);
        ExecutorService executor = Executors.newFixedThreadPool(2);

        Runnable producer = () -> {
            try {
                for (int i = 1; i <= 10; i++) {
                    queue.put(i);
                    System.out.println("Produced: " + i);
                    Thread.sleep(100);
                }
                queue.put(POISON_PILL); // signal consumer to stop
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };

        Runnable consumer = () -> {
            try {
                while (true) {
                    int value = queue.take();
                    if (value == POISON_PILL) break;
                    System.out.println("Consumed: " + value);
                    Thread.sleep(150);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };

        executor.submit(producer);
        executor.submit(consumer);
        executor.shutdown();
        executor.awaitTermination(5, TimeUnit.SECONDS);
    }
}
```

### Follow-up they may ask
- "Now implement this WITHOUT `BlockingQueue`, using only `wait()`/`notify()`." → **Problem 2 below.**
- What happens with multiple producers and one poison pill? (Answer: need one poison pill per consumer, or a shared "done" flag.)

---

## Problem 2: Bounded Buffer from Scratch (manual `wait()`/`notify()`)

### Requirements
Same as Problem 1, but implement the blocking behavior yourself — no `BlockingQueue`.

### Steps to implement
1. Create a class wrapping a `Queue<Integer>` (e.g., `LinkedList`) and a max `capacity`.
2. `put(item)`: acquire the object's intrinsic lock (`synchronized`), while queue is full call `wait()`. Once space available, add item, then call `notifyAll()`.
3. `take()`: acquire lock, while queue is empty call `wait()`. Once an item exists, remove it, call `notifyAll()`, return item.
4. **Critical:** always use `while`, never `if`, to re-check the condition after waking up (guards against spurious wakeups).

### Solution
```java
import java.util.LinkedList;
import java.util.Queue;

public class BoundedBuffer {
    private final Queue<Integer> queue = new LinkedList<>();
    private final int capacity;

    public BoundedBuffer(int capacity) {
        this.capacity = capacity;
    }

    public synchronized void put(int value) throws InterruptedException {
        while (queue.size() == capacity) {
            wait(); // release lock, wait for consumer to make space
        }
        queue.add(value);
        System.out.println("Produced: " + value);
        notifyAll(); // wake up any waiting consumers
    }

    public synchronized int take() throws InterruptedException {
        while (queue.isEmpty()) {
            wait(); // release lock, wait for producer to add something
        }
        int value = queue.poll();
        System.out.println("Consumed: " + value);
        notifyAll(); // wake up any waiting producers
        return value;
    }
}
```

### Why `while` not `if` (they WILL ask this)
When a thread is woken up by `notifyAll()`, it doesn't automatically get the lock or guarantee the condition still holds — another thread might have grabbed the lock first and changed the state again. `while` re-checks the condition every time the thread wakes up, before proceeding. Using `if` is a classic, subtle bug.

### Follow-up
- Why `notifyAll()` instead of `notify()`? (`notify()` wakes only one arbitrary waiting thread — if that thread happens to be another producer waiting for space, while a consumer was what you actually needed to wake, you get a stuck/starved thread. `notifyAll()` is safer by default, though less efficient.)
- Convert this to use `ReentrantLock` + `Condition` objects instead of `synchronized`/`wait`/`notify` (two separate `Condition`s: `notFull` and `notEmpty`, so you wake only the relevant side).

---

## Problem 3: Thread-Safe LRU Cache

### Requirements
- `get(key)` — returns value, marks key as recently used.
- `put(key, value)` — inserts/updates; if capacity exceeded, evicts the least recently used entry.
- Must be thread-safe under concurrent reads/writes.

### Steps to implement
1. Use `LinkedHashMap` with `accessOrder = true` — it already maintains recency order on access.
2. Override `removeEldestEntry()` to auto-evict when size exceeds capacity.
3. Wrap all public methods in `synchronized` (simplest correct approach) — or use a `ReentrantReadWriteLock` if reads vastly outnumber writes (though LRU's `get()` also mutates order, so a plain read lock isn't safe here — that's a good discussion point in interview).

### Solution
```java
import java.util.LinkedHashMap;
import java.util.Map;

public class LRUCache<K, V> {
    private final int capacity;
    private final LinkedHashMap<K, V> map;

    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.map = new LinkedHashMap<>(capacity, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
                return size() > LRUCache.this.capacity;
            }
        };
    }

    public synchronized V get(K key) {
        return map.getOrDefault(key, null);
    }

    public synchronized void put(K key, V value) {
        map.put(key, value);
    }

    public synchronized String snapshot() {
        return map.toString();
    }
}
```

### Why `synchronized` on `get()` too (common mistake)
`get()` on an access-ordered `LinkedHashMap` internally *mutates* the linked list to move the entry to the end — it's not a pure read. If you let `get()` run without synchronization while `put()` is happening, you can corrupt the internal linked list. This is a great point to bring up proactively — shows depth.

### Follow-up
- Implement this **without** `LinkedHashMap`, using a `HashMap` + manual doubly linked list — this is the "real" version interviewers often want, especially for Round 3 (LLD).
- How would you make it lock-free / higher throughput? (`ConcurrentHashMap` + segment-based locking, or `Caffeine`-style approach with a concurrent eviction policy — mention this exists, you're not expected to build it live.)

---

## Problem 4: Rate Limiter (Token Bucket)

### Requirements
- `allowRequest()` returns `true` if a request is allowed under the configured rate, `false` otherwise.
- Must be thread-safe — many threads call `allowRequest()` concurrently.

### Steps to implement
1. Maintain `tokens` (current available), `capacity` (max tokens), `refillRatePerSecond`, and `lastRefillTimestamp`.
2. On each call, first "refill": compute elapsed time since last refill, add `elapsed * refillRate` tokens (capped at `capacity`).
3. If `tokens >= 1`, decrement and allow; else reject.
4. Guard the whole refill+check+decrement sequence atomically (it's a compound operation, so `synchronized` or a lock is needed — `AtomicLong` alone isn't enough for the multi-field update).

### Solution
```java
import java.util.concurrent.locks.ReentrantLock;

public class TokenBucketRateLimiter {
    private final long capacity;
    private final double refillRatePerSecond;
    private double availableTokens;
    private long lastRefillTimestamp;
    private final ReentrantLock lock = new ReentrantLock();

    public TokenBucketRateLimiter(long capacity, double refillRatePerSecond) {
        this.capacity = capacity;
        this.refillRatePerSecond = refillRatePerSecond;
        this.availableTokens = capacity;
        this.lastRefillTimestamp = System.nanoTime();
    }

    public boolean allowRequest() {
        lock.lock();
        try {
            refill();
            if (availableTokens >= 1) {
                availableTokens -= 1;
                return true;
            }
            return false;
        } finally {
            lock.unlock();
        }
    }

    private void refill() {
        long now = System.nanoTime();
        double elapsedSeconds = (now - lastRefillTimestamp) / 1_000_000_000.0;
        double tokensToAdd = elapsedSeconds * refillRatePerSecond;
        if (tokensToAdd > 0) {
            availableTokens = Math.min(capacity, availableTokens + tokensToAdd);
            lastRefillTimestamp = now;
        }
    }
}
```

### Why not `AtomicLong` / lock-free here
Token bucket needs to atomically read-and-update **two related fields** (`availableTokens` and `lastRefillTimestamp`) together — a single `AtomicLong` can only make one variable atomic, not a compound multi-field transaction. You'd need `AtomicReference` to an immutable state object + a CAS loop to go lock-free correctly — worth mentioning as an optimization if they push on performance.

### Follow-up
- Implement **sliding window** rate limiting instead of token bucket — discuss trade-offs (token bucket allows bursts up to capacity; sliding window is stricter/smoother).
- How would you make this work across multiple servers (distributed rate limiting)? (Redis + Lua script, or a centralized counter service — good to mention even briefly.)

---

## Problem 5: Custom Thread Pool from Scratch

### Requirements
- Fixed number of worker threads.
- `submit(Runnable task)` queues work for any available worker.
- Graceful `shutdown()`.

### Steps to implement
1. Hold a `BlockingQueue<Runnable>` as the task queue.
2. On construction, start N worker threads; each runs a loop: `take()` a task from the queue, execute it, repeat.
3. `submit()` just does `queue.put(task)`.
4. `shutdown()`: set a flag, then interrupt all workers so they exit their `take()` call and check the flag.

### Solution
```java
import java.util.List;
import java.util.ArrayList;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;

public class SimpleThreadPool {
    private final BlockingQueue<Runnable> taskQueue = new LinkedBlockingQueue<>();
    private final List<WorkerThread> workers = new ArrayList<>();
    private volatile boolean isShutdown = false;

    public SimpleThreadPool(int numThreads) {
        for (int i = 0; i < numThreads; i++) {
            WorkerThread worker = new WorkerThread("Worker-" + i);
            workers.add(worker);
            worker.start();
        }
    }

    public void submit(Runnable task) {
        if (isShutdown) {
            throw new IllegalStateException("Pool is shut down, cannot accept new tasks");
        }
        try {
            taskQueue.put(task);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    public void shutdown() {
        isShutdown = true;
        for (WorkerThread worker : workers) {
            worker.interrupt();
        }
    }

    private class WorkerThread extends Thread {
        WorkerThread(String name) { super(name); }

        @Override
        public void run() {
            while (!isShutdown || !taskQueue.isEmpty()) {
                try {
                    Runnable task = taskQueue.poll(500, java.util.concurrent.TimeUnit.MILLISECONDS);
                    if (task != null) task.run();
                } catch (InterruptedException e) {
                    if (isShutdown) break;
                }
            }
        }
    }
}
```

### Follow-up (very likely — this is a design extension, not just coding)
- Add support for `maximumPoolSize` and dynamic thread creation like real `ThreadPoolExecutor`.
- Add a `RejectedExecutionHandler` when the queue is bounded and full.
- Add `Future<T>` support to `submit()` so callers can get results back — this means wrapping tasks in a `FutureTask`.

---

## Problem 6: Print Numbers in Order Across Multiple Threads

### Requirements
3 threads print `1, 2, 3` repeatedly in strict round-robin order: Thread-A prints 1, Thread-B prints 2, Thread-C prints 3, back to Thread-A, etc.

### Steps to implement
1. Shared state: an integer `turn` (0, 1, or 2) indicating whose turn it is.
2. Each thread loops: acquire lock, `while (turn != myTurn) wait();`, print, update `turn = (turn+1) % 3`, `notifyAll()`.

### Solution
```java
public class OrderedPrinter {
    private int turn = 0;
    private final Object lock = new Object();

    public void print(int myTurn, int value, int totalThreads) {
        for (int i = 0; i < 5; i++) { // print 5 rounds
            synchronized (lock) {
                while (turn != myTurn) {
                    try { lock.wait(); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
                }
                System.out.println(value);
                turn = (turn + 1) % totalThreads;
                lock.notifyAll();
            }
        }
    }

    public static void main(String[] args) {
        OrderedPrinter printer = new OrderedPrinter();
        new Thread(() -> printer.print(0, 1, 3)).start();
        new Thread(() -> printer.print(1, 2, 3)).start();
        new Thread(() -> printer.print(2, 3, 3)).start();
    }
}
```

### Why this is asked
It directly tests whether you truly understand `wait()`/`notify()` semantics and shared-state coordination — no library shortcuts available. Good warm-up before Problem 2.

---

## Problem 7: Dining Philosophers (Deadlock Avoidance)

### Requirements
N philosophers sit at a table with N forks (one between each pair). Each needs both adjacent forks to eat. Avoid deadlock (all picking up their left fork simultaneously, forever waiting for the right).

### Steps to implement (classic fix: resource ordering)
1. Represent each fork as a `Lock` (or synchronized object).
2. **The fix:** don't let every philosopher pick up "left fork, then right fork" blindly — instead, always acquire the **lower-numbered fork first**. This breaks the circular wait condition.

### Solution
```java
import java.util.concurrent.locks.ReentrantLock;

public class DiningPhilosophers {
    static class Philosopher extends Thread {
        private final ReentrantLock leftFork, rightFork;
        private final int id;

        Philosopher(int id, ReentrantLock leftFork, ReentrantLock rightFork) {
            this.id = id;
            this.leftFork = leftFork;
            this.rightFork = rightFork;
        }

        @Override
        public void run() {
            // Acquire in a globally consistent order to prevent circular wait
            ReentrantLock first = leftFork.hashCode() < rightFork.hashCode() ? leftFork : rightFork;
            ReentrantLock second = leftFork.hashCode() < rightFork.hashCode() ? rightFork : leftFork;

            first.lock();
            try {
                second.lock();
                try {
                    System.out.println("Philosopher " + id + " is eating");
                } finally {
                    second.unlock();
                }
            } finally {
                first.unlock();
            }
        }
    }

    public static void main(String[] args) {
        int n = 5;
        ReentrantLock[] forks = new ReentrantLock[n];
        for (int i = 0; i < n; i++) forks[i] = new ReentrantLock();

        for (int i = 0; i < n; i++) {
            new Philosopher(i, forks[i], forks[(i + 1) % n]).start();
        }
    }
}
```

### Why lock ordering fixes it
Deadlock requires a circular wait — each thread holding one resource while waiting for another, forming a cycle. If every thread acquires locks in the same global order (e.g., always lower ID first), a cycle can never form, since the "last" philosopher in any potential cycle would need to wait for a lower-numbered fork already held earlier in the chain instead of creating a loop.

### Follow-up
- Alternative fix: limit to N-1 philosophers allowed to sit at once (via `Semaphore(n-1)`) — discuss trade-offs vs. lock ordering.

---

## Problem 8: Thread-Safe Singleton (multiple correct approaches)

### Steps to implement — know all three, and their trade-offs

**A. Double-Checked Locking**
```java
public class Singleton {
    private static volatile Singleton instance; // volatile is REQUIRED here

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
**Why `volatile` is required:** `new Singleton()` isn't a single atomic step — it involves (1) allocate memory, (2) run constructor, (3) assign reference. Without `volatile`, the JVM/CPU may reorder steps 2 and 3, so another thread could see a non-null reference to a **half-constructed** object. `volatile` prevents this reordering (it establishes a happens-before relationship).

**B. Static Holder Idiom (preferred — simpler, no volatile needed)**
```java
public class Singleton {
    private Singleton() {}

    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}
```
**Why this works without explicit locking:** The JVM guarantees a class is initialized lazily (on first access) and thread-safely (class loading itself is synchronized by the classloader) — so `Holder.INSTANCE` is created exactly once, safely, the first time `getInstance()` is called.

**C. Enum Singleton (simplest, immune to reflection/serialization attacks)**
```java
public enum Singleton {
    INSTANCE;
    public void doSomething() { /* ... */ }
}
```

### Follow-up
- Why is enum singleton considered the most robust? (JVM guarantees enum instances are created once, and it's naturally safe against reflection-based instantiation and serialization creating duplicate instances — both of which can break approaches A and B if not handled carefully.)

---

## Problem 9: Semaphore-Based Connection Pool

### Requirements
Simulate a pool of N database connections; threads must wait if all are in use.

### Steps to implement
1. Use `Semaphore(N)` to represent available connections.
2. `acquire()` on the semaphore before using a "connection", `release()` after.
3. Maintain the actual pool of connection objects in a `BlockingQueue` or simple synchronized list, guarded by the semaphore's permit count.

### Solution
```java
import java.util.concurrent.*;
import java.util.*;

public class ConnectionPool {
    private final Semaphore semaphore;
    private final BlockingQueue<String> availableConnections;

    public ConnectionPool(int size) {
        this.semaphore = new Semaphore(size, true); // fair = true avoids starvation
        this.availableConnections = new LinkedBlockingQueue<>();
        for (int i = 0; i < size; i++) {
            availableConnections.add("Connection-" + i);
        }
    }

    public String acquire() throws InterruptedException {
        semaphore.acquire(); // blocks if none available
        return availableConnections.take();
    }

    public void release(String connection) {
        availableConnections.offer(connection);
        semaphore.release();
    }
}
```

### Why `Semaphore` here instead of just a `BlockingQueue` alone
A `BlockingQueue` alone would already provide the blocking behavior — but `Semaphore` is the idiomatic tool when you're modeling "N permits to access a resource" conceptually (and is what interviewers expect you to reach for). Mentioning that a bounded `BlockingQueue` of connections is *functionally* similar shows good judgment, but demonstrate you know `Semaphore` explicitly since that's likely the ask.

### Follow-up
- Why pass `fair = true` to `Semaphore`? (Default is unfair — can allow thread starvation under high contention, where a newly-arriving thread jumps ahead of one that's been waiting. `fair=true` uses FIFO ordering at a throughput cost.)

---

## Suggested Practice Order

1. Problem 6 (ordered printing) — warm-up, tests raw `wait/notify` fluency
2. Problem 2 (bounded buffer from scratch) — same primitives, harder
3. Problem 1 (producer-consumer with `BlockingQueue`) — shows you know the "easy way" too
4. Problem 9 (semaphore pool) — quick, tests a different primitive
5. Problem 8 (singleton) — quick, but be ready to explain **all three** approaches and the JMM reasoning
6. Problem 3 (LRU cache) — very likely to reappear in Round 3 (LLD), so know it cold
7. Problem 4 (rate limiter) — combines timing + locking, good "senior" signal
8. Problem 7 (dining philosophers) — classic, tests deadlock reasoning specifically
9. Problem 5 (custom thread pool) — the most likely "design + implement" combo question given your Round 2 description

For each, practice **explaining the "why" out loud while you code** — that's specifically what "advanced concurrency semantics" interviews are scoring, not just whether the code compiles.
