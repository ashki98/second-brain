# RPC

| **Feature** | **Lambda (RPC-style)** | **gRPC** |
| --- | --- | --- |
| **Infra** | Fully managed (serverless) | Self-managed servers |
| **Protocol** | HTTP/HTTPS | HTTP/2 |
| **Serialization** | JSON (typically) | Protocol Buffers (binary, fast) |
| **Latency** | Higher (cold starts, network overhead) | Lower (persistent connections) |
| **Streaming** | Not supported | Supported (client/server/bidirectional) |
| **Scaling** | Auto (per-invocation) | You scale it |
| **Best for** | Lightweight APIs, low-maintenance workloads | High-performance internal service comms |

### **🔹**

### **What is RPC (Remote Procedure Call)?**

- A way to **call functions on another machine** as if they were local.
- Common in microservices and distributed systems.
- Handles sending requests, executing the function remotely, and returning results.

---

### **🔹**

### **Are Lambda Functions RPC?**

- **Yes**, if you’re calling them remotely (e.g., via SDK, API Gateway).
- **No**, if they’re triggered by events (e.g., S3 uploads) — that’s event-driven.
- They can *act like RPC* in serverless setups when used as on-demand function calls.

### **✅**

### **Use Lambda when:**

- You want minimal infrastructure.
- You’re building event-driven or low-traffic APIs.
- You prefer serverless and JSON-based communication.

### **✅**

### **Use gRPC when:**

- You need fast, real-time, or streaming communication.
- You’re building high-performance microservices.
- You want strict typing and better efficiency with Protobuf.
