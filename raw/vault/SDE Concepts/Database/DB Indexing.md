# DB Indexing

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
