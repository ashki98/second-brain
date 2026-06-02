# Trail: Databases & Storage

**Objective:** Understand how databases store, find, and replicate data — from the query planner to the wire protocol — so you can reason about persistence trade-offs from first principles.
**Prerequisites:** Trail 04 — Backend Patterns (for context on data access patterns)
**Review time:** ~45 min
**Nodes:** 9

---

1. [[SQL vs NoSQL - Tradeoffs]]
   > Start with the fundamental choice: relational (structured, consistent, joins) vs non-relational (flexible schema, horizontal scale, eventual consistency). This note maps use cases to storage models. Everything that follows assumes you've made this choice and are now going deeper on how your chosen DB actually works.

2. [[DB Indexing]]
   > The most impactful performance lever in any database. This note covers B-tree and hash indexes, how the query planner uses them, and the cost of over-indexing. After this, "add an index" stops being a magic spell and becomes a reasoned decision about read/write trade-offs.

3. [[DB Sharding]]
   > When your data outgrows one machine: horizontal partitioning. This note covers sharding strategies (range, hash, directory-based), the trade-offs each makes on query flexibility and rebalancing cost, and how sharding interacts with the joins and constraints you have in a relational DB.

4. [[DB Deadlocks]]
   > Concurrency at the DB level: two transactions each waiting for the other to release a lock. This note covers how deadlocks form, how databases detect them (wait-for graphs), and how to prevent them in your application code. A note about what happens when your app's concurrency model collides with the DB's.

5. [[Part 1 Foundation of data systems]]
   > Kleppmann's DDIA gives you the theoretical framework for everything above. Chapter 1–4 cover reliability/scalability/maintainability as design goals, data models (relational vs document), storage engines (SSTables, LSM trees, B-trees), and encoding formats (Protobuf, Avro). This is where your practical DB knowledge gets a formal underpinning.

6. [[Replication]]
   > DDIA Chapter 5: how to keep multiple copies of your data consistent across machines. Single-leader, multi-leader, and leaderless models; replication lag anomalies (read-your-own-writes, monotonic reads); conflict resolution. After sharding (data split across machines) comes replication (data copied across machines) — they're complementary.

7. [[Redis Internals]]
   > Redis achieves speed not through threads but through IO multiplexing: one thread monitors many connections via `epoll`, processes commands as data arrives, never blocks. This note explains why Redis is fast, when it's the right tool, and what its single-threaded model means for your architecture.

8. [[Wire Protocols - RESP]]
   > One layer below the Redis API: RESP (REdis Serialization Protocol). This note covers how commands and responses are encoded on the wire — type prefixes, CRLF termination, binary-safe bulk strings. Understanding RESP is what makes Redis feel less like a black box and more like a designed system.

9. [[TCP Echo Server Implementation]]
   > Build a raw TCP server yourself. After reading about IO multiplexing, wire protocols, and event loops, this note closes the loop by showing you the actual socket, bind, listen, accept, read, write cycle. Once you've implemented an echo server, Redis's architecture isn't abstract anymore.
