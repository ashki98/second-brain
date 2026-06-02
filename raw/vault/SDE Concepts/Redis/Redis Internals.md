# Redis Internals

https://www.youtube.com/watch?v=h30k7YixrMo

Based on the video provided, here is a comprehensive summary of the concepts regarding why Redis is single-threaded yet remains exceptionally fast and capable of handling multiple connections.

### **What Makes Redis Special?**

- **In-Memory Data Store**: Redis is an open-source, in-memory data structure store used as a database, cache, message broker, and streaming engine.
- **Rich Data Structures**: It provides built-in structures like hashes, lists, sets, sorted sets, bit maps, geospatial indexes, and streams.
- **Atomicity**: Every operation in Redis is atomic. When a command is executing, no other command can interrupt or be scheduled until that execution is complete.
- **Simplicity and Flexibility**: Its popularity stems from its straightforward nature and its ability to handle a plethora of use cases, from real-time analytics to gaming leaderboards.

### **Key Features for Performance and Reliability**

- **Configurable Persistence**: While data is stored in memory, Redis can be configured to periodically dump data to disk or use **Append Only Files (AOF)** to reconstruct data after a crash.
- **Transactions and Pub/Sub**: Redis supports transactions (where no other operations occur during execution) and a push-based Pub/Sub model for real-time messaging.
- **TTL and Eviction**: Keys can have a **Time To Live (TTL)** for automatic expiration, and Redis uses configurable eviction strategies to handle full memory without crashing.

### **Concurrency Models: Multi-threading vs. IO Multiplexing**

The video explains how Redis handles multiple connections through different programming models:

### **1. Multi-threading (The Traditional Approach)**

- **Concept**: A new thread is created for every client connection.
- **The Problem (Data Correctness)**: Multiple threads trying to increment the same value simultaneously can lead to unpredictable results (race conditions).
- **The Solution (Locking)**: To ensure correctness, developers use **mutexes** or **semaphores** (pessimistic locking), making threads wait to acquire a lock.
- **The Downside**: This adds complexity and can slow down the system because threads that are ready to work are often unnecessarily blocked waiting for locks.

### **2. IO Multiplexing (The Redis Approach)**

- **Apparent Concurrency**: Redis achieves "apparent" concurrency rather than "true" concurrency. It uses a single thread to manage multiple connections.
- **Non-blocking IO**: In traditional systems, a read system call is **blocking**, meaning the CPU waits and does nothing until data arrives over the network.
- **Monitoring Sockets**: Instead of waiting, Redis uses IO monitoring calls to check which sockets actually have data ready to be read.
- **The Event Loop**: Redis runs an event loop in a single thread that:
    1. Accepts new TCP connections.
    2. Identifies sockets with available data.
    3. Reads and executes the command immediately.
    4. Repeats the process for the next available socket.

## More on IO Multiplexing:

**IO Multiplexing** is a technique that allows a single process or thread to monitor multiple network connections (sockets) simultaneously to see if any of them have data ready to be processed.

Instead of getting "stuck" waiting for one specific client to send a message, the system waits for *any* client to be ready, then handles that request immediately.

---

### 1. The Problem: Blocking IO

To understand multiplexing, you first have to understand why traditional "blocking" IO is a bottleneck for single-threaded systems:

- 
    
    **Sequential Bottleneck**: In a naive single-threaded server, if you call `read()` on a socket, the execution halts (blocks) until that specific client sends data.
    
- 
    
    **Idle Resources**: If Client A connects but doesn't send anything for 10 seconds, the server remains stuck waiting, even if Client B is ready and waiting to send a command.
    
- 
    
    **Inefficiency**: This forces developers to use multi-threading, where every connection gets its own thread. However, this adds complexity like **mutexes** and **semaphores** to prevent data corruption.
    

### 2. The Solution: Apparent Concurrency

IO Multiplexing provides **apparent concurrency**. While the CPU isn't doing two things at the exact same microsecond (true concurrency), it switches between "ready" tasks so fast that it appears concurrent to the user.

- 
    
    **Monitoring, Not Waiting**: Instead of firing a `read` request blindly, the server uses system calls to monitor a collection of sockets.
    
- **The Notification System**: Think of it like a waiter in a restaurant. Instead of standing at one table waiting for a customer to decide what to eat (Blocking IO), the waiter watches the whole room and only goes to a table when a customer raises their hand (Multiplexing).
- 
    
    **Triggering Action**: The system call notifies the server: *"Hey, Sockets 1, 4, and 7 have data ready"*. The server then reads and executes those specific commands one by one.
    

### 3. How the Event Loop Uses Multiplexing

In Redis, this concept is implemented via an **Event Loop**. Everything happens in one single thread through these steps:

1. 
    
    **Accept**: The thread accepts a large number of TCP connections.
    
2. 
    
    **Monitor**: It asks the operating system which of these many sockets have data.
    
3. 
    
    **Execute**: It loops through the "ready" sockets, reads the command, and performs the in-memory operation (like `INCR` or `LPUSH`).
    
4. 
    
    **Repeat**: Once the "ready" tasks are done, it goes back to monitoring.
    

### 4. Why This Makes Redis Special

- 
    
    **No Locking Overhead**: Because it is single-threaded, Redis doesn't need to waste time on mutexes or context switching between threads.
    
- 
    
    **Exploiting Network Latency**: Network IO is slow, but memory operations are lightning-fast. Redis uses the time it *would* have spent waiting for the slow network to process thousands of other ready commands from the memory.
    

Would you like to see how this "monitoring" is actually coded using specific system calls like `select` or `epoll`?

### **Why Is Single-Threaded Redis Fast?**

- **Exploiting Slow Network IO**: Network IO is much slower than CPU operations. Redis uses the time it would have spent waiting for data to process other ready commands.
- **Fast Memory Operations**: Because Redis is in-memory, operations like adding to a list or incrementing a value are extremely fast.
- **No Overhead**: By remaining single-threaded, Redis avoids the overhead of context switching between threads and the complexities of mutexes and semaphores.
- **High Throughput**: It can support a massive number of concurrent TCP connections because it evaluates them one by one at lightning speed once the data is actually received.
