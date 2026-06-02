# Replication

# Chapter 5: Replication

> **DDIA · Part II — Distributed Data**
> 
> 
> *"The major difference between a thing that might go wrong and a thing that cannot possibly go wrong is that when a thing that cannot possibly go wrong goes wrong it usually turns out to be impossible to get at or repair."* — Douglas Adams
> 

---

## Why replicate?

Replication = keeping copies of the same data on **multiple machines** connected via a network. This chapter assumes each machine holds the **full dataset** (partitioning comes in Ch. 6).

| Goal | What it means |
| --- | --- |
| **Low latency** | Data geographically close to users |
| **High availability** | System works even if nodes fail |
| **Read throughput** | Scale out machines serving reads |
| **Disconnected operation** | Work offline, sync later |

> **Core tension:** All difficulty lies in handling *changes* to replicated data. If data never changed, you'd copy once and be done.
> 

Three algorithms for replicating changes: **single-leader**, **multi-leader**, and **leaderless**. Almost all distributed databases use one of these three.

---

## 1. Single-Leader Replication

Also called *active/passive* or *master–slave*.

### How it works

1. One replica is designated the **leader** (master/primary). All client writes go to the leader.
2. Leader writes to its local storage, then sends the change to all **followers** (read replicas / slaves / secondaries) via a **replication log** or **change stream**.
3. Each follower applies writes in the same order as the leader.
4. **Reads** can go to the leader OR any follower. **Writes** are only accepted on the leader.

**Used by:** PostgreSQL (≥9.0), MySQL, Oracle Data Guard, SQL Server AlwaysOn, MongoDB, RethinkDB, Kafka, RabbitMQ HA queues, DRBD.

---

### Synchronous vs. Asynchronous Replication

|  | Synchronous | Asynchronous |
| --- | --- | --- |
| **Mechanism** | Leader waits for follower ACK before confirming write to client | Leader sends change, doesn't wait — confirms immediately |
| **Advantage** | Follower guaranteed up-to-date. If leader fails, data safe on follower | Leader never blocked — can keep writing even if followers fall behind |
| **Disadvantage** | One slow/dead follower blocks ALL writes | Data can be lost if leader fails before replication completes |

**Semi-synchronous (practical default):** One follower is synchronous, rest are async. If the sync follower goes down, an async one is promoted to sync. Guarantees data exists on **≥2 nodes** (leader + 1 sync follower).

**Fully async** is widely used (especially with many or geo-distributed followers) despite the durability risk.

> **Research note:** Chain replication (used in Azure Storage) is a variant that avoids data loss while maintaining good performance.
> 

---

### Setting Up New Followers

You can't just copy files (data is always in flux) and you can't lock the DB (kills availability). The process:

1. **Take a consistent snapshot** of the leader's database (same as backup — no lock needed).
2. **Copy snapshot** to the new follower node.
3. **Follower connects to leader**, requests all changes since the snapshot. Requires the snapshot's exact position in the replication log (PostgreSQL: *log sequence number*; MySQL: *binlog coordinates*).
4. **Once caught up**, follower continues processing the live change stream.

---

### Handling Node Outages

### Follower failure: Catch-up recovery

Simple — follower knows the last transaction it processed from its local log. Reconnects to leader, requests all changes since that point, applies them, and it's caught up.

### Leader failure: Failover

Much trickier. A follower must be promoted to new leader, clients reconfigured, other followers pointed at the new leader.

**Automatic failover steps:**

1. **Detect leader failure** — usually via timeout (e.g., no heartbeat for 30 seconds → assumed dead).
2. **Choose new leader** — election by remaining replicas, or appointed by a controller node. Best candidate = replica with most up-to-date data (minimizes data loss). This is a *consensus problem* (→ Ch. 9).
3. **Reconfigure the system** — clients send writes to new leader. If old leader returns, it must become a follower.

**What can go wrong with failover:**

