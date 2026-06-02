# DB Deadlocks

YT video: 

[https://www.youtube.com/watch?v=QB-1tPVxNek](https://www.youtube.com/watch?v=QB-1tPVxNek)

Here’s a **comprehensive, point-wise summary** to refresh your memory on the concepts covered in the video “Understanding and Visualizing Database Deadlocks: A Practical SQL Guide.” This summary focuses on all major concepts and practical demonstrations discussed, following your preference for ease of access and note-taking:

---

**Database Deadlocks: Concept and Visualization**

- **Definition of Deadlock**
    - A *deadlock* in a SQL database occurs when two (or more) transactions are each waiting for resources (such as table or row locks) held by the other, creating a circular wait that cannot be resolved automatically without intervention.
- **Typical Deadlock Scenario**
    - Transaction A locks Resource 1 and waits for Resource 2.
    - Transaction B locks Resource 2 and waits for Resource 1.
    - Neither can proceed, leading to a deadlock.
    - Most modern databases can detect this cycle and abort one transaction to break the deadlock.
- **Demonstrating Deadlock Practically (SQL Example)**
    - Used *Postgres* and two terminals for hands-on demonstration.
    - Created a table: `bank_balance` with sample rows (e.g., accounts Anubhav and Arjun).
    - Started two transactions:
        - Transaction 1 updates balance for ID=1 (locks row) and then tries to update ID=2.
        - Transaction 2 updates balance for ID=2 (locks row) and then tries to update ID=1.
    - When both try to lock each other’s resources, the database detects a deadlock, aborts one transaction, and allows the other to continue.

**Key Concepts Explained**

- **How Deadlocks Are Detected**
    - The database monitors locking patterns; if a circular wait is observed, it considers it a deadlock.
    - The transaction that completes the cycle is typically the one that is failed/killed.
- **Impact of Deadlocks**
    - Causes failed transactions—requires the client or application to re-try operations.
    - Adds overhead, as the database must detect cycles and roll back operations.

**Strategies for Avoiding Deadlocks**

- **1. Access Resources in the Same Order**
    - Always update or lock rows/tables in a *consistent*, predetermined order in all transactions.
    - This removes circular waits and prevents deadlocks from forming.
    - Demo: Both transactions first lock ID=1 and then ID=2.
- **2. Acquire All Needed Locks Upfront (Pessimistic Locking)**
    - Use SQL syntax (e.g., `SELECT ... FOR UPDATE`) to acquire locks on all target rows at the start before performing any updates.
    - Ensures that all resources needed are locked before proceeding, preventing other transactions from causing cyclical waits.
    - Demo: Both transactions issue `SELECT ... FOR UPDATE` on the desired rows before update/commit.

**Visualizing and Practicing**

- The video encouraged using two terminals to *visualize deadlock scenarios* in real time for deeper understanding.
- This helps to reinforce how transactions interact, how waits and locks are managed, and how deadlocks manifest in actual systems.

**Summary Points (End of Video)**

- Deadlocks are a critical concept to understand for safe database management.
- Most common causes: circular locking and non-deterministic resource access patterns.
- Two main solutions:
    - Same order resource access
    - Pessimistic locking (acquire all locks before updates)
- Hands-on visualization provides intuition on how deadlocks occur and how to avoid them.

---

**Highlighted Concepts Covered in Our Conversation/Doubts Addressed**

- Focused on **demo-driven deadlock scenarios:** explained using two terminals and SQL statements.
- Emphasized **detection and avoidance mechanisms** to refresh your memory for production-facing systems.
- Covered the **exact SQL techniques** (ordering, `FOR UPDATE`) for practical deadlock prevention.
- Discussed the **impact on backend transactions**, retry logic, and database workload implications.

---

**Use this note** whenever you want a quick refresh on:

- What a deadlock is
- How it occurs and is detected
- Practical ways to diagnose and prevent it in SQL databases (especially with Postgres)
- Hands-on steps to visualize and troubleshoot deadlocks in real systems.

If you want a deeper dive (ACID, isolation levels, advanced locking) for future notes, those weren’t fully covered in this video but could be expanded in follow-up material. Let me know if you want more on those topics!

1. [https://www.youtube.com/watch?v=QB-1tPVxNek](https://www.youtube.com/watch?v=QB-1tPVxNek)
