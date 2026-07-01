# Chapter 6: Partitioning

> **DDIA · Part II — Distributed Data**
> 
> _"We must break away from the sequential and not limit the computers."_ — Grace Murray Hopper

---

## Why partition?

Partitioning (a.k.a. _sharding_) = breaking a large dataset into smaller subsets, each stored on a different node. Unlike replication (Ch. 5), which copies the _same_ data everywhere, partitioning splits data so each piece lives in exactly one partition.

|Goal|How partitioning helps|
|---|---|
|**Scalability**|Spread data across many disks; spread query load across many CPUs|
|**Throughput**|Each node independently executes queries for its own partitions|
|**Parallelism**|Complex queries can be parallelized across nodes (harder in practice)|

> **Terminology varies:** MongoDB/Elasticsearch call it a _shard_, HBase a _region_, Bigtable a _tablet_, Cassandra/Riak a _vnode_, Couchbase a _vBucket_. All mean the same thing.

> **Not the same as network partitions:** Partitioning is _intentional_ data splitting. Network partitions (netsplits) are faults — covered in Ch. 8.

---

## Partitioning + Replication

These two concepts are usually combined:

- Each record belongs to **exactly one partition**.
- Each partition is **replicated** across multiple nodes (for fault tolerance, per Ch. 5).
- A node can be **leader** for some partitions and **follower** for others simultaneously.
- The choice of partitioning scheme is **independent** of the replication scheme.

---

## Partitioning of Key-Value Data

The fundamental question: **how do you decide which records go to which partition?**

Goal: spread data and query load **evenly**. Uneven distribution = _skew_. A partition with disproportionately high load = _hot spot_.

