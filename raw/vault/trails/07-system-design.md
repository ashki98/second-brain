# Trail: System Design

**Objective:** Practice the end-to-end skill of designing large-scale systems — from requirements to architecture — using increasingly complex case studies.
**Prerequisites:** Trail 05 — Databases & Storage, Trail 06 — Scaling & Distributed Systems
**Review time:** ~35 min
**Nodes:** 4

---

1. [[System Design Primer]]
   > The vocabulary and trade-off framework before you attempt any case study. This note covers: scaling patterns (vertical vs horizontal), CAP theorem, consistency models, latency numbers, back-of-envelope estimation, and the standard interview structure. Don't skip this — the next three notes assume you speak this language.

2. [[Design URL Shortner]]
   > The canonical beginner case. Simple domain, but teaches everything: hashing vs sequential IDs, read-heavy caching, DB choice (key-value vs relational), redirect handling (301 vs 302), rate limiting. Work through this one with pencil and paper before checking your answers against the note.

3. [[Design TikTok]]
   > The difficulty jumps here. Video upload/transcoding pipelines, CDN distribution, social graph, personalized feed generation, and the scale requirements of a billion-user platform. Every subsystem you learned in Trails 05 and 06 gets exercised: object storage, Kafka for async processing, Redis for feeds, sharded DBs.

4. [[Google Docs System Design – Interview Summary]]
   > The hardest case: real-time collaborative editing. Multiple users editing the same document simultaneously requires either Operational Transforms (OT) or CRDTs to resolve conflicts. This note is the frontier of distributed systems thinking — if you can explain Google Docs, you can explain almost any distributed system.
