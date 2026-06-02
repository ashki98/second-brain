# Asyncio

YT: 

[https://www.youtube.com/watch?v=oAkLSJNr5zY&t=3498s](https://www.youtube.com/watch?v=oAkLSJNr5zY&t=3498s)

Here’s a **comprehensive, note-style summary of the entire “Python Tutorial: AsyncIO - Complete Guide to Asynchronous Programming with Animations” video**—optimized for easy memory refresh and with your specific conversation points emphasized.

---

## **Core Concepts & Terminology**

- **AsyncIO** is Python’s core library for writing asynchronous code using `async`/`await`. It enables *concurrent* I/O-bound tasks within a single thread & process.
- **Concurrency vs Parallelism:**
    - *Concurrency* (“McDonald’s orders taken while food is made”) lets multiple tasks proceed, *not* necessarily faster but with more efficient waiting.
    - *Synchronous* code is like Subway—one sandwich, one customer, at a time.
    - Not all async code is faster; *faster* occurs if you’re waiting (I/O bound), not crunching numbers (CPU bound).
- **Event Loop:** The “engine” that schedules, suspends, and resumes coroutines & tasks. Must be running for async code to work.
- **Cooperative multitasking:** Async code *yields* control when it hits an `await`; other tasks can progress while waiting.

---

## **AsyncIO Key Abstractions**

- **Coroutine functions**: Defined with `async def`; “pausable” functions.
- **Coroutine objects**: Created when you call an async function, need to be *awaited* or scheduled.
- **Tasks**: Wrap/capture coroutine objects for scheduling, tracking completion and results.
- **Futures**: Low-level containers for “eventual” results; rarely directly used, mostly handled by tasks.

---

## **Writing & Running Asynchronous Code**

- Must run everything inside an event loop (`asyncio.run(...)`).
- Only `async` functions can use `await`.
- *Synchronous* code (e.g., `time.sleep()`) does **not** work with `await`, and blocks the entire event loop!
- To offload **blocking sync functions**, use `asyncio.to_thread` or `run_in_executor`. (*Your specific doubt: to_thread is ONLY for sync functions you want to run non-blocking!*)

---

## **Scheduling & Running Concurrency**

- **Mistake:** Awaiting coroutine objects directly schedules and runs them to completion sequentially—no concurrency.
- **Correct:**
    - Create **tasks** first (`asyncio.create_task(...)`), then await them—now you get true concurrency!
    - Use batch scheduling tools:
        - `asyncio.gather(coroutines/tasks)` to run many concurrently; result order matches input order.
        - **TaskGroup** (Python 3.11+) to safely schedule/wait for groups, better error handling.
- Your questions about **threads & processes**:
    - Use **ThreadPoolExecutor** for I/O-bound work, *ProcessPoolExecutor* for CPU-bound work.
    - AsyncIO handles both, but thread pool doesn’t bypass the GIL; processes do.
    - You can “parallelize” sync functions in threads/processes via AsyncIO.

I left that out because, although it demonstrates scheduling, it represents a “broken” execution state that would likely fail in a real-world script. To make this documentation truly self-sufficient, we should include it as a **Critical Warning** section.

Below is the updated documentation, including that scenario and its consequences.

---

# Documentation: Python Concurrency & AsyncIO

## 1. Core Logic: The Event Loop

AsyncIO uses a single-threaded **Event Loop**. It works by “pausing” one task during its idle time (like waiting for a website to respond) and switching to another task.

---

## 2. Scheduling vs. Awaiting (The Timing Patterns)

To understand these patterns, assume two tasks:

- **Task A (`fetch_data`)**: Takes 2 seconds.
- **Task B (`get_time`)**: Takes 2 seconds.

### A. The Sequential Mistake

Python

#

`await task_a() 
await task_b()`

**Behavior:** The code waits for A to finish completely before starting B.

**Total Time:** **4 seconds**.

### B. The "Abandoned Task" Trap (Your Example)

Python

#

`async def main():
    asyncio.create_task(task_a())
    asyncio.create_task(task_b())
    # main() ends here`

**Behavior:** `create_task` tells the loop “do this when you can,” but `main()` then finishes immediately. When `main()` ends, the loop shuts down.

**Result:** **Failure.** The tasks are cancelled before they can finish because the program never waited for them.

### C. True Concurrency (The Goal)

Python

#

`task1 = asyncio.create_task(task_a())
task2 = asyncio.create_task(task_b())
await task1 # main() pauses here, but BOTH tasks are now running in the loop
await task2`

**Behavior:** Both tasks are scheduled and allowed to run.

**Total Time:** **2 seconds** ($max(A, B)$).

---

## 3. Toolset for Grouping & Safety

| **Tool** | **Usage** | **Best For...** |
| --- | --- | --- |
| **`asyncio.gather()`** | `results = await asyncio.gather(a, b)` | Getting a **list of results** back in a specific order. |
| **`asyncio.TaskGroup`** | `async with TaskGroup() as tg:` | **Structured Concurrency (3.11+).** If Task A fails, Task B is automatically cleaned up/cancelled. |

---

## 4. Hardware Constraints: The GIL

The **Global Interpreter Lock (GIL)** prevents Python from running multiple threads of Python code at once.

- **I/O Tasks (Network/Disk):** Use **AsyncIO**. The GIL is released while waiting for a response, enabling concurrency.
- **CPU Tasks (Math/Data):** Use **`ProcessPoolExecutor`**. This bypasses the GIL by creating separate worker processes on different CPU cores.

---

## 5. Self-Sufficient Implementation Template

This script demonstrates how to handle I/O, CPU-bound work, and proper task cleanup simultaneously.

Python

#

`import asyncio
from concurrent.futures import ProcessPoolExecutor

def heavy_math(n):
    return sum(range(n)) # CPU-intensive

async def main():
    # 1. Properly Grouped I/O Tasks
    async with asyncio.TaskGroup() as tg:
        t1 = tg.create_task(asyncio.sleep(2))
        t2 = tg.create_task(asyncio.sleep(2))
    
    # 2. Bypassing the GIL for Math
    loop = asyncio.get_running_loop()
    with ProcessPoolExecutor() as pool:
        # Prevents heavy_math from freezing the sleep tasks above
        result = await loop.run_in_executor(pool, heavy_math, 10**8)
    
    print(f"Calculated {result} while I/O tasks ran concurrently.")

if __name__ == "__main__":
    asyncio.run(main())`

### Summary Checklist for Documentation

1. **Never leave a task un-awaited:** Use `await`, `gather`, or `TaskGroup` so the program doesn’t exit prematurely.
2. **Schedule First:** Use `create_task` or `tg.create_task` before you start awaiting, so tasks can run concurrently.
3. **Process for Power:** If CPU usage hits 100%, `async` won’t help; move that specific work into a `ProcessPoolExecutor`.

---

# Documentation: Python Concurrency & AsyncIO

## 1. Core Logic: The Event Loop

AsyncIO uses a single-threaded **Event Loop**. It works by "pausing" one task during its idle time (like waiting for a website to respond) and switching to another task.

---

## 2. Scheduling vs. Awaiting (The Timing Patterns)

To understand these patterns, assume two tasks:

- **Task A (`fetch_data`)**: Takes 2 seconds.
- **Task B (`get_time`)**: Takes 2 seconds.

### A. The Sequential Mistake

Python

#

`await task_a() 
await task_b()`

**Behavior:** The code waits for A to finish completely before even starting B.

**Total Time:** **4 seconds**.

### B. The "Abandoned Task" Trap (Your Example)

Python

#

`async def main():
    asyncio.create_task(task_a())
    asyncio.create_task(task_b())
    # main() ends here`

**Behavior:** `create_task` tells the loop "do this when you can," but then `main()` immediately finishes. When `main()` ends, the loop shuts down.

**Result:** **Failure.** The tasks are cancelled before they ever finish because the program didn't wait for them.

### C. True Concurrency (The Goal)

Python

#

`task1 = asyncio.create_task(task_a())
task2 = asyncio.create_task(task_b())
await task1 # main() pauses here, but BOTH tasks are now running in the loop
await task2`

**Behavior:** Both tasks are scheduled and allowed to run.

**Total Time:** **2 seconds** ($max(A, B)$).

---

## 3. Toolset for Grouping & Safety

| **Tool** | **Usage** | **Best For...** |
| --- | --- | --- |
| **`asyncio.gather()`** | `results = await asyncio.gather(a, b)` | Getting a **list of results** back in a specific order. |
| **`asyncio.TaskGroup`** | `async with TaskGroup() as tg:` | **Structured Concurrency (3.11+).** If Task A fails, Task B is automatically cleaned up/cancelled. |

---

## 4. Hardware Constraints: The GIL

The **Global Interpreter Lock (GIL)** stops Python from running multiple threads of code at once.

- **I/O Tasks (Network/Disk):** Use **AsyncIO**. The GIL is released while waiting for a response, allowing "concurrency."
- **CPU Tasks (Math/Data):** Use **`ProcessPoolExecutor`**. This bypasses the GIL by creating separate "worker" processes on different CPU cores.

---

## 5. Self-Sufficient Implementation Template

This script demonstrates how to handle I/O, CPU-bound work, and proper task cleanup simultaneously.

Python

#

`import asyncio
from concurrent.futures import ProcessPoolExecutor

def heavy_math(n):
    return sum(range(n)) # CPU-intensive

async def main():
    # 1. Properly Grouped I/O Tasks
    async with asyncio.TaskGroup() as tg:
        t1 = tg.create_task(asyncio.sleep(2))
        t2 = tg.create_task(asyncio.sleep(2))
    
    # 2. Bypassing the GIL for Math
    loop = asyncio.get_running_loop()
    with ProcessPoolExecutor() as pool:
        # Prevents heavy_math from freezing the sleep tasks above
        result = await loop.run_in_executor(pool, heavy_math, 10**8)
    
    print(f"Calculated {result} while I/O tasks ran concurrently.")

if __name__ == "__main__":
    asyncio.run(main())`

### Summary Checklist for Documentation

1. **Never leave a task un-awaited:** Use `await`, `gather`, or `TaskGroup` so the program doesn't exit prematurely.
2. **Schedule First:** Always use `create_task` or `tg.create_task` before you start awaiting to ensure they run at the same time.
3. **Process for Power:** If the CPU usage hits 100%, `async` won't help; move that specific function into a `ProcessPoolExecutor`.

---

## **Practical Convert & Profile**

- Use **profilers** to identify bottlenecks—figure out which functions are I/O bound vs CPU bound.
- Convert codebase:
    - Network/download operations → async libraries (`httpx`, `aiohttp`), or threads if library isn’t async.
    - Data processing/computation → processes for true parallelism.
- Demo: Real-world example shows how switching to async downloads (`httpx`), and process-based parallel image processing, *dramatically reduces runtime* from tens of seconds to just a few.

---

## **Resource Limiting**

- **Semaphores**:
    - Limit concurrency (max downloads, API calls, etc.)—critical when scaling or rate-limiting.
    - *In context of the event loop*: Semaphore yields if max count reached; event loop suspends coroutines waiting for it—this prevents overloading your system/server.
- **CPU worker counts**: Get system CPU count to size your process pool for max parallelism in heavy computations.

---

## **Common Pitfalls (Emphasized from Your Questions)**

- **Forgetting `await`**: Tasks not awaited won’t run! Always `await` tasks, coroutines, and blocking futures.
- **Blocking event loop with sync code**: *Your confusion clarified!*
    - Never run sync (time.sleep, requests, etc.) in an async function unless you offload it via threads/processes.
    - **`asyncio.to_thread`** is ONLY for making sync code non-blocking. Not for async code.
- **Mixing up promises and async/await in Node.js vs Python**:
    - Node’s event loop manages Promises (and async I/O), but true parallelism via Worker threads/processes (like Python’s pools).

---

## **Advanced Techniques**

- `asyncio.gather` vs **TaskGroup**:
    - `gather`: Continues running even if some tasks fail if `return_exceptions=True`.
    - **TaskGroup**: Cancels all remaining tasks on first failure, gives error group.
- Async context managers (`async with ...`): Used wherever setup/teardown involves I/O (like TaskGroup, file/network/database).
- Async iterators: “async for” allows loop bodies to await each iteration—critical for streaming data or chunked reads.

---

## **How to Choose: AsyncIO vs Threads vs Processes**

- **AsyncIO:** If the task/library is async-compatible *and* I/O-bound.
- **Threads:** For sync, I/O-bound work without async library support.
- **Processes:** For CPU-bound tasks; true parallel Python execution.

---

## **Summary for Fast Recall**

- **Event loop**: Schedules and runs tasks cooperatively; each coroutine/task yields via `await`.
- **Coroutines must be scheduled before awaiting for concurrency.**
- **Never block event loop with sync code; use threads or processes for those.**
- **Semaphores**: Limit number of concurrent coroutines/tasks.
- **Profiling** is key to knowing what to optimize—distinguish I/O from CPU work.
- Use latest async-compatible libraries wherever possible for fastest, most scalable performance.

**Emphasized from our conversation:**

- *asyncio.to_thread* = for *sync/blocking* functions only
- *You must `await` coroutine or task for it to run!*
- Limiting concurrency (semaphores) prevents resource exhaustion in the event loop.
- Node.js Promises and Python coroutines look similar but are scheduled differently; don’t confuse “parallel” with true multicore concurrency unless using threads/processes.

---

This summary is **optimally structured for quick review and recall**—refer to it whenever refreshing your memory about Python AsyncIO’s real-world, architectural, and advanced programming concepts!

1. [https://www.youtube.com/watch?v=oAkLSJNr5zY&t=3498s](https://www.youtube.com/watch?v=oAkLSJNr5zY&t=3498s)
