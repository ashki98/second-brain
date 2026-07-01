# Chapter 7: Transactions

> **DDIA · Part II — Distributed Data**
> 
> _"It is better to have application programmers deal with performance problems due to overuse of transactions as bottlenecks arise, rather than always coding around the lack of transactions."_ — Spanner paper, Google (2012)

---

## Why transactions?

In reality, many things go wrong: hardware fails mid-write, apps crash mid-operation, network cuts connections, concurrent clients overwrite each other, clients read partially-updated data. Transactions simplify this by grouping reads and writes into a logical unit: either everything succeeds (commit) or everything is undone (abort/rollback). On abort, the app can safely retry.

Transactions are not a law of nature — they're a design choice with trade-offs. Not every application needs them. But without them, error handling and concurrency reasoning become extremely complex.

---

## The Meaning of ACID

ACID = Atomicity, Consistency, Isolation, Durability. Coined in 1983, but the implementations vary wildly between databases. "ACID compliant" is largely a marketing term.

|Letter|What it actually means|Who is responsible|
|---|---|---|
|**Atomicity**|All-or-nothing. If a fault occurs mid-transaction, all writes are undone. Better name: _abortability_. NOT about concurrency (that's Isolation).|Database|
|**Consistency**|Application invariants (e.g., credits = debits) are preserved. The _application_ defines what's valid. The C in ACID "was tossed in to make the acronym work."|Application|
|**Isolation**|Concurrent transactions don't interfere. Ideally: same result as if they ran serially. In practice, most DBs use weaker levels.|Database|
|**Durability**|Once committed, data survives crashes. Single-node: written to disk/WAL. Replicated: copied to n nodes. Perfect durability doesn't exist.|Database|

> **"Consistency" is terribly overloaded** — at least 4 different meanings in this book: (1) replica consistency (Ch. 5), (2) consistent hashing (Ch. 6), (3) linearizability (Ch. 9), (4) ACID consistency (here). Context matters.

### BASE (the "opposite"): Basically Available, Soft state, Eventual consistency. Even vaguer than ACID. Essentially means "not ACID."

---

## Single-Object vs. Multi-Object Operations

**Single-object operations:** atomicity via WAL/crash recovery, isolation via object locks. Databases also offer atomic increments (`UPDATE counters SET value = value + 1`) and compare-and-set. These are sometimes called "lightweight transactions" — misleading marketing.

**Multi-object transactions** are needed when:

- Foreign key references must stay valid across tables
- Denormalized data must stay in sync (e.g., email count + unread flag)
- Secondary indexes must be updated atomically with the primary data

### Handling errors and aborts

Retrying aborted transactions is the whole point of atomicity. But retries aren't perfect:

- **Network failure after commit** → client doesn't know it succeeded → retry = duplicate (need app-level deduplication or idempotency keys)
- **Overload caused the failure** → retrying adds more load. Use exponential backoff.
- **Permanent errors** (constraint violation) → retry is pointless
- **Side effects** (sent an email) → can't undo. Need two-phase commit (→ Ch. 9)
- **Client crashes during retry** → data lost

> Many ORMs (Rails ActiveRecord, Django) don't even retry aborted transactions — the error bubbles up and user input is lost. This defeats the purpose of transactions.

---

## Weak Isolation Levels

Serializable isolation (the ideal) has performance costs. Most databases use weaker levels that prevent _some_ race conditions but not all. This section is the heart of the chapter.

### Read Committed

The most basic useful isolation level. Two guarantees:

**1. No dirty reads:** You only see committed data. A write by an uncommitted transaction is invisible to other transactions.

- **Why it matters:** Without this, you could see a half-finished update (unread counter shows 0 but mailbox shows the new email) or read data that gets rolled back (transaction aborts but you already acted on its writes).
- **Implementation:** DB remembers both old committed value and new uncommitted value. Reads return the old value until the writing transaction commits.

**2. No dirty writes:** You only overwrite committed data. If two transactions write to the same object, the second waits until the first commits/aborts.

- **Why it matters:** Without this, mixed-up writes across objects. Example: Alice and Bob buy the same car — listing updated to Bob, but invoice sent to Alice.
- **Implementation:** Row-level write locks. Transaction holds lock until commit/abort.
- **What it does NOT prevent:** The read-modify-write lost update in the counter increment example (both reads happen after the first write commits).

**Default in:** PostgreSQL, Oracle, SQL Server, MemSQL.

### Snapshot Isolation (a.k.a. Repeatable Read)

Read committed still has problems. Alice sees $500 in one account and $400 in another (total $900) during a transfer — the $100 is "missing" because she caught the transfer mid-flight. This is **read skew** (nonrepeatable read).

Usually harmless (refresh the page), but critical for:

- **Backups:** Backup takes hours. Parts captured at different times → inconsistent snapshot → corrupted backup.
- **Analytics/integrity checks:** Query scanning the whole DB sees different points in time → nonsensical results.

**Solution:** Each transaction reads from a **consistent snapshot** of the DB at the start of the transaction. Even if data changes during the transaction, it sees the old data.

#### Implementation: Multi-Version Concurrency Control (MVCC)

- Every write creates a new version of the row, tagged with the writing transaction's ID.
- Every row has `created_by` and `deleted_by` transaction IDs.
- Updates = delete old version + create new version (both tagged).
- **Visibility rules** at read time:
    1. Ignore writes by transactions not yet committed when this transaction started
    2. Ignore writes by aborted transactions
    3. Ignore writes by transactions with higher IDs (started after us)
    4. Everything else is visible
- Garbage collection removes old versions once no transaction can see them.

**Key property:** Readers never block writers, writers never block readers. Long-running read queries work on a consistent snapshot without interfering with writes.

**Indexes:** Either point to all versions and filter at query time, or use append-only/copy-on-write B-trees (CouchDB, Datomic, LMDB) where each write creates a new root → the root IS the snapshot.

#### Naming confusion

- Oracle calls snapshot isolation "serializable" (it isn't)
- PostgreSQL and MySQL call it "repeatable read"
- IBM DB2 uses "repeatable read" to mean actual serializability
- The SQL standard doesn't define snapshot isolation (predates its invention)
- **"Nobody really knows what repeatable read means."**

---

### Preventing Lost Updates

The **lost update** problem: two transactions both read-modify-write the same value. The second write clobbers the first. Examples: incrementing a counter, editing a wiki page, updating a JSON document.

|Solution|How it works|Risk|
|---|---|---|
|**Atomic write operations**|`UPDATE counters SET value = value + 1` — DB does it atomically via exclusive lock or single thread|Not all logic can be expressed atomically|
|**Explicit locking**|`SELECT ... FOR UPDATE` locks the rows you read. Perform your update, then commit releases locks.|Easy to forget a lock somewhere|
|**Automatic lost update detection**|DB detects lost updates during snapshot isolation, aborts the offending transaction|PostgreSQL, Oracle, SQL Server do this. **MySQL/InnoDB does NOT.**|
|**Compare-and-set**|`UPDATE ... WHERE content = 'old content'` — only succeeds if value hasn't changed|Unsafe if WHERE reads from old snapshot (check your DB!)|
|**Conflict resolution (replicated)**|Siblings + merge (CRDTs, application code). LWW loses data. Commutative operations (increment) work well across replicas.|LWW (Cassandra default) is data-lossy|

---

### Write Skew and Phantoms

The subtlest race condition. Two transactions read the same data, make decisions based on it, then update _different_ objects. Neither is a dirty write or lost update, but the combined effect violates an invariant.

**The pattern (appears everywhere):**

1. **SELECT** checks a precondition (≥ 2 doctors on call, room not booked, username not taken, balance positive)
2. **Application logic** decides to proceed based on the result
3. **WRITE** (INSERT/UPDATE/DELETE) changes the data, invalidating the precondition
4. If another transaction did the same thing concurrently, both proceeded based on a stale premise

#### Worked example: Doctor on-call (write skew on existing rows)

Hospital requires ≥ 1 doctor on call. Alice and Bob are both on call. Both feel sick and click "go off call" at the same time.

|Time|Txn A (Alice)|Txn B (Bob)|
|---|---|---|
|t1|`SELECT COUNT(*) FROM doctors WHERE on_call = true` → **2**||
|t2||`SELECT COUNT(*) FROM doctors WHERE on_call = true` → **2** (same snapshot)|
|t3|2 ≥ 2, safe. `UPDATE doctors SET on_call = false WHERE name = 'Alice'`||
|t4||2 ≥ 2, safe. `UPDATE doctors SET on_call = false WHERE name = 'Bob'`|
|t5|COMMIT|COMMIT|
|Result|**0 doctors on call. Invariant violated.**||

Each txn updates a _different_ row (Alice updates her own, Bob updates his own). No write-write conflict is detected. But their combined effect breaks the constraint. Snapshot isolation misses this because the precondition ("2 doctors on call") became stale between the read and the commit, and the DB doesn't track that dependency.

**Real-world examples (same pattern):**

|Scenario|Precondition checked|Write that breaks it|Fix|
|---|---|---|---|
|Doctor on-call|≥ 2 doctors on call|Remove self from on-call|`FOR UPDATE` on matching rows, or serializable|
|Meeting room booking|No overlapping bookings|Insert new booking|Serializable (can't `FOR UPDATE` rows that don't exist yet = **phantom**)|
|Claiming a username|Username not taken|Insert new user|Unique constraint (simple fix)|
|Double-spending|Balance ≥ amount|Insert spending item|Serializable|
|Multiplayer game|Position not taken|Move piece to position|Unique constraint or serializable|

#### Phantoms

A **phantom** is when a write in one transaction changes the _result set_ of a query in another transaction. The problem: you can't lock rows that don't exist yet (`SELECT ... FOR UPDATE` on an empty result set locks nothing).

#### Worked example: Meeting room booking (phantom — no existing rows to lock)

Room 101 has no booking for noon-1pm. Alice and Bob both try to book it simultaneously.

|Time|Txn A (Alice)|Txn B (Bob)|
|---|---|---|
|t1|`SELECT COUNT(*) FROM bookings WHERE room_id = 101 AND end_time > '12:00' AND start_time < '13:00'` → **0 rows**||
|t2||Same SELECT → **0 rows** (Alice hasn't committed; under snapshot isolation, her uncommitted INSERT is invisible)|
|t3|0 = no conflict. `INSERT INTO bookings VALUES (101, '12:00', '13:00', 'Alice')`||
|t4||0 = no conflict. `INSERT INTO bookings VALUES (101, '12:00', '13:00', 'Bob')`|
|t5|COMMIT|COMMIT|
|Result|**Room 101 double-booked.**||

This is different from the doctor example in a critical way: the doctor example has _existing rows_ (Alice's and Bob's on-call records) that can be locked with `SELECT ... FOR UPDATE`. The booking example has _no rows_ at the time of the SELECT — the bookings table is empty for that slot. `SELECT ... FOR UPDATE` on an empty result set locks nothing. The phantom (Bob's INSERT changing the result of Alice's earlier SELECT) slips through.

**Key distinction: write skew on existing rows vs. phantoms**

- **Doctor example:** The SELECT returns existing rows → `FOR UPDATE` can lock them → solvable without full serializability.
- **Booking example:** The SELECT returns zero rows → nothing to lock → need predicate locks, index-range locks, or serializable isolation.

**Materializing conflicts:** Create a table of all possible lock targets (e.g., all room-timeslot combinations for 6 months). Lock rows in this table with `SELECT ... FOR UPDATE`. Ugly — leaks concurrency control into the data model. Last resort.

---

## Serializability

The only isolation level that prevents ALL race conditions. Three implementation approaches:

### Approach 1: Actual Serial Execution

Just run one transaction at a time, on a single thread. Sounds obvious — but only became practical around 2007 due to:

- **RAM got cheap** → entire active dataset fits in memory → no disk I/O waits
- **OLTP transactions are short** → a single thread can process many per second
- **Analytics queries** run separately on snapshot isolation

**Requirements:**

- **Stored procedures** — entire transaction submitted as one unit (no interactive back-and-forth between app and DB). Otherwise the single thread spends most of its time waiting for the network.
- **Transactions must be fast** — one slow transaction stalls everything.
- **Dataset fits in memory.**
- **Write throughput fits one CPU core,** OR data is partitioned so each partition has its own serial thread.

**Cross-partition transactions** require lock-step coordination across all involved partitions → orders of magnitude slower. VoltDB: ~1,000 cross-partition writes/sec (vs. millions for single-partition).

**Used by:** VoltDB/H-Store, Redis, Datomic.

### Approach 2: Two-Phase Locking (2PL)

The standard serializability algorithm for ~30 years. Much stronger than read committed locks.

**Core rule:** Readers block writers AND writers block readers. (Contrast with snapshot isolation where neither blocks the other.)

**Lock modes:**

- **Shared lock** (read): multiple transactions can hold simultaneously
- **Exclusive lock** (write): no other lock (shared or exclusive) can coexist
- **Lock upgrade:** shared → exclusive when a read transitions to a write
- **Two phases:** Phase 1 (acquire locks during execution), Phase 2 (release ALL locks at commit/abort)

**Deadlocks:** Transaction A waits for B's lock, B waits for A's lock. DB detects and aborts one. The aborted transaction must be retried. Much more frequent under 2PL than weaker isolation.

**Performance:** Bad. Significantly worse throughput and latency than weak isolation. One slow transaction holding many locks can stall the entire system. High percentile latencies are very unstable.

> **2PL ≠ 2PC.** Two-Phase Locking and Two-Phase Commit are completely different concepts (2PC is in Ch. 9).

#### Basic 2PL (row-level locks) solves write skew on existing rows

The doctor on-call example works with basic row-level 2PL because the SELECT returns existing rows:

|Time|Txn A (Alice)|Txn B (Bob)|
|---|---|---|
|t1|`SELECT * FROM doctors WHERE on_call = true` → returns Alice's row, Bob's row. Acquires **shared locks** on both rows.||
|t2||Same SELECT. Also acquires **shared locks** on Alice's row and Bob's row. (Shared locks coexist — multiple readers OK.)|
|t3|`UPDATE doctors SET on_call = false WHERE name = 'Alice'` → needs **exclusive lock** on Alice's row. But Bob holds a shared lock on it. **Alice waits.**||
|t4||`UPDATE doctors SET on_call = false WHERE name = 'Bob'` → needs **exclusive lock** on Bob's row. But Alice holds a shared lock on it. **Bob waits.**|
|t5|**DEADLOCK.** DB detects the cycle, aborts one (say Bob). Alice proceeds, commits.|Bob retries. SELECT now returns only Bob (Alice is off call). count = 1. Application refuses the request.|

This works because the locks had something to attach to — the existing doctor rows.

#### Basic 2PL (row-level locks) FAILS for phantoms

The same approach fails for the booking example because there are no rows to lock:

|Time|Txn A (Alice)|Txn B (Bob)|
|---|---|---|
|t1|`SELECT COUNT(*) FROM bookings WHERE room_id = 101 AND time overlaps noon-1pm` → **0 rows**. 2PL wants to place shared locks on returned rows, but there are no rows. **Lock attaches to nothing.**||
|t2||Same SELECT → 0 rows. Also acquires zero locks.|
|t3|`INSERT INTO bookings (101, noon-1pm, 'Alice')` → creates new row, exclusive lock on that new row.|`INSERT INTO bookings (101, noon-1pm, 'Bob')` → creates a _different_ new row, exclusive lock on _that_ new row.|
|t4|COMMIT.|COMMIT.|
|Result|**Double-booked.** Each INSERT locked its own new row. No conflict between them.||

Row-level locks can only protect rows that exist. The phantom (a new row appearing that changes a query's result) slips through. This is why 2PL needs something beyond row locks.

#### Predicate locks: locking the condition itself

A predicate lock doesn't attach to any specific row. It attaches to a **logical condition** — all objects (existing and future) that match a WHERE clause.

**Booking example with predicate locks:**

|Time|Txn A (Alice)|Txn B (Bob)|Lock state|
|---|---|---|---|
|t1|`SELECT ... WHERE room_id = 101 AND time overlaps noon-1pm` → 0 rows. DB acquires a **shared predicate lock** on the condition `(room_id=101 AND time overlaps noon-1pm)`.||Shared predicate lock (Alice)|
|t2||Same SELECT → 0 rows. Bob also acquires a shared predicate lock on the same condition. (Shared locks coexist.)|Shared predicate lock (Alice + Bob)|
|t3|`INSERT INTO bookings (101, noon-1pm, 'Alice')` → Before inserting, DB checks: **does this new row match any existing predicate lock?** The new row has room_id=101 and time=noon-1pm, which matches Bob's predicate lock. **Alice must wait** for Bob to release his predicate lock.||Alice blocked|
|t4||`INSERT INTO bookings (101, noon-1pm, 'Bob')` → Same check: new row matches Alice's predicate lock. **Bob must wait.**|**DEADLOCK**|
|t5|Bob aborted. Alice's lock upgrades to exclusive. INSERT succeeds. COMMIT.|Retries. SELECT now finds Alice's booking. Room is taken.|Resolved|

**How the predicate lock catches the phantom:** The lock guards the _answer_ to the query ("zero rows match this condition"). When Alice tries to INSERT a row that _would change that answer_, the DB evaluates: "does this new row satisfy any held predicate lock?" Yes — it matches Bob's predicate lock on `(room_id=101 AND time overlaps noon-1pm)`. So Alice is blocked, even though the row doesn't exist yet.

**Downside:** On every INSERT, UPDATE, or DELETE, the DB must evaluate the new/old row values against _every active predicate lock_ in the system. If there are thousands of concurrent transactions each holding complex WHERE predicates, this check becomes O(active_locks) per write — expensive.

#### Index-range locks: the practical approximation

Real databases don't use predicate locks because of the evaluation cost. Instead, they use **index-range locks** — a coarser but much cheaper alternative.

The idea: instead of locking the exact predicate `(room_id=101 AND time overlaps noon-1pm)`, lock the **index entry** that the query used to search. This is a superset of the original condition.

**Booking example with index-range locks (index on `room_id`):**

|Time|Txn A (Alice)|Txn B (Bob)|Lock state on index|
|---|---|---|---|
|t1|`SELECT ... WHERE room_id = 101 AND time overlaps noon-1pm` → DB uses room_id index. Attaches **shared lock to the index entry for room_id = 101**. This covers ALL time slots for room 101, not just noon-1pm.||room_id=101: shared (Alice)|
|t2||Same SELECT. Also acquires shared lock on room_id=101 index entry.|room_id=101: shared (Alice + Bob)|
|t3|INSERT (room_id=101, noon-1pm, 'Alice') → inserting a row with room_id=101 requires **updating the room_id index**. That index entry has Bob's shared lock. Alice needs exclusive access. **Alice waits.**||Alice blocked|
|t4||INSERT → same index entry, same problem. **Bob waits.** DEADLOCK. Bob aborted.|DEADLOCK → resolved|
|t5|Alice commits. Room booked.|Retries correctly.||

**Why it works:** Any INSERT for room_id=101 must modify the room_id index entry. If that entry is locked, the INSERT is blocked. The check is O(1) — just look at the index entry — not O(active_locks).

**The trade-off — false positives:** The index lock covers room_id=101 at _all times_. If Carol tries to book room 101 at 3pm-4pm (no conflict with Alice's noon booking), she's still blocked until Alice commits. A predicate lock would have allowed Carol through because her time range doesn't overlap. With a more specific composite index on `(room_id, start_time)`, the index-range lock would be tighter and produce fewer false positives.

**Fallback:** If there is no suitable index, the DB falls back to a shared lock on the **entire table**. Correct but terrible for concurrency — all writes to the table are serialized.

**Summary of the three locking levels:**

|Lock type|What it locks|Phantom prevention|Check cost|False positives|
|---|---|---|---|---|
|**Row locks** (basic 2PL)|Specific existing rows|No — can't lock rows that don't exist|O(1)|None|
|**Predicate locks**|The logical condition (all matching objects, present and future)|Yes — catches any row that would match|O(active_locks) per write|None|
|**Index-range locks**|The index entry covering the query's search range (superset)|Yes — catches anything touching that index range|O(1)|Yes — blocks writes outside the original predicate|

**Used by:** MySQL/InnoDB (serializable), SQL Server (serializable), DB2 (repeatable read).

### Approach 3: Serializable Snapshot Isolation (SSI)

The newest approach (2008). Optimistic: transactions proceed without blocking. At commit time, the DB checks for conflicts and aborts if serialization was violated.

**How it detects conflicts (two cases):**

Both cases detect the same fundamental problem — a transaction acted on stale data. The reason there are two cases is an **implementation necessity**: the DB discovers the staleness through different signals depending on when the conflicting write happened relative to the read.

#### Case 1 — Detecting stale MVCC reads

This case applies when the conflicting write existed _before_ your read, but MVCC visibility rules hid it (because the writing transaction hadn't committed yet).

**Doctor on-call example — txn 42 writes first, txn 43 reads second:**

|Time|Txn 42 (Alice)|Txn 43 (Bob)|SSI tracker|
|---|---|---|---|
|t1|`UPDATE doctors SET on_call = false WHERE name = 'Alice'` (uncommitted)||Txn 42: wrote Alice.on_call|
|t2||`SELECT COUNT(*) FROM doctors WHERE on_call = true` → **2**. MVCC hides txn 42's uncommitted write. Txn 43 sees old value (Alice.on_call = true).|**Txn 43 ignored txn 42's write (MVCC)**|
|t3||`UPDATE doctors SET on_call = false WHERE name = 'Bob'` (no blocking — SSI is optimistic)|Txn 43: wrote Bob.on_call|
|t4|**COMMIT.** Alice is off call (permanent).||Txn 42: committed. The ignored write is now permanent.|
|t5||**Tries to COMMIT.** SSI checks: "Did txn 43 ignore any MVCC writes that have since committed?" **Yes** — txn 42's write to Alice was ignored, and txn 42 has committed. Txn 43 also made writes. → **ABORT.**|Stale read + write = potential write skew → ABORT|

**How the DB knows at read time (t2):** When txn 43 reads, the MVCC engine literally encounters txn 42's uncommitted row version on disk and skips it (visibility rule: "ignore writes by uncommitted transactions"). At that exact moment, the DB _knows_ it hid something. It records: "txn 43 ignored txn 42's write." It doesn't abort yet — txn 42 might still abort (making the ignore harmless), and txn 43 might be read-only (no write skew risk). SSI just takes a note and waits until commit to decide.

#### Case 2 — Detecting writes that affect prior reads

This case applies when your read happens first and the data is genuinely current at that moment — then another transaction writes _after_ your read, invalidating what you saw.

**Doctor on-call example — both read first, then both write:**

|Time|Txn 42 (Alice)|Txn 43 (Bob)|SSI tracker|
|---|---|---|---|
|t1|`SELECT COUNT(*) FROM doctors WHERE on_call = true AND shift_id = 1234` → **2**|Same SELECT → **2**|Index[shift_id=1234].readers = {txn 42, txn 43}|
|t2|`UPDATE doctors SET on_call = false WHERE name = 'Alice'` → Before writing, SSI checks the index: "who else read shift_id=1234?" → Txn 43. **Notifies txn 43:** "your read may be stale."||**Txn 43 notified by txn 42's write**|
|t3||`UPDATE doctors SET on_call = false WHERE name = 'Bob'` → Checks index: "who read shift_id=1234?" → Txn 42. **Notifies txn 42.**|**Txn 42 notified by txn 43's write.** Both flagged.|
|t4|**Tries to COMMIT.** SSI checks: "was txn 42 notified?" Yes, by txn 43. "Has txn 43 committed?" **No** — still running. An uncommitted write is not yet final. → **Txn 42 can commit safely.**||Txn 42: COMMITTED (first committer wins)|
|t5||**Tries to COMMIT.** SSI checks: "was txn 43 notified?" Yes, by txn 42. "Has txn 42 committed?" **Yes.** Txn 42's write is permanent. Txn 43's read was invalidated by a committed write. Txn 43 also made writes. → **ABORT.**|Stale read (confirmed) + write → ABORT|

**How the DB knows at write time (t2, t3):** When txn 42 writes, it checks the index entries for other transactions that recently read the same data. This uses the same index infrastructure as 2PL's index-range locks, but non-blocking — instead of "you must wait" (2PL), SSI says "I'll just note that you might have a problem" (tripwire). The notification doesn't block txn 43. It just sets a flag.

**The "first committer wins" rule:** When two transactions mutually notify each other (t2 and t3), whichever commits first succeeds. The first committer checks: "has the other transaction committed?" No → safe. The second committer checks the same thing: "has the other committed?" Yes → my read was confirmed stale → abort.

#### Why SSI needs both cases

Both cases catch the same conceptual problem (acting on stale data), but the DB discovers the staleness through different signals:

||Case 1|Case 2|
|---|---|---|
|**Timing**|Conflicting write existed _before_ the read|Conflicting write happens _after_ the read|
|**What the DB sees at read time**|MVCC engine skips an uncommitted version → DB knows it hid something|Data is genuinely current → DB has no reason to flag anything|
|**Detection source**|**Reader side** — the MVCC engine records what it ignored|**Writer side** — the writer checks the index for past readers and notifies them|
|**At commit**|"Did any write I ignored become committed?"|"Did any writer who notified me commit?"|

If you only had Case 1, you'd miss the scenario where the conflicting write happens _after_ the read (nothing was hidden by MVCC — the write simply didn't exist yet). If you only had Case 2, you'd miss the scenario where the write already existed but was invisible due to MVCC snapshot rules.

Put differently: Case 1 is reader-detected ("I was given stale data at read time"). Case 2 is writer-detected ("I just made someone's past read stale, after the fact"). Together they cover all possible orderings of reads and writes.

**Why wait until commit to abort?**

- The stale read might not matter — if the transaction turns out to be read-only, there's no write skew risk.
- The other transaction might abort — making the ignored write (Case 1) or the notification (Case 2) irrelevant.
- Waiting reduces unnecessary aborts, preserving snapshot isolation's support for long-running read queries.

**Performance:**

- Writers don't block readers, readers don't block writers (like snapshot isolation)
- Query latency much more predictable than 2PL
- Not limited to one CPU core (unlike serial execution) — FoundationDB distributes conflict detection across machines
- Abort rate matters: long read-write transactions are more likely to conflict
- Less sensitive to slow transactions than 2PL or serial execution
- PostgreSQL uses theory to reduce unnecessary aborts — sometimes it can prove the execution was serializable even with flagged conflicts

**Used by:** PostgreSQL ≥9.1, FoundationDB.

---

## Chapter Summary

### Race conditions by isolation level

|Race condition|Description|Prevented by|
|---|---|---|
|**Dirty reads**|Reading uncommitted data|Read committed and above|
|**Dirty writes**|Overwriting uncommitted data|Almost all implementations|
|**Read skew**|Seeing DB at different points in time|Snapshot isolation and above|
|**Lost updates**|Concurrent read-modify-write clobbers one write|Snapshot isolation (some), serializable|
|**Write skew**|Decision based on stale premise, write different objects|Serializable only|
|**Phantoms**|Write changes another txn's search results|Serializable only|

### Three serializability approaches

|Approach|Philosophy|Strength|Weakness|Used by|
|---|---|---|---|---|
|**Serial execution**|Pessimistic (extreme)|Simple, no conflicts by definition|One CPU core, stored procs required, dataset must fit RAM|VoltDB, Redis, Datomic|
|**Two-phase locking**|Pessimistic|Prevents all races, well-understood|Deadlocks, terrible p99 latency, readers block writers|MySQL/InnoDB, SQL Server|
|**SSI**|Optimistic|Near-snapshot performance, scales across partitions|Abort rate under contention, relatively new|PostgreSQL 9.1+, FoundationDB|

### Key insights

- **Isolation levels are inconsistently named across databases.** Oracle's "serializable" is actually snapshot isolation. MySQL's "repeatable read" doesn't detect lost updates. "Nobody really knows what repeatable read means."
- **ACID consistency is the application's job**, not the database's. The C doesn't belong in ACID.
- **Write skew is the race condition that catches experienced developers.** It looks safe (different rows updated) but violates invariants. The fix is either explicit locking (`FOR UPDATE`) or true serializability.
- **Phantoms are write skew where the problematic rows don't exist yet.** Can't lock what doesn't exist → materializing conflicts or serializable isolation.
- **Transactions in distributed databases** (across partitions, across nodes) open a new set of challenges → Ch. 8 (faults) and Ch. 9 (consensus, distributed transactions, 2PC).

### Connections to previous chapters

|Concept|Connection|
|---|---|
|WAL / crash recovery (Ch. 3)|Implements single-object atomicity|
|Replication lag (Ch. 5)|Read-your-writes, monotonic reads are weaker versions of the consistency that transactions provide|
|Async replication + failover (Ch. 5)|Lost writes on failover = durability violation|
|Partitioning (Ch. 6)|Serial execution + partitioning = linear throughput scaling (but cross-partition txns are orders of magnitude slower)|
|Secondary indexes (Ch. 6)|Must be updated atomically with the primary data — a multi-object transaction problem|

### Forward references

|Topic|Where|
|---|---|
|Network faults and unreliable clocks|Chapter 8|
|Consensus and distributed transactions (2PC)|Chapter 9|
|Exactly-once semantics / end-to-end argument|Chapter 12|

---

_Notes from Designing Data-Intensive Applications by Martin Kleppmann, Chapter 7: Transactions._