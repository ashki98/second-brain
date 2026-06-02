# Trail: NodeJS Runtime & Event-Driven Systems

**Objective:** Understand how Node.js achieves high concurrency on a single thread, and where its model breaks down.
**Prerequisites:** Trail 01 — Python Runtime (for contrast with asyncio)
**Review time:** ~20 min
**Nodes:** 4

---

1. [[Event Loop- Role]]
   > Start with the "what": the event loop is Node's answer to the question "how do I handle thousands of concurrent connections without threads?" This note establishes the mental model — a single thread that delegates I/O to the OS and processes callbacks when they're ready.

2. [[Event Loop- Deep dive]]
   > Now the "how": libuv under the hood, the six phases of the loop (timers → pending callbacks → idle → poll → check → close), and the critical distinction between microtasks (Promise callbacks) and macrotasks (setTimeout, I/O). This note explains the precise order operations execute in — which is what you need to debug async behavior.

3. [[Worker threads]]
   > The event loop is great for I/O but helpless for CPU-bound work — one heavy computation blocks everything. Worker threads give you true parallelism within Node without forking a new process. Compare this to Python's multiprocessing (Trail 01): similar problem, similar solution, different API.

4. [[Node vs Fastapi and asyncio nuances]]
   > The synthesis trail node. With Node's event loop and Python's asyncio both in mind, this note draws the direct comparison: single-threaded async vs multi-process WSGI vs ASGI coroutines. Use this when choosing between a Python and Node backend or explaining a concurrency bug to a teammate.