Random assignment distributes evenly but makes reads impossible (you'd have to query all nodes). We need smarter approaches.

### Strategy 1: Key range partitioning

Assign a continuous range of keys to each partition (like volumes of an encyclopedia: A-B in volume 1, C-D in volume 2, etc.).

**How it works:**

- Boundaries don't have to be evenly spaced — they adapt to the data distribution.
- Within each partition, keys are kept in sorted order.
- Range scans are efficient (fetch all sensor readings for a month).
- Boundaries can be chosen manually or automatically.

**Used by:** Bigtable, HBase, RethinkDB, MongoDB (pre-2.4).

**The hot spot problem:** If the key is a timestamp, all current writes hit the same partition (today's date range). Fix: prefix the timestamp with something like sensor name — `(sensor_name, timestamp)`. Writes spread across sensors, but range queries now need one query per sensor.

### Strategy 2: Hash partitioning

Apply a hash function to each key; assign each partition a range of hashes.

**How it works:**

- Good hash function → even distribution regardless of input pattern.
- Hash need not be cryptographic — Cassandra/MongoDB use MD5, Voldemort uses Fowler–Noll–Vo.
- Partition boundaries can be evenly spaced or pseudorandom ("consistent hashing").

**Caveat on "consistent hashing":** Despite the name, this doesn't work well for databases in practice. The term is confusing (unrelated to replica consistency or ACID consistency). Better to just call it _hash partitioning_.

**Used by:** Most Dynamo-style systems.

**The trade-off:** Destroys sort order. Keys that were adjacent are now scattered. Range queries must go to all partitions. MongoDB with hash sharding: range queries on primary key are not supported by Riak, Couchbase, or Voldemort.

### Cassandra's compound key compromise

A table's primary key has multiple columns. The **first column** is hashed (determines the partition). **Remaining columns** are used as a concatenated index for sorting within the partition.

Example: primary key `(user_id, update_timestamp)`:

- Can't range-scan across users (user_id is hashed).
- Can efficiently fetch all updates for one user within a time range (timestamp is sorted within the user's partition).

This enables elegant one-to-many data models: different users on different partitions, but each user's data sorted by time on a single partition.

### Skewed workloads and hot spots

Hashing reduces hot spots but can't eliminate them entirely. If all reads/writes target the same key (e.g., a celebrity's user ID on social media), all requests go to one partition regardless of hashing.

**Application-level workaround:** Append a random 2-digit number to hot keys → splits writes across 100 keys on different partitions. But reads must query all 100 and combine. Only worth it for the small number of truly hot keys — overhead is wasted on normal keys. Most databases don't auto-compensate; it's the application's responsibility.

---

## Partitioning and Secondary Indexes

Primary key access is straightforward — hash or range the key, route to the right partition. Secondary indexes complicate things because they don't map neatly to partitions.

### Approach 1: Document-partitioned indexes (local index)

Each partition maintains its **own** secondary index, covering **only** the documents in that partition.

- **Write:** Update one partition only (the one containing the document).
- **Read:** Must query **all** partitions and combine results (_scatter/gather_).
- Scatter/gather is prone to **tail latency amplification** (slowest partition determines response time).

**Used by:** MongoDB, Riak, Cassandra, Elasticsearch, SolrCloud, VoltDB.

> Vendors recommend structuring partitioning so secondary index queries can be served from a single partition — but not always possible, especially with multiple filters.

### Approach 2: Term-partitioned indexes (global index)

A global index covering all partitions, itself partitioned by the indexed term (e.g., `color:red` in one index partition, `color:silver` in another).

- **Read:** Query just the index partition containing the term you want. Efficient.
- **Write:** A single document write may affect multiple index partitions (each term might be on a different partition → need distributed transaction or accept async updates).

In practice, global secondary index updates are often **asynchronous** — reads right after a write may not reflect the change. DynamoDB: "updated within a fraction of a second in normal circumstances, but may experience longer propagation delays."

**Used by:** DynamoDB, Riak search, Oracle data warehouse.

### The core trade-off

||Document-partitioned (local)|Term-partitioned (global)|
|---|---|---|
|**Writes**|Fast (single partition)|Slow (multiple partitions)|
|**Reads**|Slow (scatter/gather)|Fast (single index partition)|

---

## Rebalancing Partitions

Things change: more throughput needed (add CPUs), more storage needed (add disks), machines fail (redistribute). Moving load between nodes = **rebalancing**.

**Requirements for any rebalancing:**

1. Load shared fairly after rebalancing.
2. Database continues accepting reads and writes during rebalancing.
3. Minimize data movement (network and disk I/O).

### How NOT to do it: `hash(key) mod N`

If N (number of nodes) changes, most keys move.

**Concrete example — a WellnessLiving-like scenario:**

Imagine you have booking records distributed across 10 nodes using `hash(booking_id) mod 10`:

|Key|hash(key)|mod 10|mod 11|mod 12|
|---|---|---|---|---|
|`booking_42`|123456|Node **6**|Node **3** ✗|Node **0** ✗|
|`order_99`|789012|Node **2**|Node **3** ✗|Node **0** ✗|
|`session_A`|901234|Node **4**|Node **0** ✗|Node **2** ✗|
|`class_12`|678901|Node **1**|Node **5** ✗|Node **5** ✗|
|`payment_1`|445566|Node **6**|Node **6** ✓|Node **6** ✓|

✗ = key must migrate. Going from 10→11 nodes, **~90% of keys move**. Going to 12, most move again. Only the rare key whose `hash mod old_N == hash mod new_N` stays put. This is why it's catastrophic — you'd be shuffling nearly your entire dataset over the network every time you add a single machine.

**The root cause:** The mod operation fundamentally changes when the divisor changes. There's no stability guarantee.

### Strategy 1: Fixed number of partitions

Create **many more partitions** than nodes upfront (e.g., 1000 partitions on 10 nodes = ~100 per node). Keys are permanently assigned to partitions; only the partition-to-node mapping changes.

**Walkthrough example — 12 partitions, 3 nodes:**

**Initial state:** P0-P3 → Node 1, P4-P7 → Node 2, P8-P11 → Node 3. Each node has 4 partitions.

**Add Node 4:** The new node "steals" one partition from each existing node:

- Node 1 gives up P3 → Node 4
- Node 2 gives up P7 → Node 4
- Node 3 gives up P11 → Node 4
- Result: Nodes 1-3 have 3 partitions each, Node 4 has 3. **Only 3 out of 12 partitions moved (25%).**

**Remove Node 2:** Its 4 partitions are redistributed:

- P4 → Node 1, P5 → Node 3, P6 → Node 1, P7 → Node 3
- Result: **Only those 4 partitions move.** The other 8 stay exactly where they are.

**Key insight:** A user whose booking hashes into P3 always goes to P3. Before rebalancing they talked to Node 1; after, they talk to Node 4. The key-to-partition mapping is permanent. During the transfer, the old node continues serving reads/writes for that partition until the move completes.

**Why "fixed" is tricky:**

- The number of partitions is decided at setup time and (usually) never changes.
- This means the partition count is your **maximum possible node count** — you can't scale beyond it.
- Too few partitions → can't scale. Too many → each partition has management overhead (memory for metadata, file handles, etc.).
- If you start with 100 GB of data in 1000 partitions (100 MB each), and grow to 100 TB, each partition is now 100 GB — rebalancing one partition means moving 100 GB over the network. If you started with 10,000 partitions for safety, but only have 100 MB of data initially, each partition is 10 KB — the overhead of 10,000 partition metadata entries dwarfs the actual data.

**Think of it like pre-allocating a hash map with a fixed bucket count** — fine if you guess right, wasteful or limiting if you don't.

**Used by:** Riak, Elasticsearch, Couchbase, Voldemort.

### Strategy 2: Dynamic partitioning

Partitions split when they exceed a configured size, merge when they shrink below a threshold. Exactly like B-tree page splitting (Ch. 3).

**Walkthrough example — HBase with 10 GB split threshold:**

1. **Empty database:** One partition (P0), one node. All writes go here. Other nodes idle.
2. **P0 reaches 10 GB:** Splits into P0a (keys A-M, ~5 GB) and P0b (keys N-Z, ~5 GB). P0b transferred to Node 2.
3. **P0a reaches 10 GB:** Splits again into P0a1 and P0a2. One half moves to Node 3.
4. **Data deleted from P0b:** Shrinks to 1 GB. Merged with adjacent partition P0a2. Node 2 can be decommissioned.

The partition count **adapts to data volume**: small data = few partitions = low overhead; large data = many partitions = parallelism. This is the advantage over fixed partitioning.

**The cold start problem:** Stage 1 above is painful. One partition on one node means no parallelism until the first split. Fix: **pre-splitting** — configure initial partition boundaries on an empty database (e.g., HBase lets you specify split points, MongoDB does it automatically). But pre-splitting requires knowing your key distribution upfront.

**Real-world analogy:** Imagine a new WellnessLiving deployment starting empty. All bookings, classes, events — everything goes to one partition on one server. As the business grows and data accumulates, partitions split automatically and spread across servers. If a studio chain closes and data is archived/deleted, partitions merge back down.

**Works with both partitioning strategies:** MongoDB ≥2.4 supports dynamic splitting for both key-range and hash-partitioned data.

**Used by:** HBase, RethinkDB, MongoDB (≥2.4).

### Strategy 3: Proportional to nodes

Fixed number of partitions **per node** (e.g., Cassandra: 256 per node). Total partition count = 256 × node count.

**Walkthrough example — 3 nodes, 256 partitions each = 768 total:**

1. **Add Node 4:** It randomly picks 256 existing partitions (out of 768), splits each in half, takes one half. Now there are 768 + 256 = 1024 partitions across 4 nodes.
2. **The randomness:** Unlike fixed partitioning where the admin or system carefully chooses which partitions move, this is random. A few splits may be unfair (one node loses more data than another), but with 256 splits happening at once, it averages out.
3. **Cassandra 3.0 improvement:** Introduced a deterministic algorithm that avoids unfair splits by analyzing the token ring distribution before choosing where to split.

**How it compares:**

||Fixed|Dynamic|Proportional to nodes|
|---|---|---|---|
|**Partition count**|Fixed at setup|Adapts to data size|Adapts to cluster size|
|**Partition size**|Grows with data|Bounded (split/merge)|Grows with data per node|
|**Requires upfront config**|Yes (hard to choose right)|Only if pre-splitting|No|
|**Empty DB problem**|No (partitions pre-exist)|Yes (starts with 1)|No|

**Used by:** Cassandra, Ketama.

### Automatic vs. manual rebalancing

|Approach|Pro|Con|
|---|---|---|
|**Fully automatic**|Less operational work|Unpredictable. Can cause cascading failures.|
|**Human-in-the-loop**|Prevents operational surprises|Slower|

**The cascading failure scenario (critical to understand):**

1. Node 3 is under heavy load from a burst of booking requests. It responds slowly.
2. Automatic failure detector thinks Node 3 is dead (timeout exceeded).
3. Automatic rebalancer kicks in: starts moving Node 3's partitions to Nodes 1 and 2.
4. This rebalancing generates massive network I/O and disk writes on Nodes 1 and 2 (they're receiving data) AND on Node 3 (it's still alive and now serving reads/writes AND shipping data out).
5. Nodes 1 and 2 slow down under the extra load. Node 3 gets even slower.
6. Failure detector might now think Nodes 1 or 2 are also dead → more rebalancing → system collapses.

This is the same pattern as the Ch. 5 failover problem — automatic detection + automatic action without human judgment can spiral. Couchbase, Riak, and Voldemort solve this by generating a _suggested_ partition reassignment that an admin must approve before it takes effect.

> **Same pattern as Ch. 5 failover:** Automatic detection + automatic action = cascading failure risk. Human oversight is safer.

---

## Request Routing

How does a client know which node holds the partition for a given key? This is **service discovery** — a general distributed systems problem.

### Three approaches

|Approach|How it works|Used by|
|---|---|---|
|**Contact any node**|Node forwards to the correct one if it doesn't own the partition|Cassandra, Riak (gossip protocol)|
|**Routing tier**|Partition-aware load balancer in front of the cluster|HBase, Kafka, MongoDB (via ZooKeeper / config servers)|
|**Smart client**|Client knows the partition map, connects directly to the right node|Some client libraries cache the partition map|

### ZooKeeper-based coordination

Many systems use ZooKeeper (or equivalent) as the source of truth for partition-to-node mapping:

- Each node registers in ZooKeeper.
- ZooKeeper maintains authoritative mapping.
- Routing tier / smart clients subscribe to changes → notified when partitions move.

**Used by:** LinkedIn Espresso (via Helix), HBase, SolrCloud, Kafka.

### Gossip protocol approach

Cassandra and Riak disseminate cluster state via gossip among nodes — no external coordination service. Any node can handle any request (forwards if needed). More complexity in the DB, but no ZooKeeper dependency.

### Parallel query execution

Most NoSQL systems support only simple key lookups or scatter/gather. MPP (Massively Parallel Processing) data warehouse systems (Teradata, etc.) break complex queries (joins, aggregations) into execution stages distributed across nodes — covered more in Ch. 10.

---

## Chapter Summary

### Two main partitioning strategies

|Strategy|Key property|Advantage|Risk|Rebalancing|
|---|---|---|---|---|
|**Key range**|Keys sorted within partitions|Efficient range queries|Hot spots (monotonic keys like timestamps)|Typically dynamic (split/merge)|
|**Hash**|Keys uniformly distributed by hash|Even load distribution|No range queries (sort order destroyed)|Typically fixed partition count|

**Hybrid:** Cassandra's compound key — hash the first column, sort on the rest.

### Two secondary index approaches

|Approach|Alias|Write cost|Read cost|
|---|---|---|---|
|**Document-partitioned**|Local index|Low (one partition)|High (scatter/gather all partitions)|
|**Term-partitioned**|Global index|High (multi-partition, often async)|Low (one index partition)|

### Key concepts to remember

- **Skew and hot spots** are the central problems. All strategies are trade-offs around distributing load evenly.
- **Rebalancing** must be done carefully — `hash mod N` is wrong, fixed/dynamic/proportional strategies each have trade-offs, and full automation is dangerous.
- **Request routing** is service discovery — solved via ZooKeeper coordination or gossip protocols.
- **Partitions operate independently** by design — that's what enables scaling. But cross-partition operations (writes touching multiple partitions) are hard → addressed in Ch. 7 (transactions).

### Forward references

|Topic|Where|
|---|---|
|What if a write to one partition succeeds but another fails?|Chapter 7 (Transactions)|
|Consensus for routing decisions|Chapter 9|
|Parallel query execution in data warehouses|Chapter 10|
|Term-partitioned secondary index implementation|Chapter 12|

### Connection to Chapter 5

- Replication handles **copies** of the same data. Partitioning handles **splitting** data.
- Combined: each partition is replicated, each node leads some partitions and follows others.
- Rebalancing echoes the failover discussion: automatic action without human oversight can cascade.
- Global secondary indexes introduce eventual consistency — same as async replication lag in Ch. 5.

---

_Notes from Designing Data-Intensive Applications by Martin Kleppmann, Chapter 6: Partitioning._