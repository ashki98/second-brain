# TCP Echo Server Implementation

This summary covers the internal workings of a simple TCP Echo server as a foundational step toward understanding Redis, based on the video **"Writing a Simple TCP Echo Server - Step 0 to Build Your Own Redis"** and the preceding discussion on Redis's single-threaded speed.

### **I. Project Overview & Setup**

The goal is to build a simple TCP Echo server from scratch in Golang with **no external dependencies**.

- **Entry Point**: The execution starts in the `main.go` file within the `main()` function.
- **Command Line Flags**:
    - **Port**: Defaulted to `7379` (Redis uses `6379`).
    - **Host**: Defaulted to `0.0.0.0` to accept connections from any IP.
- **Initialization**: The server prints "Rolling the dice" and starts a synchronous TCP server on the specified port.

### **II. Core Server Logic (The Code Flow)**

The server utilizes a **nested loop structure** to handle connections and commands, which directly illustrates the "blocking" nature of a naive single-threaded server.

### **Visual Representation of the Logic Flow**

Plaintext

# 

`[Main Process Starts]
  |
  V
[net.Listen(host:port)] (Server starts listening)
  |
  V
[Outer Infinite Loop] <------------------------------------|
  |                                                        |
  |-- [listener.Accept()] (BLOCKING CALL)                  |
  |   (Waits here until a new client connects)             |
  |                                                        |
  |-- [New Client Connected]                               |
  |   (Increments concurrent client count)                 |
  |                                                        |
  |-- [Inner Infinite Loop] <-----------------------|      |
        |                                           |      |
        |-- [readCommand()] (BLOCKING CALL)         |      |
        |   (Waits for data from THIS client)       |      |
        |                                           |      |
        |-- [respond()]                             |      |
        |   (Echoes message back to client)         |      |
        |                                           |      |
        |-- [Repeat Inner Loop] --------------------|      |
              (Loop breaks only if client disconnects)     |
                                                           |
  |-- [Client Disconnected] -------------------------------|
      (Decrements count, returns to Outer Loop to Accept next)`

### **III. Key Functions and System Calls**

- **`net.Listen`**: Initializes the server to listen for incoming TCP traffic.
- **`listener.Accept()`**: A **blocking system call**. The code execution halts here until a client initiates a connection.
- **`readCommand()`**: Uses the `read` system call to listen for messages from the connected client. This is also a **blocking call**; the server will wait indefinitely for the client to send data.
- **`respond()`**: Takes the input command and writes it back to the socket, completing the "Echo".

### **IV. Identifying the "Single-Threaded" Limitation**

The video demonstrates a critical bottleneck that addresses your previous doubts about performance and concurrency:

- **The Problem**: Because the server is single-threaded and uses nested loops, it can **only handle one connection at a time**.
- **Observed Behavior**:
    - **Client 1 connects**: Works perfectly; commands are echoed.
    - **Client 2 attempts to connect**: The server does **not** log the connection or respond. It is currently "stuck" in the **inner loop** waiting for Client 1 to send a command.
    - **Resolution**: Client 2 is only accepted by the server *after* Client 1 disconnects (e.g., via `Ctrl+C`), allowing the code to exit the inner loop and return to the `Accept()` call in the outer loop.

### **V. Redis Interaction & RESP Protocol**

The server is tested using a standard Redis CLI to see how a real client communicates.

- **Protocol Discovery**: When `redis-cli` connects, it sends commands formatted in **RESP (Redis Serialization Protocol)**.
- **Example Payload**: A simple `PUT K V` command appears on the server as a series of formatted strings: `3`, `\r\n`, `$3`, `PUT`, `$1`, `K`, `$1`, `V`.
- **Significance**: This highlights that building a Redis clone requires not just a server, but a parser for this specific serialization format.

---

###
