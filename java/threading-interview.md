
# Thread Interruption in Java

## Does a Thread Get Destroyed After Interruption?

No, when a thread moves to the `RUNNABLE` state after interruption, it is not destroyed immediately.

`RUNNABLE` means:
- the thread is ready to execute
- or currently executing

What happens next depends on the thread's code logic.

## What Happens During Interruption?

Suppose a thread is inside:
- `sleep()`
- `wait()`
- `join()`

and another thread calls:

```java
thread.interrupt();
```

Then internally:

1. JVM sets the interrupt flag
2. The blocking method detects interruption
3. `InterruptedException` is thrown
4. The thread wakes up
5. The thread moves back to the `RUNNABLE` state
6. The thread continues execution

## Example

```java
Thread t = new Thread(() -> {

    try {

        System.out.println("Sleeping");

        Thread.sleep(10000);

    } catch (InterruptedException e) {

        System.out.println("Interrupted");
    }

    System.out.println("Thread still running");
});

t.start();

Thread.sleep(2000);

t.interrupt();
```

## Output

```text
Sleeping
Interrupted
Thread still running
```

### Observation

The thread:
- was interrupted
- woke up
- continued execution
- did not terminate immediately

## When Does a Thread Actually Terminate?

A thread terminates only when:
- the `run()` method completes
OR
- an uncaught exception occurs

Then the thread state becomes:

```text
TERMINATED
```

## Thread Lifecycle During Interruption

```text
RUNNING
   ↓
TIMED_WAITING (sleep)
   ↓ interrupt()
RUNNABLE
   ↓
catch block executes
   ↓
remaining code runs
   ↓
run() method completes
   ↓
TERMINATED
```

## Important Point

`interrupt()` does not forcibly kill the thread.

It only sends an interruption signal.

The thread itself decides:
- whether to stop
- continue working
- cleanup resources
- terminate gracefully

## Common Graceful Shutdown Pattern

```java
while (!Thread.currentThread().isInterrupted()) {

    // do work
}
```

When interruption occurs:
- interrupt flag becomes true
- loop exits
- thread finishes gracefully

## Example Graceful Shutdown

```java
Thread t = new Thread(() -> {

    while (!Thread.currentThread().isInterrupted()) {

        System.out.println("Working");
    }

    System.out.println("Cleaning up...");
});

t.start();

Thread.sleep(2000);

t.interrupt();
```

## Output

```text
Working
Working
Cleaning up...
```

Then the thread terminates normally.

## Senior-Level Understanding

Thread interruption is called:

### Cooperative Thread Cancellation

Meaning:
- JVM does not forcefully stop the thread
- the thread cooperates by checking interruption status

This prevents:
- inconsistent state
- resource leaks
- corrupted data

## Interview-Ready Answer

When a waiting or sleeping thread is interrupted, it moves back to the RUNNABLE state and resumes execution, usually by handling `InterruptedException`. The thread is not destroyed automatically. A thread only terminates when its `run()` method finishes or an uncaught exception occurs.

# 2. # What is ReentrantLock?

`ReentrantLock` is a thread synchronization mechanism from:

```java
java.util.concurrent.locks
```

It provides more control and flexibility than `synchronized`.

---

# Why Called Reentrant?

Same thread can acquire the same lock multiple times without deadlock.

---

# Example

```java
import java.util.concurrent.locks.ReentrantLock;

class Counter {

    ReentrantLock lock = new ReentrantLock();

    void increment() {

        lock.lock();

        try {
            System.out.println("Locked");
        } finally {
            lock.unlock();
        }
    }
}
```

---

# Features of ReentrantLock

---

# 1. lock()

Acquires lock.

```java
lock.lock();
```

Thread waits until lock becomes available.

---

# 2. unlock()

Releases lock.

```java
lock.unlock();
```

Usually placed inside:

```java
finally
```

---

# 3. tryLock()

Attempts to acquire lock without waiting forever.

```java
if(lock.tryLock()) {

    try {

    } finally {
        lock.unlock();
    }

} else {
    System.out.println("Lock not available");
}
```

Useful to avoid deadlocks.

---

# 4. tryLock(timeout)

Waits for specific time.

```java
lock.tryLock(5, TimeUnit.SECONDS);
```

---

# 5. lockInterruptibly()

Allows thread interruption while waiting for lock.

```java
lock.lockInterruptibly();
```

---

# 6. Fair Locking

```java
new ReentrantLock(true);
```

Threads acquire lock in FIFO order.

Prevents thread starvation.

---

# Difference Between synchronized and ReentrantLock

| synchronized | ReentrantLock |
|---|---|
| Less flexible | More flexible |
| No tryLock() | Supports tryLock() |
| No fairness policy | Supports fairness |
| Automatically releases lock | Must manually unlock |
| Simpler syntax | Advanced control |

---

# Interview Ready Answer

> ReentrantLock is an advanced locking mechanism in Java that provides more flexibility than synchronized blocks. It supports features like `tryLock()`, timed lock waiting, interruptible locking, and fair locking. It is called reentrant because the same thread can acquire the same lock multiple times safely.
