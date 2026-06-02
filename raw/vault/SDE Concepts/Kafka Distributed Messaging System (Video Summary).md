# Kafka: Distributed Messaging System (Video Summary)

Kafka: Distributed Messaging System (Video Summary)

## Overview

Apache Kafka is a distributed event streaming system from LinkedIn (2011) that is widely used for:

- Message queuing between publishers and subscribers
- Event streaming and processing
- Replicating data stores through event logs

**Video Source**: Apache Kafka: a Distributed Messaging System for Log Processing by Gaurav Sen
**Duration**: 15:32 minutes
**Published**: Dec 07, 2024

---

## Core Components

### 1. **Producers**

- Applications that generate and send messages to Kafka
- Use Kafka clients to publish messages to specific topics
- Messages are distributed across partitions for scalability

### 2. **Brokers (Kafka Servers)**

- Store messages temporarily (retention period: 2+ weeks configurable)
- Handle thousands of partitions per physical server
- Provide isolation between producers and consumers
- Simplify architecture by extracting common persistence/retry logic

### 3. **Consumers**

- Pull messages from Kafka brokers (pull architecture, not push)
- Track consumption progress using message offsets (index of last pulled message)
- Consumer responsibility for pulling reduces broker complexity

## Key Concepts

### **Topics & Partitions**

- Messages are organized into **topics**
- Topics are split into multiple **partitions** for scalability
- **Ordering guarantee**: Messages within a single partition are ordered
- **No ordering guarantee**: Messages across different partitions may be processed out of order

### **Message Flow**

1. Producer writes message to a partition
2. Broker stores message with offset
3. Consumer pulls messages using offset tracking

---

## Scalability & Reliability

### **Horizontal Scaling**

- Producers, brokers, and consumers all scale independently
- LinkedIn processes 7 trillion messages per day
- Even 0.1% failure rate = 7 billion failures, requiring automated recovery

### **Replication Strategy**

- Each partition has multiple replicas across different servers

---

- **Primary replica**: Handles all write operations
- **Read replicas**: Serve read operations and stay in sync by pulling from primary
- Prevents data inconsistency across replicas

### **Failover & Leader Election**

- Uses **Apache ZooKeeper** with Paxos algorithm
- When primary fails, an in-sync replica is promoted to primary
- Only replicas that received messages in last 10s can participate in election
- **High watermark**: Consumers only see messages replicated across all replicas

---

## Performance Optimizations

### **1. Message Batching**

- Messages batched together (e.g., max 50 KB) before transmission
- Increases throughput significantly
- Applied on both producer → broker and broker → consumer paths
- Brokers only manage offsets; rate control handled by clients

### **2. Zero Copy Messaging**

- Avoids copying messages through application cache
- Direct transfer from filesystem to socket
- **Benefits**:
    - Nearly 2x faster message transmission
    - Lower memory usage
    - Avoids Java garbage collection issues (nepotism problem with young/old generations)
- Fewer I/O calls

---

## Delivery Guarantees

### **At Least Once Delivery** (Default)

- Consumer processes message, then sends acknowledgment
- If processing fails, Kafka retries the same message
- Example: Email verification - retries ensure delivery
- Offset stored in Kafka broker or ZooKeeper for fault tolerance

### **At Most Once Delivery**

- Consumer acknowledges immediately upon receiving message
- Processing failure doesn't trigger retry
- Use case: Low-priority notifications where occasional loss is acceptable

### **Exactly Once Delivery** (Complex)

Two approaches:

1. **Distributed transactions**: Two-phase commit across partition replicas (expensive)
2. **Consumer groups**: More efficient approach

---

## Consumer Groups

### **Purpose**

- Ensure exactly-once delivery
- Prevent duplicate processing of viral/popular messages

### **How It Works**

- Each consumer in a group is assigned exclusive partitions
- One partition → one consumer (within a group)
- Consumers never "step on each other's toes"
- Same topic can have multiple partitions across different brokers
- Consumers process partitions sequentially (finish one message before pulling next)

### **Example**

- Famous influencer posts message → goes to Broker 1, Partition 1
- Consumer 1 assigned to this partition exclusively
- Consumer 2 assigned to different partition
- Prevents duplicate distribution to millions of followers

---

## Use Cases

### **1. Publisher-Subscriber Pattern**

- Example: Instagram/LinkedIn celebrity posts
- Message from Brad Pitt → broadcast to millions of followers
- Kafka handles scalability and reliability

### **2. Event Sourcing**

- Record sequence of events instead of copying databases
- Example: Facebook profile creation
    1. Add profile picture
    2. Add first name
    3. Add address
    4. Add email
- Replay event log in new data store to recreate state
- Guarantees consistency between original and replicated stores

---

## Architecture Benefits

1. **Async Processing**: Producers write quickly and continue with app logic
2. **Isolation of Concerns**: Common logic extracted to Kafka
3. **Reduced Duplication**: Persistence/retry logic not repeated across apps
4. **Engineering Efficiency**: Saves time and company money
5. **Battle-Tested**: Open source, used by many large organizations

---

## Technical Notes

- **Language**: Java
- **Garbage Collection Issue**: Nepotism problem with linked lists (young/old generation promotion)
- **Bandwidth Optimization**: Batching critical for 7 trillion daily messages
- **Research Paper**: Published by Jay Kreps (LinkedIn Principal Engineer) in 2011
- **Industry Impact**: Concepts became standard across the industry

---

## Resources

- Video: [https://www.youtube.com/watch?v=hNDjd9I_VGA](https://www.youtube.com/watch?v=hNDjd9I_VGA)
- Research Paper: [https://notes.stephenholiday.com/Kafka.pdf](https://notes.stephenholiday.com/Kafka.pdf)
- Garbage Collection Details: [https://www.youtube.com/watch?v=ZhbIReLe-r8](https://www.youtube.com/watch?v=ZhbIReLe-r8)

Kafka: Distributed Messaging System (Video Summary)
