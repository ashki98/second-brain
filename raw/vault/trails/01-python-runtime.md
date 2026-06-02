# Trail: Python Runtime & Concurrency

**Objective:** Understand how Python actually executes your code — from hardware threads to async coroutines — so concurrency bugs stop being mysterious.
**Prerequisites:** None
**Review time:** ~35 min
**Nodes:** 8

---

1. [[OS Scheduling, Process and Threads]]
   > Start here: before Python adds its own layer, understand what the OS gives you. This note covers how your CPU's cores map to OS-level threads, how the scheduler context-switches between processes, and what "parallelism vs concurrency" actually means at the hardware level. Everything Python does with threads and async sits on top of this.

2. [[Memory Allocation and Management in Python]]
   > Now zoom into Python's runtime. Every Python variable is a reference to a heap object — this note explains the heap/stack split, reference counting, and garbage collection. Understanding this is why the next note (GIL) will make sense: reference counts are shared data that must be protected.

3. [[GIL, Threads and Processes]]
   > The GIL exists because CPython's reference counts live on the heap and multiple threads would corrupt them. This note explains why the GIL was introduced, what it prevents, and critically — how to work around it (multiprocessing, C extensions). The limitation it creates is exactly what drives the next note.

4. [[Asyncio]]
   > The GIL makes OS threads unsuitable for I/O concurrency in Python. Asyncio's answer: don't use threads at all. One thread, one event loop, many coroutines that yield on I/O. This note covers the coroutine model, `async/await`, and how the event loop schedules work.

5. [[Node vs Fastapi and asyncio nuances]]
   > Now that you understand asyncio, compare it to how Django (WSGI, multiple processes) and FastAPI (ASGI, single-thread async) handle concurrency. The GIL appears again — multiple processes bypass it; ASGI sidesteps it. This note makes the "choose your framework" decision feel grounded rather than arbitrary.

6. [[Python and JS based Server concepts]]
   > Step back and look at the full server model in both Python and JavaScript. With OS scheduling, GIL, and asyncio in hand, you can now read this note and map each concept back to the mechanism underneath it.

7. [[Language Nuances]]
   > The runtime model predicts a lot of Python's surprising behaviors. This note covers the specific nuances — mutable default arguments, late binding closures, `is` vs `==` — that trip people up. You'll recognize why each one works the way it does.

8. [[Packaging in python]]
   > The outer layer: how Python code is structured into modules, packages, and distributions. After understanding how Python runs code, this covers how Python finds and loads it. A practical note for when you're shipping code rather than just writing it.