| Problem | Description |
| --- | --- |
| **Lost writes** | Async follower promoted → old leader's unreplicated writes are discarded. Violates durability expectations. |
| **External system inconsistency** | GitHub incident: out-of-date MySQL follower promoted → reused autoincrement IDs → Redis/MySQL inconsistency → private data leaked to wrong users. |
| **Split brain** | Two nodes both believe they're leader. Both accept writes → data loss/corruption. Safety mechanism: STONITH ("Shoot The Other Node In The Head"). |
| **Timeout tuning** | Too long → slow recovery. Too short → unnecessary failovers during load spikes, making things worse. |

> Many operations teams prefer **manual failover** because of these risks.
> 

---

### Implementation of Replication Logs

Four methods used in practice:

### 1. Statement-based replication

Leader logs every write statement (INSERT, UPDATE, DELETE) and forwards it to followers.

- **Pro:** Simple, compact.
- **Con:** Breaks with nondeterministic functions (`NOW()`, `RAND()`), auto-increment ordering dependencies, triggers with side effects.
- **Used by:** MySQL < 5.1 (legacy). VoltDB (requires deterministic transactions).

### 2. Write-Ahead Log (WAL) shipping

Ship the exact byte-level WAL to followers — they rebuild identical data structures.

- **Pro:** Already exists for crash recovery.
- **Con:** Tightly coupled to storage engine. Can't run different DB versions on leader vs. followers. Blocks zero-downtime upgrades.
- **Used by:** PostgreSQL, Oracle.

### 3. Logical (row-based) log replication

Row-level change records, decoupled from storage engine internals:

- Inserted row → new values of all columns
- Deleted row → primary key (or all column values if no PK)
- Updated row → PK + new values of changed columns
- Transaction → sequence of row records + commit marker

**Pro:** Backward compatible (mixed versions OK), easy for external systems to parse (→ **change data capture**, Ch. 11).

**Used by:** MySQL binlog (row-based mode).

### 4. Trigger-based replication

App-layer: DB triggers log changes to a separate table → external process reads and replicates.

- **Pro:** Maximum flexibility — filter data, cross-DB replication, custom conflict resolution.
- **Con:** Higher overhead, more bugs than built-in replication.
- **Used by:** Oracle GoldenGate, Bucardo (Postgres), Databus (Oracle).

---

## Problems with Replication Lag

Read-scaling (many followers serving reads) only works with **async replication**. But async means followers can fall behind → **stale reads** → **eventual consistency**.

> *"Eventually"* is deliberately vague — replication lag can be a fraction of a second in normal operation, or minutes/hours under load or network problems.
> 

### Three anomalies

### Anomaly 1: Reading your own writes

User submits data (write → leader), then immediately reads from a stale follower → their own data appears lost.

**Need: Read-after-write consistency** (a.k.a. read-your-writes consistency). Guarantees users always see their own submissions. No promises about other users' data.

**Fixes:**

- Read user's own data from leader (e.g., always read own profile from leader, others' profiles from followers)
- Track time of last update: read from leader for 1 minute after any write
- Client remembers timestamp of last write → system ensures replica is at least that fresh (can be logical timestamp like log sequence number, or actual clock — but then clock sync is critical, → Ch. 8)
- **Cross-device:** Need centralized metadata + route all of a user's devices to same datacenter

### Anomaly 2: Monotonic reads

User reads from a fresh replica, then from a stale one → data appears to "go back in time" (e.g., a comment appears then disappears on page refresh).

**Need: Monotonic reads guarantee** — once you've seen data at some point in time, you won't later see it from an earlier point. Stronger than eventual consistency, weaker than strong consistency.

**Fix:** Route each user to the same replica consistently (e.g., hash of user ID).

### Anomaly 3: Consistent prefix reads

Causality violation: Observer sees Mrs. Cake's answer before Mr. Poons's question — because they replicated through different followers at different speeds.

**Need: Consistent prefix reads guarantee** — if writes happen in a certain order, anyone reading them sees them in the same order.

**Fix:** Write causally related data to the same partition. Or use algorithms that explicitly track causal dependencies (→ "happens-before" later in this chapter).

### Solutions for replication lag

