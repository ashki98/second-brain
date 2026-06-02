# DB Sharding

video summary: 

[https://www.youtube.com/watch?v=5faMjKuB9bc](https://www.youtube.com/watch?v=5faMjKuB9bc)

---

**Database Sharding – Concept Notes**

- **Sharding Definition:** Horizontal partitioning of a database to divide data across multiple servers using a *shard key* (e.g., user ID, location), improving scale, reliability, and performance.
- **Pizza Analogy:** Database as a large pizza, with each server (friend) getting a slice (shard), subset determined by shard key.
- **Horizontal vs. Vertical Partitioning:**
    - *Horizontal (Sharding):* Splits rows into different tables/databases.
    - *Vertical:* Splits columns (less common for sharding).
- **Sharding Process & Shard Key Choices:**
    - Choose an even-distribution key, like user ID or location.
    - Each server stores only its key range subset.
    - Example: Shard on location for apps like LinkedIn.
- **Key Considerations:**
    - *Consistency:* Writes/reads remain correct across distributed shards.
    - *Availability:* Maintaining uptime (sometimes less critical than consistency).
- **Drawbacks/Challenges:**
    - Joins across shards require network queries, incurring overhead.
    - Shard inelasticity—fixed slices make scaling tricky; *consistent hashing* helps.
    - Overloaded “hot” shards need splitting—can use hierarchical sharding.
- **Indexing in Shards:**
    - Create indexes on shard-local data, possibly on different attributes than shard key.
- **Replication & Master-Slave Architecture:**
    - Each shard can have master-slave setup: master for writes, slaves for reads/scalability.
    - Slaves mirror master and can be promoted if needed.
- **Application & Advice:**
    - Sharding is simple conceptually, harder to do well.
    - Prefer built-in solutions (indexes, NoSQL databases) before custom sharding.
- **Final Challenge:**
    - Weigh simpler solutions first before sharding—don’t do it unnecessarily.

---
