
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
