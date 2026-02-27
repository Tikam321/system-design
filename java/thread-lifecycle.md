### The Thread States
## State 1: NEW (Created)
- A thread in the NEW state has been created as an object in memory, but hasn't started executing yet. The operating system hasn't allocated resources for its execution.

In this state:

The thread object exists in your program's heap
No OS-level thread has been created
The thread cannot run any code yet
Calling most thread methods will fail or have no effect
```java
public class NewStateDemo {
    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            System.out.println("Running");
        });

        // Thread is in NEW state
        System.out.println("State: " + thread.getState());  // NEW
        System.out.println("Is alive: " + thread.isAlive()); // false

        // The thread exists but hasn't started
        // No OS thread has been created yet
    }
}
```
## state 2: RUNNABLE (Ready to Run)

- Once start() is called, the thread moves to the RUNNABLE state. The OS has created the thread and it's eligible to run, but it might not be running right now. The scheduler decides which runnable threads get CPU time.

In this state:

The OS thread exists and is scheduled
The thread may or may not be currently executing
It's competing with other threads for CPU time
The scheduler can preempt it at any time
```java
public class RunnableStateDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            // This thread is RUNNABLE while executing
            // (Java combines RUNNABLE and RUNNING)
            for (int i = 0; i < 1_000_000; i++) {
                // Busy work
                Math.sqrt(i);
            }
        });

        thread.start();  // Transitions from NEW to RUNNABLE

        // Give it a moment to start
        Thread.sleep(1);
        System.out.println("State: " + thread.getState());  // RUNNABLE

        thread.join();
    }
}
```

## State 3: RUNNING (Executing)
A thread is RUNNING when it's actively executing instructions on a CPU core. This is a sub-state of RUNNABLE in most models. The thread has been selected by the scheduler and is consuming CPU cycles.

A thread leaves the RUNNING state when:

Its time slice expires (preemption)
It voluntarily yields
It blocks on I/O or synchronization
It terminates

<img width="1046" height="509" alt="Screenshot 2026-02-27 at 10 55 00 PM" src="https://github.com/user-attachments/assets/80edaaa0-648c-4332-806e-23f2e7ee394e" />

## State 4: BLOCKED (Waiting for Lock)
- A thread enters the BLOCKED state when it tries to acquire a lock (monitor) that another thread holds. It cannot proceed until the lock becomes available.
- This is different from WAITING. BLOCKED specifically means "waiting to enter a synchronized block or method." The thread isn't waiting for a signal; it's waiting for exclusive access to a resource.

```java
public class BlockedStateDemo {
    private static final Object lock = new Object();

    public static void main(String[] args) throws InterruptedException {
        // Thread 1 holds the lock
        Thread holder = new Thread(() -> {
            synchronized (lock) {
                System.out.println("Holder: acquired lock");
                try {
                    Thread.sleep(5000);  // Hold lock for 5 seconds
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
                System.out.println("Holder: releasing lock");
            }
        }, "LockHolder");

        // Thread 2 will be blocked waiting for the lock
        Thread waiter = new Thread(() -> {
            System.out.println("Waiter: trying to acquire lock");
            synchronized (lock) {
                System.out.println("Waiter: acquired lock");
            }
        }, "LockWaiter");

        holder.start();
        Thread.sleep(100);  // Let holder acquire lock first

        waiter.start();
        Thread.sleep(100);  // Let waiter attempt to acquire

        // Waiter is now BLOCKED
        System.out.println("Waiter state: " + waiter.getState());  // BLOCKED

        holder.join();
        waiter.join();
    }
}
```
## State 5: WAITING (Indefinite Wait)
- A thread enters the WAITING state when it explicitly waits for another thread to perform an action. Unlike BLOCKED, the thread isn't competing for a lock; it's parked until signaled.
```java
public class WaitingStateDemo {
    private static final Object monitor = new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread waiter = new Thread(() -> {
            synchronized (monitor) {
                System.out.println("Waiter: entering wait");
                try {
                    monitor.wait();  // Wait indefinitely
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
                System.out.println("Waiter: woken up!");
            }
        }, "WaiterThread");

        waiter.start();
        Thread.sleep(100);  // Let waiter enter wait state

        // Thread is now WAITING
        System.out.println("Waiter state: " + waiter.getState());  // WAITING

        // Wake up the waiter
        synchronized (monitor) {
            monitor.notify();
        }

        waiter.join();
    }
}
```
## state 6: timed waiting
- TIMED_WAITING is similar to WAITING, but with a timeout. The thread will wake up either when signaled or when the timeout expires, whichever comes first.

```java
public class TimedWaitingDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread sleeper = new Thread(() -> {
            System.out.println("Sleeper: going to sleep for 1 seconds");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                System.out.println("Sleeper: interrupted!");
                return;
            }
            System.out.println("Sleeper: woke up naturally");
        }, "SleeperThread");

        sleeper.start();
        Thread.sleep(100);  // Let sleeper enter sleep

        // Thread is now TIMED_WAITING
        System.out.println("Sleeper state: " + sleeper.getState());  // TIMED_WAITING

        // Optionally interrupt to wake early
        // sleeper.interrupt();

        sleeper.join();
    }
}
```
##  State 7: TERMINATED (Dead)
- A thread enters the TERMINATED state when its execution completes, either normally or by throwing an uncaught exception. The thread can never run again. Its resources are released.
```java
public class TerminatedStateDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread worker = new Thread(() -> {
            System.out.println("Worker: starting");
            // Do some work
            for (int i = 0; i < 5; i++) {
                System.out.println("Worker: " + i);
            }
            System.out.println("Worker: done");
        }, "WorkerThread");

        worker.start();
        worker.join();  // Wait for completion

        // Thread is now TERMINATED
        System.out.println("Worker state: " + worker.getState());  // TERMINATED
        System.out.println("Is alive: " + worker.isAlive());        // false

        // Cannot restart a terminated thread
        try {
            worker.start();
        } catch (IllegalThreadStateException e) {
            System.out.println("Cannot restart: " + e.getMessage());
        }
    }
}
``` 

