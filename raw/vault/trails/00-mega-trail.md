# Trail: From Bits to Production — The Full Stack Engineer Path

**Objective:** A single continuous journey from how CPUs work to deploying AI-powered production systems — with your real work woven in at every stage.
**Prerequisites:** None (this trail is self-contained; the focused trails go deeper on each phase)
**Review time:** ~3 hrs (designed for periodic full re-reads, not one sitting)
**Nodes:** 47

---

> **How to use this trail:** Read it top-to-bottom once a month or so. Each phase builds directly on the previous one. When a node feels fuzzy, open the focused trail for that phase and go deeper. Your own work examples appear throughout — they're marked with 🏢.

---

## Phase 1 — The Foundation: How Machines Run Code

*Before frameworks and databases, understand what the computer is actually doing.*

1. [[OS Scheduling, Process and Threads]]
   > Hardware cores, OS threads, context switching. Everything else you learn is running on top of this scheduler.

2. [[Memory Allocation and Management in Python]]
   > Python's heap/stack split, reference counting, GC. Why variables behave the way they do.

3. [[GIL, Threads and Processes]]
   > Python's Global Interpreter Lock: what it is, why it exists, how to work around it.

4. [[Asyncio]]
   > One thread, one event loop, many coroutines — Python's answer to the GIL for I/O-bound work.

---

## Phase 2 — The Network: How Computers Talk

*You write backend code that lives on the network. Understand the plumbing.*

5. [[What happens when you visit a website]]
   > The full end-to-end journey of a single HTTP request. Your mental anchor for everything networking.

6. [[Networking Fundamentals - Protocols, Layers, and W]]
   > OSI model, TCP/IP stack, DNS, TLS. The vocabulary every backend engineer needs.

7. [[Networking Deep Dive — How the Internet Actually Works]]
   > BGP, routing, physical infrastructure. How packets actually traverse the world.

8. [[Status Code]]
   > HTTP status codes — the language your servers speak back to clients.

---

## Phase 3 — Backend Patterns: How Services Are Structured

*The architectural patterns that structure every web application.*

9. [[MVC]]
   > The canonical separation of concerns. The starting point for all backend architecture.

10. [[SOLID]]
    > The five principles that explain *why* patterns like MVC work, and when to break them.

11. [[RPC]]
    > How services call each other — remote procedure calls vs REST.

12. [[GRPC]]
    > High-performance RPC with Protobuf + HTTP/2. When REST isn't enough.

13. [[JWT]]
    > Stateless auth tokens — how requests prove identity without server-side sessions.

14. [[CQRS]]
    > When MVC strains: separate your read and write models. Scales both independently.

---

## Phase 4 — The Server: Python and Node Runtimes

*Two languages, two concurrency models, both in your stack.*

15. [[Event Loop- Role]]
    > Node.js's event loop: one thread, non-blocking I/O, callbacks.

16. [[Event Loop- Deep dive]]
    > libuv phases, microtasks vs macrotasks — the internals of why Node behaves the way it does.

17. [[Worker threads]]
    > When Node's event loop isn't enough: escaping to true parallelism.

18. [[Fast API (vs Django)]]
    > Python framework choice: WSGI multi-process (Django) vs ASGI async (FastAPI).

19. [[FastAPI Dependency Injection (Depends) - Comprehen]]
    > SOLID's Dependency Inversion as a framework feature. Thin routes, testable code.

20. [[Node vs Fastapi and asyncio nuances]]
    > The synthesis: Python asyncio vs Node event loop vs Django WSGI, side by side.

21. [[Express middleware chain and error handling]]
    > 🏢 Real Workifi code: how middleware composes in Node/Express. Theory becomes concrete.

22. [[Core Architecture]]
    > 🏢 Workifi's system architecture — your first startup's full stack.

---

## Phase 5 — Databases: How Data Is Stored

*Persistence, performance, and what happens when you have too much data.*

23. [[SQL vs NoSQL - Tradeoffs]]
    > The fundamental storage choice. Consistency and joins vs flexibility and scale.

24. [[DB Indexing]]
    > How databases find data fast. B-trees, query planners, the read/write trade-off.

