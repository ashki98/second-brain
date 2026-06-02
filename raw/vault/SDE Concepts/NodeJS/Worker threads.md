# Worker threads

YT: 

[https://www.youtube.com/watch?v=Vej327jN8WI](https://www.youtube.com/watch?v=Vej327jN8WI)

**Worker Threads in Node.js: Concepts & Key Points (Memory Refresh Summary)**

- **Node.js Single-Threaded Model**
    - JavaScript runs on a single thread by default, managing tasks through the event loop.
    - Efficiently handles I/O operations (file read/write, network requests) by *not* blocking the main thread.
- **libuv and Hidden Thread Pool**
    - Node.js leverages `libuv` for asynchronous I/O.
    - libuv maintains an internal thread pool (default 4 threads) to offload blocking operations (e.g., file system access, DNS, crypto).
    - **Crucially:** These threads do *not* run your JS code—they’re strictly for system-level I/O tasks.
- **Event Loop and Non-blocking I/O**
    - I/O operations are handed off to libuv’s thread pool.
    - Event loop keeps running other requests while awaiting I/O results.
    - Results are returned and callback code (your JS) runs back on the main thread.
    - Modern async/await provides cleaner, non-blocking code.
- **Limitation: CPU-bound Tasks**
    - Heavy computation (math, parsing, encryption, loops) *blocks the main thread*, causing the app to freeze/unresponsive.
    - libuv’s thread pool is *not* used for JS logic and *cannot* help here.

---

**Worker Threads: Solving True Parallelism**

- **What Are Worker Threads?**
    - OS-level threads with their *own* V8 context, event loop, and memory.
    - Run user JS code in parallel to the main thread—no blocking!
    - Ideal for CPU-bound tasks (e.g., data processing, image manipulation, large computations).
- **How Worker Threads Work**
    - You create them using Node’s `worker_threads` module.
    - Main thread remains responsive while computation runs in the worker.
    - Messaging system (events) is used to transfer results/errors between threads.
- **Worker Threads vs. libuv Thread Pool**
    - **Worker threads:** Parallel JS logic, OS threads, solve compute bottleneck.
    - **libuv thread pool:** Shared between all threads for system I/O *only*, never executes JS code.
    - **No direct connection:** Worker threads exist independently of libuv’s thread pool.
    - *We covered this in-depth in this conversation:*
        - **Worker threads do not use, nor are managed by, libuv’s thread pool.**
        - They are separate from the I/O thread pool—except all threads (main + workers) share it for I/O (not for running JS).

---

**Practical Example Shown in Video:**

- Main thread creates a worker for a big loop (e.g., counting to 1 billion).
- Main thread stays snappy (e.g., logs console messages instantly).
- Worker performs heavy computation and returns result asynchronously.

---

**Your Key Doubts, Highlighted:**

- **Is Node truly single-threaded with a thread pool?**
    - Yes, for JS code. The thread pool only helps with system I/O, not JS computation.
- **Do worker threads use thread pools?**
    - No—they are separate OS-level threads dedicated to parallel JS.
- **Any link between worker threads and libuv?**
    - Only indirect: both *can* perform I/O, and all use the shared libuv thread pool for I/O, but worker threads manage their own scheduling for JS code.

---

**Summary: High-Performance Node.js**

- Use regular async code for fast, I/O-bound tasks.
- Use worker threads for *CPU-bound* parallelism—keep main thread and event loop free.
- Both main and worker threads can access async I/O, but they do *not* share user JS tasks via libuv.

*Keep these concepts handy whenever you need a refresh on how Node.js achieves high performance, the core role of libuv, and how/why/when to use worker threads for compute-heavy work!*

1. [https://www.youtube.com/watch?v=Vej327jN8WI](https://www.youtube.com/watch?v=Vej327jN8WI)
