# Trails Log

Append-only record of all ingests and significant changes. Most recent at bottom.

Quick grep: `grep "^## \[" log.md | tail -10`

---

## [2026-05-30] ingest | Hash Table Resizing and Optimizations
Placed in: Trail 01 — Data Structures & Hashing (position 5, after "Linear, Quadratic, and Double Hashing")
Summary: deep dive on resize mechanics — the O(N²) vs O(1) amortized cost proof, why doubling works, the power-of-two bitwise AND trick (hash % M ≡ hash & (M-1)), and why the shrink threshold is 1/8 (not 1/4) to avoid thrashing.

---

## [2026-05-06] init | Initial vault setup
Converted Notion export (84 notes) to Obsidian vault. Created 12 focused trails + 1 mega trail covering all notes. All notes placed on at least one trail.

Trails created:
- 00 Mega Trail (55 nodes, full career arc)
- 01 Python Runtime & Concurrency (8 nodes)
- 02 NodeJS Runtime (4 nodes)
- 03 Networking (5 nodes)
- 04 Backend Patterns (10 nodes)
- 05 Databases & Storage (9 nodes)
- 06 Scaling & Distributed Systems (7 nodes)
- 07 System Design (4 nodes)
- 08 DevOps & Infrastructure (4 nodes)
- 09 Frontend (4 nodes)
- 10 Security (4 nodes)
- 11 AI & Modern Stack (6 nodes)
- 12 WellnessLiving Work (10 nodes)
