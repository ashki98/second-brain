

> **Source:** [Arpit Bhayani — Hash Table Internals, Part 3](https://www.youtube.com/watch?v=6_yFb7icd_c) **Series:** Hash Table Internals (Part 3 of ~9) **Prerequisite:** [Part 1 — Internal Structure](https://claude.ai/chat/internal-structure-of-a-hash-table.md), [Part 2 — Chaining](https://claude.ai/chat/conflict-resolution-chaining.md)

---

## The Key Difference from Chaining

Chaining handles collisions by using an **auxiliary data structure** (linked list / BST) attached to each slot. Open addressing takes a completely different approach: **no auxiliary structures at all**. Instead, when a collision occurs, we look for the next available empty slot _within the array itself_.

```
Chaining:                        Open Addressing:
Array of pointers → lists        Array of KV pairs directly

slot[0] → NULL                   slot[0]: empty
slot[1] → [X] → [Y] → NULL      slot[1]: X
slot[2] → [A] → NULL             slot[2]: Y  ← Y collided at 1, probed here
slot[3] → NULL                   slot[3]: A
                                 slot[4]: empty

Uses extra memory (pointers)     Everything lives in the array itself
```

This makes open addressing **more space-efficient** — no pointer overhead, no heap-allocated list nodes. Every KV pair sits directly in the contiguous array, which is great for CPU cache performance.

---

## Probing: The Core Mechanism

When a slot is occupied, we need a deterministic way to find the next slot to check. This process is called **probing**.

Formally, the probing function is:

```
j = p(k, i)

where:
  k = the key being inserted/looked up
  i = the attempt number (0, 1, 2, ...)
  j = the slot index to check (in range [0, m-1])
  m = size of the hash table
```

For attempt `i = 0`, we get the **primary slot** (the one the hash function originally points to). For subsequent attempts, the probing function deterministically generates the next slot to try.

### What Makes a Good Probing Function?

A good probing function generates a **deterministic permutation of all slots [0, m-1]** for any given key `k`. This ensures we eventually check every slot in the table before concluding it's full.

```
Example: for key k1 in a table of size 8, the probing sequence might be:

  Attempt:  0  1  2  3  4  5  6  7
  Slot:     5  7  2  1  0  6  4  3

  → tries slot 5 first, then 7, then 2, etc.
  → covers all 8 slots — no slot is skipped
```

If the probing function _doesn't_ cover all slots, you might fail to find an empty slot even when one exists.

---

## Three Common Probing Strategies

### 1. Linear Probing

The simplest strategy: if slot `h(k)` is occupied, try `h(k) + 1`, then `h(k) + 2`, etc.

```
p(k, i) = (h(k) + i) mod m

Sequence: h(k), h(k)+1, h(k)+2, h(k)+3, ...  (wrapping around)

Insert "eve" where h("eve") % 8 = 3, but slot 3 is occupied:

  [0]     [1]     [2]     [3]      [4]     [5]     [6]     [7]
               "alice"  "bob"             "carol"
                          ↑
                       h("eve")=3
                       occupied!
                          ↓ try 4
                                  "eve"
                                  ↑ slot 4 is free → placed here
```

**Pros:** Simple, cache-friendly (sequential memory access — CPU prefetcher loves this).

**Cons:** Suffers from **primary clustering**. When collisions happen, keys pile up in contiguous blocks. Once a cluster forms, any new key that hashes anywhere into that cluster extends it further. The cluster grows like a snowball.

```
Clustered collision (linear probing):

  [0]     [1]     [2]     [3]     [4]     [5]     [6]     [7]
           A       B       C       D       E
           └───────────────────────────────┘
                    cluster grows →

Any key hashing to slots 1-5 extends this cluster further.
New key hashing to slot 1 must probe through 5 occupied slots.
```

### 2. Quadratic Probing

Instead of stepping linearly (+1, +2, +3...), step quadratically:

```
p(k, i) = (h(k) + c1·i + c2·i²) mod m

Sequence: h(k), h(k)+1, h(k)+3, h(k)+6, h(k)+10, ...
                      (offsets grow quadratically)
```

**Pros:** Breaks up the primary clustering problem — collided keys spread out faster.

**Cons:** Still suffers from **secondary clustering** (keys with the same primary hash follow the same quadratic sequence). Also, doesn't guarantee visiting all slots unless `m` is chosen carefully (prime numbers help).

### 3. Double Hashing

Uses a **second hash function** to determine the step size:

```
p(k, i) = (h1(k) + i · h2(k)) mod m

Sequence: h1(k), h1(k)+h2(k), h1(k)+2·h2(k), h1(k)+3·h2(k), ...
```

**Pros:** Each key gets its own unique step size, so even keys that share the same primary slot spread out in completely different directions. Eliminates both primary and secondary clustering.

**Cons:** Requires computing two hash functions (extra CPU), and the probing pattern jumps across memory unpredictably (poor cache locality — the opposite of linear probing).

### Comparison

```
Strategy         Clustering    Cache-friendliness    Computation cost
──────────────   ───────────   ──────────────────    ────────────────
Linear           Worst         Best (sequential)     Cheapest (1 hash)
Quadratic        Better        Medium                Medium (1 hash + math)
Double hashing   Best          Worst (random jumps)  Most expensive (2 hashes)
```

The optimal choice is contextual — there's no universally best strategy. If your hash table is performance-critical, you need to benchmark with your actual data.

---

## Operations in Detail

### Insert(key, value)

```
function insert(key, value):
    for i = 0 to m-1:
        slot = p(key, i)           // probe for next slot
        if table[slot] is EMPTY or DELETED:
            table[slot] = (key, value)
            return
        if table[slot].key == key:
            table[slot].value = value   // update existing
            return
    error("hash table is full")
```

### Lookup(key)

```
function lookup(key):
    for i = 0 to m-1:
        slot = p(key, i)
        if table[slot] is EMPTY:
            return NULL              // key doesn't exist
        if table[slot].key == key AND not DELETED:
            return table[slot].value
    return NULL                      // checked entire table
```

**Critical detail:** We stop when we hit an **empty** slot (never been used), but we skip over **deleted** slots (were used, then removed). This is because a key that was inserted after the deleted key might be sitting further along the probe sequence.

### Delete(key) — Soft Delete Only

This is the most subtle part of open addressing. You **cannot** simply empty the slot, because it would break the probe chain for keys that were inserted after it.

```
Why hard delete breaks lookups:

Step 1: Insert A at slot 3, Insert B at slot 3 → collision, B probes to slot 4

  [3]     [4]     [5]
   A       B      empty

Step 2: Hard-delete A (empty slot 3)

  [3]     [4]     [5]
  empty    B      empty

Step 3: Lookup B → hash says slot 3 → slot 3 is EMPTY → "B not found" ❌
         But B is right there at slot 4!

The empty slot at 3 terminates the probe too early.
```

The solution: **soft delete** (also called tombstoning). Mark the slot as `DELETED` instead of `EMPTY`. During lookup, we skip over `DELETED` slots and keep probing. During insert, we _can_ reuse a `DELETED` slot.

```
Each slot has three possible states:

  EMPTY    → never been used (terminates probe on lookup)
  OCCUPIED → holds a live KV pair
  DELETED  → was occupied, now soft-deleted (skip during lookup, reuse on insert)
```

---

## Slot Structure

```
struct slot {
    int32    hash_key;    // Pre-computed hash
    void     *key;        // Application key
    void     *value;      // Stored value
    bool     is_empty;    // True if never used
    bool     is_deleted;  // True if soft-deleted
};
```

The `is_empty` vs `is_deleted` distinction is critical — they look similar (both "unused") but behave very differently during probing.

---

## Key Limitation

Since there's no auxiliary data structure, the **maximum number of keys = number of slots**. The load factor can never exceed 1.0 (unlike chaining where it can). This is why resizing (Part 8-9 of the series) is even more critical for open addressing.

---

## Chaining vs Open Addressing — Side by Side

```
                        Chaining                    Open Addressing
                        ────────                    ───────────────
Collision strategy      Auxiliary linked list/BST    Probe for next empty slot
Memory overhead         Pointer per node + heap      None (all in-place)
Cache performance       Poor (pointer chasing)       Good (contiguous array)*
Max load factor         Can exceed 1.0               Must stay below 1.0
Deletion                Real delete                  Soft delete (tombstone)
Worst-case lookup       O(n) or O(log n) with BST    O(n)
Implementation          Simple                       More complex (3 states)
Best for                Variable/unpredictable load   Known/bounded key count

* Linear probing specifically; double hashing loses this advantage
```

---

## Connections to What You Already Know

### Python's `dict` Uses Open Addressing

When you write `memo = {}` in your LeetCode solutions (Word Break, Course Schedule), Python uses **open addressing with a custom probing strategy** (a perturbation-based probe that mixes in higher bits of the hash). It doesn't use chaining. This means:

- Deletions (`del memo[key]`) are soft deletes internally
- The dict automatically resizes when LF reaches ~2/3
- The contiguous memory layout is part of why Python dicts are surprisingly fast

### DDIA Chapter 5 — Replication and Conflict

Interestingly, the word "conflict" in this context (two keys wanting the same slot) is structurally similar to write conflicts in multi-master replication from DDIA Chapter 5. In both cases, the core question is: when two things compete for the same resource (slot / row), what's the resolution strategy? Chaining is like "keep both" (multi-value). Open addressing's probing is like "find somewhere else to put it."

### The Tombstone Pattern

Soft deletes / tombstones appear across many systems. In LSM-trees (DDIA Ch 3), a delete is also a write (a tombstone marker) rather than an actual removal, for the same fundamental reason: removing data in-place would break the sequential structure that other operations depend on.

---

## Summary for Quick Recall

```
OPEN ADDRESSING = no auxiliary structures; collided keys go into empty slots

Probing function: j = p(k, i)  →  deterministic sequence of slots to check
  Linear:    p = (h(k) + i) mod m           → simple, cache-friendly, clusters
  Quadratic: p = (h(k) + c1·i + c2·i²) mod m → less clustering, may miss slots
  Double:    p = (h1(k) + i·h2(k)) mod m   → no clustering, cache-unfriendly

Three slot states: EMPTY | OCCUPIED | DELETED
  EMPTY     → never used (stops lookup probe)
  DELETED   → tombstone (skip on lookup, reuse on insert)
  OCCUPIED  → live KV pair

Delete = SOFT DELETE only (hard delete breaks probe chains)

Limitation: max keys = array size (LF cannot exceed 1.0)

Tradeoffs vs Chaining:
  ✅ No pointer overhead (space-efficient)
  ✅ Cache-friendly (contiguous memory, especially linear probing)
  ✅ No heap allocation per entry
  ❌ Must soft-delete (tombstones accumulate, degrade performance)
  ❌ Cannot exceed array size
  ❌ More complex state management (3 states per slot)
```