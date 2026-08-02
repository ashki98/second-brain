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

---

## Deep Dive: Partitions, Replication & the Controller

The video's summary above is the gist; this section is the mechanical detail underneath each piece, worked through with real code from `workifi_repos` (`core-be`, `cai`, `notifi`).

### One message → exactly one partition, never all of them

- A message is **never broadcast** to every partition. The producer's partitioner hashes the key and routes to **one** partition, deterministically — same key always lands on the same partition.
- No key → round-robin across partitions for load balancing, with no ordering guarantee.
- Ordering only exists **within** a partition. Partition 0 of `orders` and partition 0 of `events` share nothing but a label — different topics' partitions are unrelated logs.

### Broker = leader/follower per partition, not per topic

- **Leader replica**: for each partition, one broker is elected leader — all reads/writes for that partition go through it only.
- **Follower replica**: passively replicates from the leader.
- **ISR (in-sync replicas)**: followers fully caught up. `acks=all` only succeeds once every ISR member has the write.
- **Controller**: one broker elected to detect failures and reassign partition leadership — this is a consensus problem (same territory as DDIA ch. 9).
- **ZooKeeper → KRaft**: the video's "ZooKeeper + Paxos" description is the old mechanism. Kafka 3.x/4.x replaced this with **KRaft**, Kafka's own Raft-based consensus layer — no external ZooKeeper cluster needed anymore.

### Hardware reality: a broker is basically one EC2/VM

- One broker = one Kafka server process = one compute instance with its own local disk. Confirmed in `core-be/src/core/kafka/config.ts`: `brokers: [settings.kafka.broker1, settings.kafka.broker2]` — two literal host:port machine addresses.
- Disk matters more than CPU (sequential append-heavy writes; production uses local SSD/NVMe or high-IOPS EBS).
- Kafka leans on the OS page cache, not JVM heap, for reads (zero-copy `sendfile` — this is exactly the "Zero Copy Messaging" section above, at the OS level).
- Network is often the real bottleneck: `replicationFactor: 3` means 1 message in ≈ 3x that in cluster-internal replication traffic before it's durable.

### Group coordinator — a role, not a separate service

- One specific broker, chosen by hashing the `group_id` (via which broker leads the relevant `__consumer_offsets` partition).
- Tracks group membership, receives heartbeats, triggers rebalances. Different consumer groups often land on different coordinators, spreading load across the cluster.

---

## Deep Dive: Where Configuration Actually Lives

The video doesn't cover this, but it's the part that was confusing in practice: **application code never touches partitions directly.** It touches *keys* (producer side) and *group membership* (consumer side). Partition count, replication factor, and topic-level retention are all decided at admin/infra time, separately from the produce/consume code path.

### Topic creation — admin API, not producer/consumer code

```ts
// core-be/src/api/services/KafkaService.ts
await kafkaAdmin.createTopics({
  topics: [{ topic, numPartitions, replicationFactor }],
});
```

`producerConfig.allowAutoTopicCreation: false` confirms this is deliberate — a topic that doesn't exist yet won't get silently auto-created with a default 1-partition/1-replica shape from app traffic.

### Producer — key in, no partition number, ever

```ts
// core-be/src/core/kafka/producer.ts
await producer.send({ topic, messages: [{ value, key, timestamp }] });
```

```python
# cai/common/kafka_utils.py
future = await producer.send(topic, message, key=key)
```

`acks` is usually **implicit**, not set directly:

```ts
// core-be/src/core/kafka/config.ts
export const producerConfig = {
  allowAutoTopicCreation: false,
  idempotent: true,           // forces acks=all internally — Kafka correctness rule
  maxInFlightRequests: 1,     // caps in-flight requests so retries can't scramble ordering
  transactionTimeout: 30000,
};
```

```python
# cai/common/kafka_utils.py
producer = AIOKafkaProducer(
    bootstrap_servers=KAFKA_BROKERS,
    compression_type='gzip',
    enable_idempotence=True,   # same forcing effect
)
```