25. [[DB Sharding]]
    > Horizontal scaling: when your data outgrows one machine.

26. [[DB Deadlocks]]
    > Concurrency at the DB level: what deadlocks are and how to avoid them.

27. [[Part 1 Foundation of data systems]]
    > DDIA Chapter 1–4: Kleppmann's formal framework for reliability, scalability, maintainability.

28. [[Replication]]
    > DDIA Chapter 5: leader/follower replication, consistency anomalies, conflict resolution.

29. [[Redis Internals]]
    > In-memory store: IO multiplexing, single-threaded event loop, atomic operations.

30. [[Wire Protocols - RESP]]
    > The layer below the Redis API: how commands are serialized on the wire.

31. [[TCP Echo Server Implementation]]
    > Build a raw TCP server. After this, databases aren't black boxes anymore.

---

## Phase 6 — Scaling: When One Server Isn't Enough

*Traffic grows. Systems need to grow with it.*

32. [[Load Balancer]]
    > Distribute traffic across servers. The first scaling move.

33. [[Load Balancer vs API Gateway]]
    > Network-level vs application-level ingress. Standard patterns for combining them.

34. [[API Throttling]]
    > Rate limiting: protect your services from overload and abuse.

35. [[CDN Concepts & AWS CloudFront Developer Flow]]
    > Static assets at the edge. Reduce origin load, reduce latency globally.

36. [[Kafka Distributed Messaging System (Video Summary)]]
    > Async messaging: decouple services, handle spikes, replay events.

37. [[Handling 1M Requests]]
    > Framework benchmarks, Redis clustering, write-behind pattern. What 1M RPS actually requires.

---

## Phase 7 — System Design: Putting It All Together

*Design real systems using everything you've learned.*

38. [[System Design Primer]]
    > The framework and vocabulary for designing large-scale systems.

39. [[Design URL Shortner]]
    > Classic case study: hashing, caching, DB choice in a simple domain.

40. [[Design TikTok]]
    > Video pipelines, social graph, personalized feed. Real scale.

41. [[Google Docs System Design – Interview Summary]]
    > Real-time collaborative editing. The frontier of distributed systems.

---

## Phase 8 — Infrastructure: Shipping to Production

*How code goes from your machine to running reliably at scale.*

42. [[Docker Container vs Registry]]
    > Package your app as a container — the unit of modern deployment.

43. [[Jenkins & CI CD Walkthrough]]
    > Automate build, test, deploy pipelines end to end.

44. [[Github Gitlab CI CD]]
    > Git-native CI/CD: workflows in YAML alongside your code.

45. [[Kubernetes]]
    > Orchestrate containers: scheduling, health checks, rolling deploys.

---

## Phase 9 — AI & Modern Stack: The New Layer

*LLMs are now infrastructure. Understand them the same way.*

46. [[LLMs- Fine Tuning, RAG]]
    > Making LLMs domain-specific: fine-tuning vs RAG vs CAG.

47. [[AI Agents]]
    > LLMs as controllers: reasoning, tool use, memory, and multi-step planning.

48. [[MCP]]
    > The protocol that connects LLMs to tools — including this second brain.

49. [[Langgraph Flow]]
    > 🏢 Production agent systems with LangGraph. Used in your Ripples product at WL.

50. [[Software Is Changing (Again) by Andrej Karpathy]]
    > The macro shift: LLM-native software and what it means for your career.

---

## Phase 10 — Applied: Your Work at WellnessLiving

*All of the above, running in production, solving real business problems.*

51. [[Give Overview]]
    > 🏢 Full Give architecture: Louise (Django+GraphQL), Charity Service (FastAPI), Elevate.

52. [[Why graphql]]
    > 🏢 Why GraphQL was chosen for Louise's data access layer.

53. [[Give Optimisations]]
    > 🏢 Performance work you've done on Give — theory from Phase 6 in your own code.

54. [[GIVE SAML]]
    > 🏢 Enterprise SSO: SAML auth in the Give context.

55. [[Ripples Infra and Setup]]
    > 🏢 WL's AI product infrastructure — Phase 8 + Phase 9 combined.
