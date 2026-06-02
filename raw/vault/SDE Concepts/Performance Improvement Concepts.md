# Performance Improvement Concepts

## Connection Pooling

💡 **Problem Without Pooling:**

•	Every time your app queries the database, it creates a **new connection**.

•	Opening and closing a connection is **slow** and **resource-intensive**.

•	If too many requests come in, the **database gets overloaded**, leading to failures.

💡 **Solution With Pooling:**

•	Instead of creating a new connection every time, a **pool of connections** is maintained.

•	When a request comes in, it **reuses an available connection** from the pool.

•	If all connections are busy, the request **waits** (or gets rejected if the pool is full). I

## DB Indexing

**2️⃣ How Does an Index Work?**

An index is like an **ordered list** that helps the database find data quickly.

🔹 Think of a library:

•	**Without an index:** You check every book to find “Python Programming.”

•	**With an index:** You check the catalog, find “Python Programming” on **Shelf B5**, and go straight there!

✅ **Use Indexes When:**

•	Querying frequently (WHERE, ORDER BY, JOIN).

•	Searching by **unique** values (email, user_id).

•	Sorting or grouping (ORDER BY, GROUP BY).

❌ **Don’t Index Everything:**

•	Low-variation columns (gender, status → “M/F”, “Active/Inactive”).

•	Frequently updated columns (last_login → Slows down updates).

•	Small tables (indexing doesn’t help much).

## Profiling

**2️⃣ Types of Profiling**

🔹 **CPU Profiling** – Measures which functions consume the most CPU time.

🔹 **Memory Profiling** – Tracks memory usage and detects memory leaks.

🔹 **Database Profiling** – Identifies slow queries and inefficient indexes.

🔹 **I/O Profiling** – Checks file, network, and disk access bottlenecks.
