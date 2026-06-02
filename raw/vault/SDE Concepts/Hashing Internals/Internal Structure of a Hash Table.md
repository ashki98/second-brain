
# Internal Structure of a Hash Table

> **Source:** [Arpit Bhayani — Hash Table Internals, Part 1](https://www.youtube.com/watch?v=jjW8w8ED3Ns) **Series:** Hash Table Internals (Part 1 of ~9)

---

## The Big Picture

Hash tables are the backbone of an enormous number of things you interact with daily: Python `dict`, JavaScript objects, Java `HashMap`, symbol tables in compilers, and even the way OOP languages internally power class member lookups. The core promise is **O(1) key-based insertion, update, and lookup** while remaining **space-efficient**.

The fundamental insight is that hash tables are built on top of **plain arrays** — the challenge is making that work without wasting enormous amounts of memory.

---

## The Two-Step Pipeline

The entire design of a hash table boils down to solving two problems in sequence:

```
┌─────────────────┐         ┌──────────────────┐         ┌──────────────────┐
│ Application Key  │────────▶│  Hash Function    │────────▶│  Index Mapping    │
│   "user_42"      │         │  h(key) → INT32   │         │  hash % array_len │
└─────────────────┘         └──────────────────┘         └────────┬─────────┘
                                                                  │
                              Step 1: Wide range                  │ Step 2: Small range
                              (0 to ~4 billion)                   ▼
                                                     ┌───┬───┬───┬───┬───┬───┐
                                                     │ 0 │ 1 │ 2 │ 3 │ 4 │ 5 │
                                                     └───┴───┴───┴─▲─┴───┴───┘
                                                                   │
                                                              KV stored here
```

**Step 1 — Application key → Hash key (wide range):** The hash function takes _any_ object as input (string, integer, composite object) and maps it to a large integer space, typically INT32 (0 to ~4.29 billion). This is the same idea behind Python's `__hash__()` or Java's `hashCode()`.

**Step 2 — Hash key → Array index (small range):** We then take this wide-ranged integer and compress it down to fit within our actual array size, typically via modulo: `index = hash_key % array_length`. This is what makes the data structure space-efficient.

---

## Why Two Steps? The Naive Approach Fails

### The Naive Idea

Create an array of size INT32 (~4.29 billion slots). Every hash key maps directly to an index.

```
Naive approach                              Practical approach
─────────────────────────                   ─────────────────────────
┌───┬───┬───┬───┬───┬───┬ ─ ─ ┐            ┌──────┬──────┬──────┬──────┐
│KV │   │   │KV │   │   │ ... │            │ KV   │      │ KV   │      │
└───┴───┴───┴───┴───┴───┴ ─ ─ ┘            └──────┴──────┴──────┴──────┘
  ↑                                          4 keys → ~8 slots
  └── ~4.29 billion slots
                                            ✅ Array proportional to # of keys
❌ ~16 GB RAM (4 × 2³² bytes)              ✅ Mapping: hash_key % 8
❌ Mostly empty                             ✅ A few bytes of memory
```

**Why 16 GB?** Each slot needs at least 4 bytes (to hold a pointer/value), and `4 bytes × 2³² slots ≈ 16 GB`. Most of this is wasted since only a handful of slots will ever be occupied.

### The Fix

Keep the array **proportional to the number of keys inserted**. If you have 4 keys, the array might be ~8 slots. The modulo operation (`hash_key % 8`) compresses the INT32 range down to fit.

This is the core tradeoff that makes hash tables work: we give up the "zero collision" guarantee of a massive array in exchange for practical memory usage.

---

## Dynamic Resizing — Growing the Array

The small array can't hold unlimited keys. As more keys are inserted, we eventually need more space. This is handled through **dynamic resizing**, typically doubling the array when it gets "full enough."

### Load Factor

The concept of "full enough" is quantified by the **load factor**:

```
load_factor = number_of_keys / total_slots
```

|Trigger|Threshold (typical)|Action|
|---|---|---|
|**Grow**|LF ≥ 0.5|Double the array|
|**Shrink**|LF ≤ 0.125 (1/8)|Halve the array|

The asymmetry between grow (0.5) and shrink (0.125) is deliberate — it prevents **thrashing** (growing then immediately shrinking after one delete).

### The Resize Process

```
BEFORE resize (LF = 2/4 = 0.5 → threshold hit!)
┌──────┬──────┬──────┬──────┐
│"a":1 │      │"b":2 │      │
└──────┴──────┴──────┴──────┘
  [0]    [1]    [2]    [3]

         ──── 2× resize ────▶

AFTER resize (LF = 2/8 = 0.25 → healthy)
┌──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┐
│      │"a":1 │"b":2 │      │      │      │      │      │
└──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┘
  [0]    [1]    [2]    [3]    [4]    [5]    [6]    [7]

⚠️  Keys are REHASHED into the new array (new modulo = hash % 8)
    so they may land in different slots than before.
```

### Why Doubling? (Not +1)

If you grew the array by +1 every time you inserted:

- Insert 1 → copy 0 keys, Insert 2 → copy 1 key, ... Insert n → copy n-1 keys
- Total copies: `0 + 1 + 2 + ... + (n-1) = O(n²)`

If you double every time:

- Resize happens at n=1, 2, 4, 8, 16...
- Total copies: `1 + 2 + 4 + 8 + ... + n = O(n)`
- **Amortised cost per insertion: O(1)**

This is the same amortised analysis pattern that makes Python's `list.append()` O(1) amortised — the expensive operation (copying the entire array) happens infrequently enough that its cost is spread across all the cheap operations.

---

## What This Video Doesn't Cover (But the Series Does)

This video is Part 1 — the foundational structure. The rest of the series builds on top:

|Part|Topic|Key Idea|
|---|---|---|
|**2**|Collision resolution — Chaining|Array of linked lists; can upgrade to BST/Red-Black tree for O(log n) worst case|
|**3**|Collision resolution — Open Addressing|Probing (linear, quadratic, double hashing) to find next empty slot|
|**4-6**|Probing strategies|Linear vs quadratic vs double hashing tradeoffs|
|**7**|Performance factors|Load factor impact, cache-friendliness, probing strategy choice|
|**8**|Why always doubled?|Amortised O(1) analysis (covered briefly above)|
|**9**|Implementing resize|Two counters (key count vs used count), soft deletes in open addressing|

---

## Connections to Other Concepts

### DDIA Chapter 3 (Storage & Retrieval)

DDIA discusses **hash indexes** (like Bitcask) where in-memory hash maps point to byte offsets on disk. The internal structure described here is exactly what powers those in-memory hash maps. The "compaction and merging" from DDIA is the on-disk analogue of the resize operation — both exist to reclaim space and maintain performance.

### LeetCode / Algorithms

The `dict`/`set` operations you rely on in problems like Word Break (memoisation) and Course Schedule (adjacency lists) are backed by exactly this structure. When you do `memo[index] = result`, under the hood Python hashes `index` to INT range, takes modulo against the internal array size, and places the entry. This is why dict lookups are O(1) average.

### Networking (from your deep dive)

DNS resolvers often use hash tables internally to cache domain → IP mappings. The same two-step pipeline applies: hash the domain string to INT32, modulo into the cache array.

---

## Summary for Quick Recall

```
1. Hash function:    any key  ──▶  wide-range INT32 integer
2. Modulo mapping:   INT32    ──▶  small array index (hash % array_len)
3. The array:        where KV pairs actually live (plain contiguous memory)
4. Collisions:       inevitable (different keys → same slot) — handled by
                     chaining (linked list) or open addressing (probing)
5. Dynamic resize:   2× growth (LF ≥ 0.5), ½× shrink (LF ≤ 0.125)
                     keeps load factor healthy → performance stays near O(1)
```

**The fundamental trade-off:** We sacrifice perfect O(1) (which the naive 16GB array would give) for space efficiency, accepting near-O(1) amortised time instead. That's the engineering bargain at the heart of every hash table.