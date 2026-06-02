# Trail: Backend Patterns & Architecture

**Objective:** Build a vocabulary of backend architectural patterns — from the basics of MVC to the advanced separation of CQRS — grounded in real framework implementations.
**Prerequisites:** Trail 03 — Networking (for HTTP/request context)
**Review time:** ~40 min
**Nodes:** 10

---

1. [[MVC]]
   > Start with the pattern that structures nearly every web framework. MVC separates concerns into Model (data), View (presentation), and Controller (logic). This note establishes the baseline — before you can appreciate why other patterns were invented, you need to deeply understand what MVC does and where it strains.

2. [[SOLID]]
   > The five principles behind MVC and most other good architecture. SOLID explains *why* separating concerns is valuable — not just as a convention but as a way to contain change. After this, you'll recognize SOLID violations in code you review, and the subsequent patterns will feel like natural consequences of these principles.

3. [[OOP Questions]]
   > Consolidate: how do OOP concepts (inheritance, polymorphism, encapsulation, abstraction) map to the patterns you just saw? This note is a reference for interview-style questions, but more usefully, it connects abstract OOP theory to the concrete patterns you're building on.

4. [[RPC]]
   > When your MVC app needs to call another service, you have a choice: REST or RPC. This note explains Remote Procedure Call — treating a network call like a local function call — and compares Lambda (serverless, high-latency) to gRPC (high-performance, streaming).

5. [[GRPC]]
   > gRPC takes RPC seriously: Protobuf serialization (binary, typed, compact), HTTP/2 multiplexing, and native streaming. This note covers when gRPC is the right choice over REST — high-throughput microservices, real-time pipelines, strict API contracts.

6. [[JWT]]
   > Stateless authentication: instead of a session on the server, you sign a token that the client presents on every request. This note covers the JWT structure (header.payload.signature), common pitfalls (algorithm confusion, long expiry), and how JWTs wire into the request/response flow you know from Trail 03.

7. [[CQRS]]
   > When MVC starts to strain — read and write models diverge, queries get complex — CQRS separates them entirely. Commands mutate state; queries are read-optimized. This note introduces the pattern and its natural companions (event sourcing, eventual consistency). This is the advanced version of "separate your concerns."

8. [[Fast API (vs Django)]]
   > Framework choice in Python: Django's batteries-included MVC vs FastAPI's async-first, type-safe approach. With SOLID and CQRS in mind, you can now evaluate each framework by its architecture, not just its syntax.

9. [[FastAPI Dependency Injection (Depends) - Comprehen]]
   > FastAPI's `Depends` system is SOLID's Dependency Inversion Principle as a first-class language feature. This note covers how to use DI for auth, DB connections, and reusable logic — making your FastAPI routes thin and testable.

10. [[Express middleware chain and error handling]]
    > Real-world example from Workifi (Node.js/Express): how middleware composes as a pipeline, how errors propagate through it, and what to watch out for. After nine notes of patterns and principles, this is what they look like in a production codebase.
