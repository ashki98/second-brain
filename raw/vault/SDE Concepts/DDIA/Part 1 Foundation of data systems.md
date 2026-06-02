# Part 1: Foundation of data systems

This summary covers **Part I: Foundations of Data Systems** (Chapters 1–4), focusing on the core principles of reliable, scalable, and maintainable applications, data modelling, storage engine internals, and data encoding.

### **Chapter 1: Reliable, Scalable, and Maintainable Applications**

This chapter establishes the three fundamental concerns for any data system:

- **Reliability:** The system should continue to work correctly even when things go wrong (faults). Faults can be hardware-related, software bugs, or human errors.
- **Scalability:** A system's ability to cope with increased load (e.g., data volume or traffic). It is measured using **load parameters** and performance metrics like **response time percentiles** (e.g., p95, p99).
- **Maintainability:** Ensuring the system is easy for people to work on over time. This includes **operability** (ease of keeping it running), **simplicity** (removing accidental complexity), and **evolvability** (ease of making changes).

### **Chapter 2: Data Models and Query Languages**

The way data is modelled has a profound effect on how we think about the problem we are solving.

- **Relational vs. Document:** The relational model organizes data into tables and rows. The document model (e.g., JSON) is better for data that has a tree-like structure and where most data is loaded at once.
- **Normalization:** The process of removing duplication by storing a piece of information in one place and referring to it via an ID.
- **Query Languages:** **Declarative languages** (like SQL) specify the pattern of data wanted but not how to achieve it, leaving optimization to the database. **Imperative code** tells the computer exactly which steps to perform in which order.

### **Chapter 3: Storage and Retrieval**

This chapter focuses on how databases store data and find it again efficiently.

**Indexing Structures (The "Signposts")**

- **Indexes:** Additional structures derived from the primary data to speed up reads, though they typically slow down writes.
- **Secondary Indexes:** Unlike primary indexes, the keys are **not unique**.
- **Storing Values within the Index:**
    - **Heap Files:** The index contains a **pointer** to the data's location in an unordered file.
    - **Clustered Indexes:** The **actual data** is stored directly within the index (e.g., MySQL’s InnoDB primary key) to avoid an extra lookup.
    - **Covering Indexes:** A compromise that stores **some columns** in the index so that certain queries can be answered using only the index.
- **Full-text Search:** Uses specialized structures like **Levenshtein automata** (in Lucene) to handle fuzzy searches and typos.

**Visual Representation of a Secondary Index (Used Car Database)**

| Index Key (Searchable Attribute) | Index Value (Reference List) |
| --- | --- |
| **Color: Red** | [ID 191, ID 306, ID 768] |
| **Make: Honda** | [ID 191, ID 512] |

*(The database finds the "Red" key, gets the IDs, and then "hops" to the primary data to fetch full details).*

**Analytics and Column Storage**

- **OLTP vs. OLAP:** **OLTP** (Transaction Processing) involves fast, key-based queries. **OLAP** (Analytics) involves scanning huge numbers of records to calculate aggregates.
- **Column-Oriented Storage:** Stores each column in a separate file, allowing the database to only read the columns needed for a query.
- **Bitmap Encoding:** A compression technique where each distinct value in a column gets its own bitmap, with **one bit per row**.
    - **1s and 0s:** A `1` means the row contains that value; `0` means it does not.
    - **Performance:** Allows the CPU to use incredibly fast **bitwise AND/OR** operators for filtering.

### **Chapter 4: Encoding and Evolution**

As applications evolve, their data formats must change while maintaining **backward** and **forward compatibility**.

**Binary Encoding and Code Generation**

- **Code Generation:** The process of using a schema (blueprint) to automatically create source code (classes) in a specific language to encode/decode data [Conversation].
- **Type Systems:**
    - **Statically Typed (Java/C++):** Code generation is vital for **compile-time safety** and IDE support [118, Conversation].
    - **Dynamically Typed (Python/JS):** Code generation is often an obstacle; safety benefits are "lost" because types are checked at **runtime** [127, Conversation].
- **Formats:** **Protocol Buffers** and **Thrift** use field tags (numbers) to be compact but require code generation. **Avro** is more flexible, embedding the schema in the file to be self-describing.

**Modes of Dataflow**

- **REST vs. RPC:** **RPC** tries to make a network request look like a local function call (**location transparency**). **REST** is a design philosophy that builds on HTTP and makes the network explicit.
- **The "RPC Model":** Even frameworks using **JSON over HTTP** (like Rest.li) are considered RPC if they focus on remote method calls rather than REST's resource-based philosophy [410, Conversation].
