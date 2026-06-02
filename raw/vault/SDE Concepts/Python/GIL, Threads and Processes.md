# GIL, Threads and Processes

YT:[https://www.youtube.com/watch?v=-NONm-Jq34Y](https://www.youtube.com/watch?v=-NONm-Jq34Y)

YT:[https://www.youtube.com/watch?v=ITc09gOrqZk](https://www.youtube.com/watch?v=ITc09gOrqZk)

YT:[https://www.youtube.com/watch?v=hwTYDQ0zZOw](https://www.youtube.com/watch?v=hwTYDQ0zZOw)
YT:[https://www.youtube.com/watch?v=yBX89xUmnyk](https://www.youtube.com/watch?v=yBX89xUmnyk)

Here’s a **comprehensive, pointwise summary** covering all core concepts from your linked videos, explicitly connecting to your questions, doubts, and the topics we discussed in this conversation:

---

## 1. **CPU Cores vs Threads: Physical and Logical Cores**

- **CPU Cores**: Physical units in your processor; each core can execute instructions independently.
- **Threads**: Logical pathways for work; the OS treats each as a "unit of work" that can be scheduled on cores.
- **Physical Core**: Actual hardware that executes tasks.
- **Logical Processor**: What the OS sees; may be more than physical cores due to SMT/Hyper-Threading.
    - Example: 4 cores / 8 threads = 4 physical, 8 logical.
- **Multithreading (SMT/Hyper-Threading)**: Allows a single core to handle multiple threads by switching efficiently.
- **Parallel vs Concurrent Execution**: Multiple cores = true parallelism; multiple threads on one core = concurrency.
- **Efficiency**: Extra threads help fill gaps in core downtime but can’t replace actual cores.
- **Your Doubt Clarified**: Having 12 physical and 12 logical means NO SMT/Hyper-Threading; each core executes only one thread at a time.

---

## 2. **Process vs Threads in OS**

- **Process**: Heavyweight; independent program with its own memory, code, data, stack, registers.
- **Thread**: Lightweight; exists within a process, shares code/data, but has its own stack/registers.
- Creating a **new process** copies code/data (overhead); creating **threads** shares code/data within same process (minimal overhead).
- **System Calls**: Processes may use `fork()` (Linux) to create new processes via the kernel; threads can be created in user space or with system calls.
- **OS Scheduling**: Processes are managed independently; threads within a process can be managed by the application or OS.
- **Blocking Behavior**:
    - **Process Block**: If a process is blocked (waiting for I/O), its child process may continue independently.
    - **Thread Block (User-level)**: If any one thread is blocked (user-level threads), often **entire process is blocked**—important distinction.
- **Context Switching**: Slower for processes due to more state; faster for threads (less context to switch).

---

## 3. **User-Level vs Kernel-Level Threads**

- **User-Level Threads**: Managed by the application/library (e.g., Python’s green threads, older Java).
    - Fast to create/destroy.
    - OS sees only the main process, not the threads inside.
    - If one thread blocks in a system call, the **whole process can be blocked** (your confusion resolved here).
- **Kernel-Level Threads**: Managed by the OS (e.g., POSIX threads, Windows threads).
    - OS schedules each thread independently.
    - If one thread blocks (I/O), **other threads keep running** (modern Linux, Windows).
    - Slower context switching than user-level threads due to OS involvement.
- **Thread Models**: One-to-one (each user thread maps to kernel thread), many-to-one, many-to-many mapping; modern systems use one-to-one.

---

## 4. **Python’s Global Interpreter Lock (GIL)**

- **GIL**: Mutex in CPython that allows only one thread to execute Python bytecode at a time.
    - Necessary for memory management/thread-safety.
    - **Limits true parallelism** for CPU-bound threads—multi-threaded Python code won’t fully utilize multiple CPU cores.
    - **I/O-bound programs**: GIL is released during I/O, so other threads may run (use threads for network/disk waits).
    - To achieve parallelism for CPU-bound tasks in Python, use multiprocessing (spawns processes instead of threads).
- **Implications for Your Questions**:
    - Python’s standard `threading` module uses kernel threads, but GIL restricts true parallelism.
    - Thread blocking due to I/O allows other threads to progress in Python because the GIL is released.

---

## **Practical Summaries & Your Doubts Emphasised**

- **How Many Processes/Threads Can CPU Run?**
    - A 12-core CPU can run 12 concurrent threads/processes truly in parallel; OS time-slices for more than 12.
- **Process vs Thread Structure**
    - Process: Isolated, heavy, separate memory/code.
    - Thread: Shared memory/code, independent stack/register.
- **Can Multiple Threads Run if One Blocks?**
    - **User-level threads**: Often not; all threads can block.
    - **Kernel-level threads**: Yes; only the blocked thread stops.
    - **Python GIL**: For CPU-bound Python code, GIL restricts to one at a time.
- **How Are Threads Implemented?**
    - User: By libraries, not visible to OS.
    - Kernel: OS schedules individual threads.
    - Example: `pthread_create()` (C), `threading.Thread()` (Python creates kernel threads but is restricted by GIL).

---

## **Quick Reference Table**

| Concept | Process | Thread | User-level thread | Kernel-level thread | Python GIL Note |
| --- | --- | --- | --- | --- | --- |
| Scheduling | By OS, independently | By OS or app, within process | Managed by app/library | Managed by OS | Kernel thread + GIL |
| Creation Overhead | High (full copy) | Low (shares code/data) | Very low | Higher (OS system call) | N/A |
| Memory Sharing | No | Yes (code/data) | Yes | Yes | Yes (GIL limits exec) |
| Blocked by I/O | No effect on others (usually) | Can block all (user-level) | All threads may block | Only blocking thread | Python: GIL released on I/O |
| Context Switching | Slow | Fast | Fast | Slower than user-level | Fast, but GIL bottleneck |

---

## **Final Summary**

- **CPU cores** determine hardware parallelism; **threads** and **processes** are software abstractions.
- **User-level vs kernel-level threads**: User-level (managed by app, fast, but all block together); kernel-level (managed by OS, slower, but each thread acts independently).
- **Python’s GIL**: Makes Python threading different—use multiprocessing for true CPU parallelism.
- Your system (12 cores/12 threads) means true parallel capacity with one thread per core, no SMT/Hyper-Threading.
- If a thread blocks (user-level), all threads in process may block; with kernel threads (modern OS/Python for I/O), only that thread blocks.
- Remember context switching costs, memory sharing strategies, and blocked thread behavior—all depend on implementation (user vs kernel).

---

**Use this as your refresh note—highlighted wherever you had doubts and connecting all the above video concepts into one accessible summary.**

1. [https://www.youtube.com/watch?v=hwTYDQ0zZOw](https://www.youtube.com/watch?v=hwTYDQ0zZOw)

User threads are basically virtual threads that the program itself handles- to an OS- it’s not a thread and is a process like any other. kernel threads are threads on an os level and that’s why OS understands that it’s a thread and therefore multithreading’s natively supported in this case.

green threads = user threads
