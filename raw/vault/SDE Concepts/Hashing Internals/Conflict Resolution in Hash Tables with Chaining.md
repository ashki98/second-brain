

> **Source:** [Arpit Bhayani — Hash Table Internals, Part 2](https://www.youtube.com/watch?v=9rb8oILi4lU) **Series:** Hash Table Internals (Part 2 of ~9) **Prerequisite:** [Part 1 — Internal Structure of a Hash Table](https://claude.ai/chat/internal-structure-of-a-hash-table.md)

---

## Why Do Collisions Happen?

From Part 1, recall the two-step pipeline: hash the key to INT32, then modulo down to a small array index. Since we're mapping a **large range** (~4.29 billion) to a **small range** (array size), it's inevitable that two different keys will eventually land on the same slot.

```
key "alice"  ──▶  h("alice") = 78291034  ──▶  78291034 % 8 = 2  ──┐
                                                                    ├── Same slot!
key "bob"    ──▶  h("bob")   = 55130210  ──▶  55130210 % 8 = 2  ──┘
```

Hash tables **cannot be lossy** — you can't just throw away "bob" because "alice" got there first. So we need a conflict resolution strategy. Chaining is the first and most intuitive approach.

---

## Core Idea: Array of Linked Lists

Instead of each slot holding a single KV pair, each slot holds the **head of a linked list**. All keys that hash to the same slot are chained together in that list.

```
Index   Slot contents
─────   ──────────────────────────────────────────
  0     ┌───────────┐
        │ "dave": 4 │ → NULL
        └───────────┘

  1     NULL (empty)

  2     ┌────────────┐    ┌──────────┐
        │ "alice": 1 │ ──▶│ "bob": 2 │ → NULL
        └────────────┘    └──────────┘

  3     NULL (empty)

  4     ┌────────────┐
        │ "carol": 3 │ → NULL
        └────────────┘

  5     NULL (empty)
```

The array itself is still the same fixed-size array from Part 1. The difference is that each element is now a pointer to a linked list, not a direct KV pair.

---

## Operations

### Insert(key, value)

1. Pass key through hash function → get slot index
2. Walk the linked list at that slot
3. If the key already exists → update the value
4. If the key doesn't exist → **prepend** a new node to the head of the list (O(1) insertion)

```
Insert("eve", 5) where h("eve") % 8 = 2

Before:  slot[2] → [alice:1] → [bob:2] → NULL

After:   slot[2] → [eve:5] → [alice:1] → [bob:2] → NULL
                    ↑ new node prepended at head
```

### Lookup(key)

1. Hash key → get slot index
2. Walk the linked list at that slot
3. Compare each node's key with the target key
4. Return value if found, NULL otherwise

**Important:** You can't just compare hash values — you must compare the **actual application keys**. Two different keys can produce the same hash (that's the whole point of this section), so hash equality alone isn't sufficient.

### Delete(key)