Idempotence only protects **within one connected producer session** — a restart gets a brand-new producer ID, no dedup across restarts. Any message in-flight and unacked at crash time is the application's problem, not Kafka's (see the `Event`/`EventStatus` tracking pattern in `core-be` — that's exactly what fills this gap).

### Consumer — topic + group_id, no partition number, ever

```ts
// core-be/src/core/kafka/consumer.ts
await consumer.subscribe({ topic, fromBeginning: false });
```

```python
# cai/common/kafka_utils.py
self.consumer = AIOKafkaConsumer(topic, bootstrap_servers=..., group_id=group_id, ...)
```

`group_id` factory pattern, shared across services:

```ts
// notifi/src/core/kafka/index.ts
const createConsumer = (groupId: string) => {
    return kafkaConnection.consumer({ groupId });
};
```

Each service (`chat-service`, `notifi`, `suggestion-engine`) runs its own `group_id`, so each gets a full independent copy of any topic they all subscribe to — independent groups never compete for partitions with each other.

### Full config map

| Setting | Lives in | Example |
|---|---|---|
| `numPartitions`, `replicationFactor` | Admin API, topic creation | `KafkaService.createKafkaTopic` |
| `retention.ms`, `cleanup.policy` | Admin API, topic config | `KafkaService.updateTopicConfig` |
| `brokers`, `clientId`, `ssl` | Shared client config | `kafkaConfig` |
| `idempotent`, `maxInFlightRequests`, `transactionTimeout`, `acks` (implicit) | Producer config | `producerConfig` |
| `groupId`, `sessionTimeout`, `heartbeatInterval`, `maxBytesPerPartition`, `maxWaitTimeInMs` | Consumer config | `consumerConfig` |

---

## Deep Dive: Consumer Groups, Rebalancing & Offset Tracking

### Rebalancing — the config knobs behind "an in-sync replica is promoted"

The video's failover description is at the partition-leader level; consumer-group rebalancing is a separate mechanism, triggered by:

| Setting | core-be (kafkajs) | cai (aiokafka) | What it does |
|---|---|---|---|
| Heartbeat | `heartbeatInterval: 3000` | `heartbeat_interval_ms: 20000` | how often the consumer pings the coordinator |
| Session timeout | `sessionTimeout: 30000` | `session_timeout_ms: 60000` | no heartbeat within this window → declared dead → rebalance |
| Max poll interval | — | `max_poll_interval_ms: 300000` | catches a hung processor even while heartbeats keep arriving — a distinct failure mode from a dead connection |

### Consumers > partitions → idle consumers, not more throughput

With 3 partitions and 4 consumers in one group, Kafka can only hand out 3 partition assignments — one consumer gets nothing, sitting as a hot standby. If any of the other 3 dies, a rebalance happens and the idle one picks up that partition instantly. **More consumers than partitions buys faster failover, not more throughput** — for more parallelism, the partition count itself has to go up.

### Offset tracking — producer and consumer are completely different here

- **Consumers track a read position, and Kafka stores it for them.** Every commit writes to the internal `__consumer_offsets` topic, keyed by `(group_id, topic, partition)` — it lives on the broker, not in the pod.
- **Producers track no position at all.** Each `send()` is independent; the only state carried by an idempotent producer is a broker-assigned producer ID + sequence number, used purely for de-duping retries within one session.

**On restart:**
- Consumer pod restarts with the same `group_id` → rejoins and resumes exactly from the last committed offset. Pod identity is irrelevant; `group_id` is what matters.
- Producer pod restarts → no stored position exists to resume from; gets a fresh producer ID; any message lost mid-flight at crash time is on the application to detect (`Event`/`EventStatus` pattern again).

### Real divergence in delivery semantics — core-be vs cai

- **core-be**: default kafkajs behavior — commit *after* processing → at-least-once. Crash mid-processing → redelivered on restart.
- **cai**: `enable_auto_commit=False`, but commits **immediately on receipt, before processing**:
  ```python
  async for msg in self.consumer:
      await self.consumer.commit()          # committed here
      self.message_queue.put_nowait(msg)     # processed later, on a separate queue
  ```
  This leans at-most-once for the actual processing step — a crash between commit and processing loses that message permanently. Deliberate trade-off: decouples "reading fast" from "processing," favoring consumer-group liveness over guaranteed redelivery (fine for streaming chat events, wrong for billing).

### DLQ pattern — what happens when at-least-once still isn't enough

Retrying a poison message forever in place would stall every message behind it in that partition (ordering is per-partition). `core-be/src/core/kafka/dlq.ts` shunts it sideways instead:

```ts
const dlqTopic = `${topic}-dlq`;
await produceMessage(dlqTopic, dlqMessage, message.key?.toString());
```

### Why offsets are tracked per-partition, not per-topic

There is no "next message in a topic" — ordering only exists within a partition, and each partition has its own independent offset counter starting at 0. Offset 5 of partition 0 and offset 5 of partition 1 are two unrelated messages. The only thing that uniquely identifies "where have I read up to" is `(group_id, topic, partition)` — a topic-level offset is meaningless.

**One-line anchor: a partition is the unit of everything** — ordering, parallelism, and offset tracking all trace back to "each partition is its own independent log."

Kafka: Distributed Messaging System (Video Summary)