Handling these in app code is complex and error-prone. This is why **transactions** exist — the database provides stronger guarantees so apps can be simpler. Single-node transactions existed for a long time; distributed databases often abandoned them claiming they're too expensive — "there is some truth in that statement, but it is overly simplistic." More in Chapters 7 and 9.

---

## 2. Multi-Leader Replication

Multiple nodes accept writes. Each leader forwards changes to all other leaders. Also called *master–master* or *active/active*.

### Use cases

| Use case | How it works |
| --- | --- |
| **Multi-datacenter operation** | Leader per DC. Within DC: normal leader-follower. Between DCs: leaders replicate async. Better performance (writes are local), DC fault tolerance, network fault tolerance. |
| **Offline clients** | Each device = a "datacenter" with local leader. Sync when online. Replication lag can be hours/days. Calendar apps are the classic (and famously broken) example. CouchDB built for this. |
| **Collaborative editing** | Google Docs, Etherpad: each user's browser = local replica. Small unit of change (keystroke) avoids locking. Requires conflict resolution. |

> **Big downside:** Write conflicts. Same data modified on two leaders → must be resolved. Often considered "dangerous territory that should be avoided if possible."
> 

**Tools:** Tungsten Replicator (MySQL), BDR (PostgreSQL), GoldenGate (Oracle). Beware subtle pitfalls with autoincrement keys, triggers, and integrity constraints.

---

### Handling Write Conflicts

Example: Two users edit a wiki page title simultaneously. User 1 changes A→B on leader 1, User 2 changes A→C on leader 2. Both succeed locally. Conflict detected on async replication.

### Conflict resolution strategies