1. Hash key → get slot index
2. Walk the linked list, find the node
3. Remove it from the list (standard linked list deletion — adjust the previous node's `next` pointer)

Unlike open addressing (Part 3), deletion in chaining is a **real delete** — you actually remove the node from memory. No tombstones needed.

---

## Node Structure

Each node in the linked list stores:

```
struct node {
    int32    hash_key;   // Pre-computed hash (avoids re-hashing during lookups)
    void     *key;       // The actual application key (for equality comparison)
    void     *value;     // The stored value
    struct node *next;   // Pointer to next node in chain
};
```

Storing the **pre-computed hash** (`hash_key`) is a key optimisation. During lookup, you first compare the cheap integer hash before doing the expensive full key comparison. This is the same pattern Java's `HashMap` uses — each entry caches its hash code.

---

## Performance Analysis

### Best case: O(1)

When keys are uniformly distributed, each slot has roughly `n/m` keys (where `n` = total keys, `m` = array size). With a good load factor (< 0.5), most chains are 0 or 1 elements long.

### Worst case: O(n)

If all keys hash to the same slot (terrible hash function or adversarial input), the entire hash table degrades to a single linked list. Every lookup becomes a linear scan.

```
Worst case: everything chains to slot 2

  0     NULL
  1     NULL
  2     [a:1] → [b:2] → [c:3] → [d:4] → [e:5] → NULL    ← O(n) scan
  3     NULL
  ...
```

---

## Optimisation: Upgrading from Linked List to BST

Arpit makes an important point: the linked list is not the only data structure you can chain with. When collisions are high (long chains), the linear scan through a linked list becomes expensive.

The fix: **replace the linked list with a self-balancing binary search tree** (BST or Red-Black tree) when the chain gets long enough.

```
Chaining with linked list          Chaining with BST
(worst case lookup: O(n))          (worst case lookup: O(log n))

slot[2]:                           slot[2]:
                                           [carol]
[alice] → [bob] → [carol]                /        \
  → [dave] → [eve] → NULL           [bob]        [dave]
                                    /                 \
Linear scan: O(n)               [alice]             [eve]

                                   Balanced tree scan: O(log n)
```

This is exactly what **Java 8's HashMap** does in practice — it starts each bucket as a linked list, and when a bucket exceeds 8 entries (the `TREEIFY_THRESHOLD`), it converts that bucket to a Red-Black tree. When the bucket shrinks below 6 entries, it converts back to a linked list.

|Chain structure|Insert|Lookup|Delete|
|---|---|---|---|
|**Linked list**|O(1)*|O(n)|O(n)|
|**BST / Red-Black**|O(log n)|O(log n)|O(log n)|

_*O(1) because we prepend to head_

---

## Drawbacks of Chaining

1. **Extra memory per node:** Each node needs a `next` pointer (8 bytes on 64-bit), plus the overhead of heap allocation for each node. For small key-value pairs, the pointer overhead can exceed the data size.
    
2. **Not cache-friendly:** Linked list nodes are scattered across memory (heap-allocated). Walking a chain means random memory access — the CPU cache can't prefetch the next node because it doesn't know where it is. This is the opposite of the contiguous memory access pattern that modern CPUs are optimised for.
    
3. **Load factor can exceed 1.0:** Unlike open addressing where you're limited by the array size, chaining can hold arbitrarily many keys per slot. This means the table works even when heavily overloaded, but performance degrades gradually.
    

---

## Connections to What You Already Know

### DDIA Chapter 3 — Storage Engines

The LSM-tree's memtable is often implemented as a Red-Black tree or skip list — the same self-balancing structures Arpit mentions as upgrades to linked-list chaining. The underlying problem is identical: maintain sorted-ish access while handling arbitrary insertions.

### LeetCode — Linked List Problems

The chaining mechanism is literally a linked list per slot. The operations (prepend, traverse, delete node) are exactly the patterns from problems like Reverse Nodes in K-Group that you worked through. The key difference is that in a hash table, you don't care about the _order_ within the chain — you just need to find or not find a specific key.

### Course Schedule (DFS Cycle Detection)

Your adjacency list representation `graph = defaultdict(list)` is effectively a hash table with chaining — each key (node ID) maps to a list of neighbours. Under the hood, Python's `dict` is implemented with open addressing (Part 3), but the _conceptual model_ is the same: key → chain of associated values.

---

## Summary for Quick Recall

```
CHAINING = each slot holds a linked list of collided keys

Insert:    hash → slot → prepend to list head             O(1)
Lookup:    hash → slot → walk list, compare keys           O(n) worst, O(1) avg
Delete:    hash → slot → find node → remove from list      O(n) worst, O(1) avg

Optimisation: swap linked list → BST/Red-Black when chain gets long
              → worst case improves from O(n) to O(log n)
              → Java 8 HashMap does this at threshold of 8 entries

Tradeoffs:
  ✅ Simple to implement
  ✅ No limit on number of keys (LF can exceed 1.0)
  ✅ Real deletes (no tombstones)
  ❌ Extra memory (next pointers + heap allocation overhead)
  ❌ Poor cache locality (scattered memory access)
```