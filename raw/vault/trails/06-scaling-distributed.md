# Trail: Scaling & Distributed Systems

**Objective:** Learn the toolbox for taking a system from "one server" to "handles anything thrown at it" — load balancing, messaging, caching, and the architecture of extreme scale.
**Prerequisites:** Trail 05 — Databases & Storage, Trail 03 — Networking
**Review time:** ~35 min
**Nodes:** 7

---

1. [[Load Balancer]]
   > The first piece of infrastructure you add when one server isn't enough: distribute incoming traffic across a pool of servers. This note covers LB algorithms (round-robin, least connections, IP hash), health checks, and sticky sessions. This is node 1 because everything else in this trail sits behind or alongside a load balancer.

2. [[Load Balancer vs API Gateway]]
   > These two are often confused. This note draws the distinction: a load balancer operates at the network level (Layer 4/7), while an API gateway handles application-level concerns (auth, rate limiting, request transformation). It also covers the standard architectural patterns for combining them — LB in front, API Gateway behind; CDN + LB + API Gateway; NLB → API Gateway → ALB.

3. [[API Throttling]]
   > Rate limiting: protect your services from traffic spikes and abuse. This note covers throttling strategies (token bucket, sliding window, fixed window) and where to enforce them (API gateway, application code, Redis-backed counters). With the gateway layer understood from note 2, you can now reason about where throttling belongs.

4. [[CDN Concepts & AWS CloudFront Developer Flow]]
   > Push static assets to edge servers close to users. This note covers how CDNs cache content, cache invalidation strategies, and the CloudFront distribution model. CDNs reduce load on your origin servers — they're the first line of scaling for read-heavy, static content.

5. [[Kafka Distributed Messaging System (Video Summary)]]
   > Decouple your services with async messaging. Kafka's model — producers write to topics, partitions enable parallel consumption, consumer groups track offsets — solves the problem of synchronous dependencies between services. This note covers Kafka's architecture, delivery guarantees (at-least-once, at-most-once, exactly-once), and the zero-copy optimization that makes it fast.

6. [[Handling 1M Requests]]
   > What does the full stack look like at extreme scale? This note benchmarks frameworks (Express at 20k RPS, Fastify at 77k, C++ Drogon at 1.2M), analyzes DB bottlenecks (PostgreSQL, Redis single-instance at 150k, Redis cluster for 1M), and traces the write-behind pattern. It's the capstone of this trail: each earlier note is a tool, this note shows them all working together.

7. [[Performance Improvement Concepts]]
   > The general toolkit: what levers exist for improving performance at every layer — caching strategies, connection pooling, batching, profiling, query optimization. Read this last, after you have concrete examples from notes 1–6 to anchor each concept to.
