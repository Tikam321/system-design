# Asyncio vs Threading in Python

## 1. What is Threading?

Threading means:
- Multiple threads run inside the same process
- Each thread has its own execution flow
- Useful for blocking I/O operations

Python uses:

```python
import threading
```

---

## Threading Example

```python
import threading
import time

def task(name):
    print(f"Task {name} started")
    time.sleep(3)
    print(f"Task {name} completed")

t1 = threading.Thread(target=task, args=("A",))
t2 = threading.Thread(target=task, args=("B",))

t1.start()
t2.start()

t1.join()
t2.join()

print("All tasks done")
```

---

## How Threading Works

- OS schedules threads
- One thread may block
- Other threads can continue execution

Example:

```python
time.sleep(5)
```

This blocks the current thread.

But:
- other threads continue running

---

## Threading is Good For

- API calls
- Database operations
- File reading/writing
- Network requests
- Blocking I/O tasks

---

## Limitation of Threading in Python

Python has:

# GIL (Global Interpreter Lock)

Only one thread executes Python bytecode at a time.

So:
- Threading is NOT ideal for CPU-heavy tasks

---

## CPU-bound vs I/O-bound

### CPU-bound Tasks

Heavy computation:
- image processing
- machine learning
- compression

Use:
- multiprocessing

---

### I/O-bound Tasks

Waiting tasks:
- API requests
- DB calls
- sockets
- files

Use:
- threading OR asyncio

---

# 2. What is Asyncio?

Asyncio is:
- Single-threaded concurrency
- Event-loop based
- Non-blocking asynchronous programming

Python uses:
- `async`
- `await`
- `asyncio`

---

## Asyncio Example

```python
import asyncio

async def task(name):
    print(f"Task {name} started")
    await asyncio.sleep(3)
    print(f"Task {name} completed")

async def main():
    t1 = asyncio.create_task(task("A"))
    t2 = asyncio.create_task(task("B"))

    await t1
    await t2

asyncio.run(main())
```

---

# 3. What Does `await` Do?

`await` means:

> Pause the current coroutine and allow the event loop to execute another task.

Example:

```python
await asyncio.sleep(3)
```

The coroutine pauses itself.

The thread is NOT blocked.

The event loop switches to another ready task.

---

## Important

### Good (Non-blocking)

```python
await asyncio.sleep(2)
```

---

### Bad (Blocking)

```python
time.sleep(2)
```

This blocks the entire thread and freezes the event loop.

---

# 4. What is the Event Loop?

The event loop is the core engine of asyncio.

It:
1. runs async tasks
2. pauses tasks during waiting
3. resumes tasks when ready
4. schedules execution efficiently

---

## Simple Definition

> Event loop is a scheduler that manages and executes asynchronous tasks efficiently inside a single thread.

---

## Event Loop Flow

```text
Event Loop
    |
    |----> Task1 running
    |         |
    |         ---> await → paused
    |
    |----> Task2 running
    |         |
    |         ---> await → paused
    |
    |----> Resume ready task
```

---

# 5. What is a Coroutine?

A coroutine is:

> A resumable async function execution state.

Example:

```python
async def fetch():
    await asyncio.sleep(1)
```

Calling:

```python
coro = fetch()
```

creates a coroutine object.

---

## Coroutine Stores

- current execution line
- local variables
- execution state
- paused position

---

## Coroutine Lifecycle

### Step 1

Coroutine starts.

---

### Step 2

Hits:

```python
await asyncio.sleep(5)
```

Coroutine pauses.

---

### Step 3

Event loop stores its state.

---

### Step 4

Later:
- event loop resumes coroutine
- execution continues from same point

---

# 6. Coroutine vs Thread

## Thread

- OS-level execution unit
- heavier
- managed by operating system

---

## Coroutine

- lightweight
- managed by event loop
- resumable function state

---

# 7. Coroutine vs Task

## Coroutine

Just an async function object.

```python
coro = fetch()
```

---

## Task

A coroutine scheduled by event loop.

```python
task = asyncio.create_task(fetch())
```

---

# 8. Threading vs Asyncio

| Feature | Threading | Asyncio |
|---|---|---|
| Model | Multiple OS threads | Single thread |
| Switching | OS-controlled | Event loop |
| Memory Usage | Higher | Lower |
| Best For | Blocking I/O | Massive concurrency |
| Parallelism | Limited by GIL | Cooperative multitasking |
| Scalability | Moderate | Very high |

---

# 9. Cooperative Multitasking

Asyncio uses:

# Cooperative Multitasking

Tasks voluntarily give control back using:

```python
await
```

Unlike threading:
- OS does NOT force switch execution

---

# 10. Real-world Use Cases

## Threading

Use for:
- background jobs
- blocking libraries
- small concurrency

---

## Asyncio

Use for:
- chat applications
- WebSockets
- realtime systems
- API gateways
- high-scale backend servers

Frameworks:
- FastAPI
- aiohttp
- Node.js
- NGINX

---

# 11. Interview One-Liners

## Threading

> Threading achieves concurrency using multiple OS threads.

---

## Asyncio

> Asyncio achieves concurrency using a single-threaded event loop and cooperative multitasking.

---

## Event Loop

> Event loop continuously schedules, pauses, and resumes asynchronous tasks.

---

## Coroutine

> A coroutine is a resumable async function execution state managed by the event loop.
