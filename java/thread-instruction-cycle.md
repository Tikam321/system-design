# Java Threads and CPU Instruction Cycle - Reference Notes

## 1) Big Picture

- Java source code is compiled to bytecode.
- The JVM interprets bytecode or JIT-compiles it to native machine code.
- The CPU executes native machine instructions.
- Java threads are mapped to native OS threads, and the OS scheduler decides when each thread runs.

Flow:

```text
Java Source -> Bytecode -> JVM (Interpreter/JIT) -> Native Instructions -> CPU
```

## 2) Where Instructions and Data Live

- Method instructions are stored in shared JVM code memory (method area/code cache).
- Each thread has its own private stack.
- Stack frames store:
- local variables
- operand stack
- return info
- Object data usually lives in heap memory (shared).

Important:
- Instructions are NOT stored in thread stack.
- Thread stack stores execution state/data, not code.

## 3) Thread + Memory Diagram

```text
Shared JVM Memory
+------------------------------------------------------+
| Code Area (shared)                                  |
| calc(): load c,d -> mul -> store x                  |
|         load a,b -> add -> store y                  |
|         load y,x -> sub -> return                   |
+------------------------------------------------------+
| Heap (shared objects)                               |
+------------------------------------------------------+

Thread-1 Stack (private)            Thread-2 Stack (private)
+-------------------------------+   +-------------------------------+
| Frame: calc(2,3,4,5)          |   | Frame: calc(10,1,2,3)        |
| a=2 b=3 c=4 d=5 x=20 y=5      |   | a=10 b=1 c=2 d=3 x=6 y=11    |
| operand stack (temporary)     |   | operand stack (temporary)     |
+-------------------------------+   +-------------------------------+
```

## 4) CPU Instruction Cycle (Standard Model)

The simplified cycle:

1. Fetch instruction
2. Decode instruction
3. Read operands
4. Execute
5. Write back result
6. Update PC (program counter)
7. Repeat

Diagram:

```text
Fetch -> Decode -> Read Operands -> Execute -> Write Back -> Update PC -> repeat
```

Detailed block view:

```text
        +---------+
        |   PC    |
        +----+----+
             |
             v
+-----------------------+
| 1) FETCH instruction  |
+-----------+-----------+
            |
            v
+-----------------------+
| 2) DECODE instruction |
+-----------+-----------+
            |
            v
+-----------------------+
| 3) READ operands      |
+-----------+-----------+
            |
            v
+-----------------------+
| 4) EXECUTE            |
+-----------+-----------+
            |
            v
+-----------------------+
| 5) WRITE BACK result  |
+-----------+-----------+
            |
            v
+-----------------------+
| 6) UPDATE PC          |
+-----------+-----------+
            |
            +-------> loop
```

## 5) Example Method

```java
int calc(int a, int b, int c, int d) {
    int x = c * d;
    int y = a + b;
    return y - x;
}
```

Execution idea:

- Both threads execute the same shared instructions of `calc`.
- Each thread uses its own stack frame values.
- Example:
- Thread-1 with `calc(2,3,4,5)` returns `-15`
- Thread-2 with `calc(10,1,2,3)` returns `5`

## 6) How OS Gives CPU Time to Threads

- Runnable threads wait in run queues.
- OS scheduler picks a thread for a CPU core.
- CPU runs it for a time slice (quantum) or until block/preemption.
- If needed, OS does context switch:
- saves current thread CPU state (registers, PC, stack pointer)
- loads next thread state

If thread blocks (`sleep`, I/O, waiting for lock), another runnable thread is scheduled.

## 7) Parallel vs Concurrent Execution

- Single core:
- only one thread executes at a moment
- concurrency is achieved by fast context switching
- Multi-core:
- multiple threads can truly run in parallel

## 8) CPU Cycles per Second

- Approx cycles/sec = CPU clock frequency.
- `1 GHz = 1 billion cycles per second`.
- Typical modern CPU cores often run around `2-5+ GHz` (dynamic with turbo/load/thermal limits).

Note:
- Cycles per second is not equal to instructions per second.
- Modern CPUs pipeline and optimize execution.

## 9) Quick Q&A Recap

Q: Are instructions stored in thread stack?  
A: No. Instructions are shared in JVM code memory.

Q: What is in thread stack then?  
A: Per-thread method frames, locals, operand stack, return metadata.

Q: Why can two threads run same method safely?  
A: They share code, but each has independent stack data.

Q: Who decides which thread runs now?  
A: OS scheduler.

---

These notes are designed as interview + concept revision reference.
