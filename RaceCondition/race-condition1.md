# Race Conditions

## What is a Race Condition?
A **race condition** occurs when multiple threads access the same shared data concurrently, and at least one thread modifies it. The program's result depends on the unpredictable order (interleaving) of thread execution.

### Formula
```
Shared State + Concurrent Access + At least One Write + No Synchronization = Race Condition
```

---

# Why Race Conditions Happen

A race condition requires all three conditions:

- **Shared State** → Multiple threads access the same variable/object.
- **Mutability** → At least one thread modifies the shared data.
- **Concurrent Access** → Threads execute without synchronization.

Remove any one of these and race conditions disappear.

---

# Example: Lost Update

Expected:

```
Thread 1: counter++
Thread 2: counter++

Expected Counter = 2
```

Actual:

```
Thread 1: Read 0
Thread 2: Read 0

Thread 1: Increment → 1
Thread 2: Increment → 1

Thread 1: Write 1
Thread 2: Write 1

Final Counter = 1 ❌
```

Both threads read the same value before either writes, so one update is lost.

---

# Why `counter++` is Unsafe

`counter++` looks like one operation but internally it performs three steps:

```
Read
   ↓
Increment
   ↓
Write
```

```
counter++
```

Equivalent to:

```java
int temp = counter;   // Read
temp = temp + 1;      // Modify
counter = temp;       // Write
```

A context switch between these steps causes data corruption.

---

# Read-Modify-Write Race

```
Read → Modify → Write
```

Another thread can execute between any of these operations.

Example:

```
T1 Read 0
T2 Read 0
T1 Write 1
T2 Write 1

Final = 1
Expected = 2
```

---

# Check-Then-Act Race

Occurs when a thread checks a condition and acts on it later.

```java
if(instance == null){
    instance = new Singleton();
}
```

Possible execution:

```
T1 checks → null
                 Context Switch
T2 checks → null
T2 creates object
                 Context Switch
T1 creates another object
```

Result:

```
Two Singleton objects ❌
```

---

# Real World Examples

### Inventory Overselling

```
Stock = 1

Customer A checks stock
Customer B checks stock

Both purchase

Stock = -1
```

---

### Bank Account

```
Balance = 500

Withdraw 100
Deposit 200

Expected = 600

Actual:
300 ❌
700 ❌
```

---

### Double Payment

Two payment requests execute simultaneously.

```
Both verify balance
Both deduct money

Incorrect final balance
```

---

### Lost Database Update

```
User A edits record
User B edits same record

User B saves last

User A changes lost
```

---

# Critical Section

A **critical section** is the portion of code that accesses shared mutable data.

Example:

```java
counter++;
balance += amount;
map.put(key, value);
queue.add(item);
```

Only one thread should execute a critical section at a time.

---

# Properties of a Good Critical Section

- Mutual Exclusion
- Progress
- Bounded Waiting (No Starvation)
- Independent of CPU Speed

---

# Mutual Exclusion

Only one thread can execute the critical section at a time.

```
Thread 1
    │
 Acquire Lock
    │
Critical Section
    │
Release Lock
    │
Thread 2 Enters
```

---

# Solutions

## 1. synchronized

```java
class Counter {

    private int count;

    public synchronized void increment() {
        count++;
    }
}
```

✔ Simple

✔ Built into Java

✔ Prevents race conditions

---

## 2. ReentrantLock

```java
Lock lock = new ReentrantLock();

lock.lock();

try{
    counter++;
}
finally{
    lock.unlock();
}
```

Advantages

- tryLock()
- Fair Locking
- Interruptible Lock
- Multiple Conditions

---

## 3. Atomic Variables

Best for simple counters.

```java
AtomicInteger counter = new AtomicInteger();

counter.incrementAndGet();
```

Advantages

- Lock-free
- Very fast
- Thread-safe

---

## 4. Immutability

Immutable objects cannot have race conditions.

```java
String name = "Java";
LocalDate today = LocalDate.now();
```

No thread can modify immutable objects after creation.

---

# Preventing Race Conditions

- Synchronize shared data.
- Use `ReentrantLock` when advanced locking is needed.
- Use `AtomicInteger`, `AtomicLong`, etc., for counters.
- Prefer immutable objects.
- Minimize shared mutable state.
- Keep critical sections small.

---

# Interview Questions

### What is a race condition?

When multiple threads access shared mutable data concurrently without proper synchronization, producing unpredictable results.

---

### Why is `counter++` not thread-safe?

Because it performs:

```
Read
Modify
Write
```

which are three separate operations.

---

### What is a critical section?

The part of code that accesses shared mutable resources and must execute by only one thread at a time.

---

### Difference between Race Condition and Critical Section

| Race Condition | Critical Section |
|---------------|------------------|
| Problem | Protected code |
| Causes incorrect results | Prevents incorrect results |
| Happens due to concurrent unsynchronized access | Uses synchronization |

---

### How can race conditions be prevented?

- synchronized
- ReentrantLock
- Atomic classes
- Immutable objects
- Reduce shared mutable state

---

# Key Takeaways

- Race conditions occur because thread execution order is unpredictable.
- `counter++` is **not atomic**.
- Shared mutable data must be synchronized.
- Critical sections ensure only one thread modifies shared data at a time.
- Prefer **Atomic classes** for simple variables and **Locks/synchronized** for complex operations.