| Strategy | How | Risk |
| --- | --- | --- |
| **Conflict avoidance** ★ | Route all writes for a record to the same leader (e.g., user's "home" datacenter) | Breaks down if you need to reroute traffic (DC failure, user relocation) |
| **Last Write Wins (LWW)** | Attach timestamp, highest wins, discard others | Achieves convergence but **loses data**. Only safe if keys written once (UUID per write) |
| **Replica priority** | Higher-numbered replica always wins | Also loses data |
| **Merge values** | Combine values (e.g., "B/C" as merged title) | Application-specific, can be surprising |
| **Preserve conflict** | Store all versions, resolve later (on read or by user) | Complex for app developers. CouchDB model. |

### Custom resolution logic

- **On write:** Conflict handler called immediately when conflict detected in replication log (e.g., Bucardo's Perl snippet). Must be fast, no user prompts.
- **On read:** All conflicting versions stored. Next read returns all versions to app. App resolves (CouchDB model).

> Conflict resolution applies at the level of individual rows/documents, not entire transactions.
> 

### Automatic conflict resolution (research)

- **CRDTs** (Conflict-free Replicated Datatypes) — data structures (sets, maps, counters) that auto-merge sensibly. Implemented in Riak 2.0.
- **Mergeable persistent data structures** — Git-like history tracking with three-way merge.
- **Operational transformation** — Algorithm behind Google Docs and Etherpad. Designed for concurrent editing of ordered lists (e.g., characters in a document).

### What counts as a conflict?

Obvious: two writes to the same field of the same record. Subtle: two meeting room bookings for the same room at the same time, made on different leaders. More examples in Ch. 7, scalable resolution in Ch. 12.

---

### Multi-Leader Replication Topologies

How writes propagate from one leader to another.

| Topology | Description | Fault tolerance |
| --- | --- | --- |
| **Circular** | Each node forwards to one other (MySQL default) | One node failure breaks the chain |
| **Star / tree** | One root node forwards to all others | Root failure breaks everything |
| **All-to-all** | Every leader sends to every other leader | Best — messages can travel alternate paths |

**Loop prevention (circular/star):** Each write tagged with node IDs it has passed through. Node receiving its own ID → ignores (already processed).

**All-to-all causality problem:** Network speed differences can cause writes to "overtake" — a leader might receive an UPDATE before the INSERT it depends on. Timestamps alone can't fix this (clocks unreliable → Ch. 8). Need **version vectors**.

> Many multi-leader replication systems have poor conflict detection. If using one, read the docs carefully and test thoroughly.
> 

---

## 3. Leaderless Replication

No leader. Client sends writes to **multiple replicas** directly (or via a coordinator that doesn't enforce ordering). Inspired by Amazon's **Dynamo** paper (2007). Used by Riak, Cassandra, Voldemort ("Dynamo-style").

> **Note:** Amazon's DynamoDB is a different architecture (single-leader).
> 

### Writing when a node is down

3 replicas, 1 is down → client writes to all 3 in parallel → 2 OK responses → write is successful. Down node misses the write. When it comes back, reads may return stale data.

**Solution:** Client reads from multiple nodes in parallel. Version numbers determine which value is newer.

### Catching up stale nodes

| Mechanism | How it works | Limitation |
| --- | --- | --- |
| **Read repair** | Client reads from multiple nodes, detects stale response via version numbers, writes newer value back to stale node | Only works for frequently-read values |
| **Anti-entropy process** | Background process scans for differences between replicas, copies missing data | No particular ordering, can have significant delay. Not all systems implement this (e.g., Voldemort doesn't). |

> Without anti-entropy, rarely-read values may be missing from some replicas indefinitely → reduced durability.
> 

---

### Quorums for Reading and Writing

The core formula:

```
w + r > n
```

- **n** = total replicas
- **w** = number of nodes that must ACK a write
- **r** = number of nodes queried for each read

If w + r > n, the read set and write set **must overlap** → at least one read node has the latest value.

| Configuration | Tolerates |
| --- | --- |
| n=3, w=2, r=2 | 1 unavailable node |
| n=5, w=3, r=3 | 2 unavailable nodes |
| w=n, r=1 | Fast reads, but 1 failed node blocks ALL writes |

Common choice: n odd (3 or 5), w = r = (n+1)/2.

Reads and writes are always sent to **all n replicas** in parallel. w and r determine how many we **wait for**.

### Limitations of quorum consistency

Even with w + r > n, stale values can be returned:

- **Sloppy quorum** → w writes land on different nodes than r reads, breaking the overlap guarantee
- **Concurrent writes** → unclear which happened first (LWW → data loss)
- **Concurrent write + read** → undefined whether read returns old or new value
- **Partial write failure** → write succeeds on some replicas (< w), not rolled back on those
- **Node restored from stale backup** → breaks the quorum count
- **Timing edge cases** → see "Linearizability and quorums" (Ch. 9, p. 334)

> You do NOT get read-your-writes, monotonic reads, or consistent prefix reads from quorums alone. Stronger guarantees require transactions or consensus.
> 

### Monitoring staleness

Leader-based: easy — compare follower's log position to leader's. Leaderless: hard — no fixed write order. Research exists on predicting stale-read percentages based on n, w, r, but not yet common practice.

---

### Sloppy Quorums and Hinted Handoff

Network partition → client can't reach the n "home" nodes for a value. Two choices:

| Option | Behavior |
| --- | --- |
| **Strict quorum** | Return errors. Guarantees correct reads. |
| **Sloppy quorum** | Accept writes on reachable non-home nodes temporarily. Higher write availability. |

**Hinted handoff:** "Locked out of your house → stay on neighbor's couch." Once the network heals, temporary writes are sent to the correct home nodes. "Once you find your keys, your neighbor politely asks you to get off their couch and go home."

> A sloppy quorum is **NOT a real quorum**. It's only a durability assurance (data stored on w nodes *somewhere*). No guarantee reads will see latest value until hinted handoff completes.
> 

**Defaults:** Enabled in Riak, disabled in Cassandra and Voldemort.

### Multi-datacenter operation (leaderless)

- **Cassandra/Voldemort:** n includes nodes in all DCs. Client waits for local DC quorum only. Cross-DC writes happen async.
- **Riak:** n = replicas within one DC. Cross-DC replication happens async in background (similar to multi-leader).

---

### Detecting Concurrent Writes

Even with strict quorums, conflicts arise. Events arrive in different order at different nodes due to variable network delays and partial failures.

### Last Write Wins (LWW)

Force ordering via timestamps. Highest timestamp wins, others discarded.

- Achieves convergence but **loses data silently** — even non-concurrent writes can be dropped due to clock skew.
- Only safe if each key is written **once** and treated as immutable (e.g., UUID per write in Cassandra).
- Cassandra's **only** conflict resolution method. Optional in Riak.

### The "happens-before" relationship

- **A happens before B** if B knows about / depends on / builds upon A.
- Two operations are **concurrent** if neither knows about the other.
- **Not about physical time** — about causal awareness. Two ops can be "concurrent" even if they're minutes apart (if network prevented awareness).

> Connection to physics: similar to special relativity — two events that can't causally affect each other are concurrent regardless of when they physically occur.
> 

Three possibilities for any two operations A and B: A happened before B, B happened before A, or A and B are concurrent.

### Version number algorithm (single replica)

Using the **shopping cart** example — two clients concurrently adding items:

1. Server maintains a **version number per key**, increments on every write.
2. Client **must read before writing** (receives all current values + latest version number).
3. Client **merges** all received values, adds its own change, sends back with the version number from its prior read.
4. Server **overwrites** values at that version or below (they've been merged). **Keeps** values at higher versions (those are concurrent).
5. Write without a version number → concurrent with everything → kept as a sibling.

### Merging sibling values

Clients must merge concurrent values ("siblings" in Riak terminology). Same problem as multi-leader conflict resolution.

- **Shopping cart (add-only):** Take the union of siblings.
- **With deletions:** Can't just take the union — removed items reappear. Use **tombstones** (deletion markers with version numbers). Amazon's shopping cart famously had this bug — items reappeared after removal.
- **CRDTs** (Riak 2.0) can auto-merge siblings, including preserving deletions.

### Version vectors (multiple replicas)

Single version number isn't enough when multiple replicas accept writes concurrently. Need a **version number per replica per key**.

- Each replica increments its own version number on writes.
- Tracks version numbers seen from other replicas.
- The collection of all version numbers = **version vector**.
- Sent to clients on reads, returned on writes (Riak calls it **"causal context"**).
- **Dotted version vectors** (Riak 2.0) are the most refined variant.

> **Version vectors ≠ vector clocks.** The difference is subtle but real — version vectors are the right structure for comparing replica state. See the referenced papers for details.
> 

---

## Chapter Summary

### Three replication approaches

| Approach | Writes go to | Conflict resolution | Complexity | Use when |
| --- | --- | --- | --- | --- |
| **Single-leader** | One leader → followers | None needed (sequential writes) | Low | Default choice. Easy to reason about. |
| **Multi-leader** | Any of several leaders → other leaders + followers | Required (concurrent writes on different leaders) | High | Multi-DC, offline clients, collaborative editing |
| **Leaderless** | Multiple replicas directly | Required (concurrent writes, read repair, hinted handoff) | High | High availability + low latency, tolerate occasional stale reads |

### Consistency guarantees (weakest → strongest)

```
Eventual consistency → Monotonic reads → Read-your-writes → Consistent prefix reads → Strong consistency
```

Stronger guarantees require **transactions** (Ch. 7) or **consensus** (Ch. 9).

### Key trade-offs

- **Sync vs. async replication:** Sync = durable but blocks on slow followers. Async = fast but risks data loss on leader failure.
- **Availability vs. consistency:** Multi-leader and leaderless gain fault tolerance at the cost of weaker consistency.
- **Application complexity vs. database guarantees:** Without transactions, apps must handle replication anomalies themselves.

### Threads that continue in later chapters

| Topic | Where |
| --- | --- |
| Partitioning (splitting data across machines) | Chapter 6 |
| Transactions (stronger guarantees) | Chapter 7 |
| Unreliable clocks and network faults | Chapter 8 |
| Consensus (getting nodes to agree) | Chapter 9 |
| Change data capture | Chapter 11 |
| Conflict resolution at scale | Chapter 12 |

---

*Notes from Designing Data-Intensive Applications by Martin Kleppmann, Chapter 5: Replication.*
