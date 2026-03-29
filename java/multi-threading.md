Imagine 1 CPU core and 3 threads: T1, T2, T3.
Time slice = 5 ms.

Timeline:

0-5ms   -> T1 runs
5-10ms  -> T2 runs
10-15ms -> T3 runs
15-20ms -> T1 runs again
20-25ms -> T2 runs again
...
Each thread gets a small turn, again and again.
This is time slicing.

Why it feels “parallel”:

Switching is very fast, so all threads make progress.
If one thread blocks early:

Example T2 does file I/O at 7ms and blocks.
OS immediately gives CPU to next runnable thread (T3), not waiting full 5ms.
So on one core:

only one thread runs at an instant,
but time slicing shares CPU fairly across many threads.
