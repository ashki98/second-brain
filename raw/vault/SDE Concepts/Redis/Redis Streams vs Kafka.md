# Redis Streams vs Kafka

Redis Streams borrows Kafka's vocabulary almost directly, but the underlying architecture is different enough that carrying Kafka assumptions over 1:1 actively misleads. Grounded in real code from `new-concerto` (`concerto-ripples`, `concerto-pulse`, `caasi-citadel`).

---

## The core mapping: a Redis Stream = one Kafka partition, not a topic

A Kafka topic is *N* independent logs (partitions) that Kafka automatically splits and routes into. A Redis Stream is **one** ordered, append-only log — full stop. No built-in partitioning inside a single stream.

If partition-like parallelism is needed, **you** create multiple streams yourself and decide client-side which one a message goes to (e.g. `orders:0`, `orders:1`) — Redis won't hash a key into a partition the way Kafka's producer partitioner does. None of the three repos below do this key-based stream-sharding — they get parallelism a different way (see the patterns section).

---

## Primitives, mapped

| Redis Streams | Kafka equivalent | Key difference |
|---|---|---|
| Stream (`XADD`) | Partition (not topic!) | Single log, no auto-sharding |
| Consumer group (`XGROUP CREATE`) | Consumer group | Same idea — named, tracks position |
| Consumer name (per `XREADGROUP`) | Consumer instance | Same idea |
| Message ID (`<ms-timestamp>-<seq>`) | Offset | ID already IS the position (no per-partition split) |
| `XACK` | Offset commit | **Per-message**, not a watermark |
| PEL (pending entries list) | *(no equivalent)* | Kafka has no per-message in-flight tracking |
| `XCLAIM` / `XAUTOCLAIM` | Rebalance | **Manual** — no heartbeat-based liveness |
| `MAXLEN` on `XADD` | Retention | Cap-based trim, not usually time-based |

**Message ID format**: `<milliseconds-since-epoch>-<sequence-number>`, e.g. `1690000000000-0`. Sequence number only increments if multiple messages land in the same millisecond. `XADD stream * field value` — the `*` auto-assigns this ID; nobody in these repos constructs IDs manually. Being always-increasing, it plays the same "defines order" role as a Kafka offset, just without a single global integer counter.

---

## PEL vs. offset watermark — the structural difference that matters most

Kafka tracks one number per `(group, topic, partition)`: "everything up to offset X is done." Crash → redeliver everything after that offset — coarse but simple.

Redis Streams instead track the **PEL**: a set of *specific message IDs* delivered to a specific consumer but not yet acked, each with its own delivery counter.

**Walking through it as a timeline, not a frozen snapshot:**
1. Consumer A reads msg-0 and msg-2; Consumer B reads msg-1. All three enter the PEL with delivery count 1.
2. A finishes and calls `XACK` on both → msg-0 and msg-2 leave the PEL entirely.
3. B crashes before acking msg-1 → it stays stuck in the PEL, attributed to a dead consumer, idle time growing.
4. A janitor sweep notices msg-1 idle past the threshold → `XAUTOCLAIM` reassigns it to a live consumer, delivery count bumps to 2, it's reprocessed.
5. If it keeps failing past `MAX_RETRIES` (checked via the delivery counter), it gets dropped instead of claimed again.

`concerto-pulse` exploits this directly: instead of a separate DLQ topic (Kafka's `dlq.ts` approach), it reads Redis's own per-message delivery counter via `XPENDING` and drops the message once it exceeds `MAX_RETRIES` — no extra topic needed, because Redis was already tracking the counter.

---

## What `XCLAIM` / `XAUTOCLAIM` actually are

**`XCLAIM`** transfers ownership of a specific, already-delivered-but-unacked message from one consumer to another. The situation it exists for: Consumer B pulled a message (marking it "delivered to B" in the PEL), then crashed or hung before calling `XACK`. Nobody else touches it by default, because Redis still thinks B "has" it. `XCLAIM` says: *"that's been idle too long — I'm taking it over now."*

**`XAUTOCLAIM`** is the lazier version — instead of supplying exact message IDs, it scans the whole PEL and claims everything idle past a threshold in one call. `concerto-pulse` uses `XAUTOCLAIM`; `caasi-citadel` uses the more manual `XPENDING` + `XCLAIM` combo.

---

## No automatic rebalancing — you build the coordinator yourself

Kafka's group coordinator detects a dead consumer via missed heartbeats and reassigns its partitions automatically — a broker-side feature, free to you. **Redis Streams has none of this.** If a consumer dies mid-processing, its unacked messages sit in the PEL forever unless the application looks for them.

```python
# concerto-pulse/pulse/services/redis_stream_processor.py
async def _janitor_loop(self):
    """XAUTOCLAIM sweep on every stream every JANITOR_INTERVAL_S.
    Recovers work from crashed consumers, retries failed deliveries..."""
```

You are the group coordinator in Redis Streams — Kafka gives you that for free, Redis makes you build it (`RECLAIM_IDLE_MS`, `JANITOR_INTERVAL_S`, `_cleanup_stale_consumers` in `redis_stream_processor.py`; `_reclaim_pending` in `caasi-citadel/citadel/services/command_stream.py`).

---

## Configuration layers

No separate "admin creates the topic" step, because there's no partition count to decide — stream + group creation happens **inline in application startup code**:

```python
# caasi-citadel/citadel/services/command_stream.py
await redis.xgroup_create(
    name=keys.COMMAND_STREAM,
    groupname=self._group_name,
    id="0-0",              # "0-0" = from beginning · "$" = only new messages
    mkstream=True,
)
```

**Producer (`XADD`) with retry** — application-level, unlike Kafka's client-managed retries:

```python
# concerto-ripples/.../redis_stream_publisher.py
async def _xadd_with_retry(redis, stream, fields, maxlen, approximate=True, attempts=3, backoff_base_ms=100):
    for attempt in range(attempts):
        try:
            return await redis.xadd(stream, fields, maxlen=maxlen, approximate=approximate)
        except Exception as e:
            if attempt == attempts - 1:
                return None
            await asyncio.sleep(backoff_base_ms/1000.0 * (1 + random.random()))
```

**Consumer (`XREADGROUP`)** — batch size and block time:

```python
# concerto-pulse/pulse/services/redis_stream_processor.py
messages = RedisService.xreadgroup(
    group=CONSUMER_GROUP,
    consumer=self._consumer_name,
    streams={s: ">" for s in self._streams},   # ">" = new, undelivered to this group
    count=BATCH_SIZE,
    block=BLOCK_MS,
)
```

**Config map:**

| Layer | Settings |
|---|---|
| Stream + group creation (inline) | `id` ("0-0" vs "$"), `mkstream=True` |
| Producer (`XADD`) | `maxlen`, `approximate`, app-level retry + backoff |
| Consumer (`XREADGROUP`) | `groupname`, `consumername`, `count`, `block`; `XACK` always explicit, no auto-commit |
| Janitor (self-built, no Kafka equivalent) | `min_idle_time`, `XCLAIM`/`XAUTOCLAIM`, `MAX_RETRIES` via `XPENDING`, stale-consumer cleanup via `XGROUP DELCONSUMER` |

No equivalent of Kafka's per-topic `numPartitions`/`replicationFactor` — durability comes from Redis's own instance/cluster replica setup, not per-stream config.

---

## Real patterns — the two ends of the group-membership spectrum

**Load-balanced (competing consumers) — `concerto-pulse`:** every replica shares one group. Redis guarantees each message goes to exactly one replica — textbook Kafka-consumer-group behavior, self-managed instead of broker-managed.

```python
CONSUMER_GROUP = "pulse-redis-stream-processors"  # shared across all replicas
```

**Broadcast (fan-out) — `concerto-ripples`:** each instance gets its *own* group, so every instance sees every message:

```python
# "Each instance uses its OWN consumer group to receive ALL messages (broadcast pattern)"
websocket_group_name = f"{StreamConfig.WEBSOCKET_CONSUMER_GROUP_PREFIX}-{self.instance_id}"
```

This is the same trick as running N separate single-member Kafka consumer groups to get fan-out instead of load-splitting.

**Broadcast + filter — `caasi-citadel`:** also per-instance groups, but then explicitly discards anything not addressed to it:

```python
if target_instance != self.instance_id:
    await redis.xack(keys.COMMAND_STREAM, self._group_name, entry_id)
    return
```

This reveals a real limitation: Kafka would let you *partition by* the owning instance so each pod only ever reads what's meant for it. Redis Streams has no equivalent of routing a message to a specific consumer by key — no partition-key hashing to land it on the "right" shard. So everyone gets everything and self-filters — a direct consequence of "one stream = one log, no partitioning" from the very first point.

---

## What genuinely doesn't exist in Redis Streams

- No brokers-and-controller layer specific to streams — a stream lives on whichever Redis node/shard owns that key.
- No per-topic replication factor — replication is a whole-instance/cluster deployment concern.
- No automatic consumer failure detection or rebalancing — the janitor loop is entirely self-built.
- No key-based deterministic routing to a specific consumer/shard — hence the broadcast-and-filter workaround.

---

## Quick reference: Kafka vs Redis Streams

| Concept | Kafka | Redis Streams |
|---|---|---|
| Unit of ordering | Partition | The whole stream (single log) |
| Getting parallelism | More partitions (broker-managed) | More streams, client-managed sharding |
| Position tracking | Single offset watermark per partition | Per-message PEL with individual delivery counts |
| Ack granularity | Coarse (commit up to offset X) | Per-message (`XACK` individual IDs) |
| Failure detection | Automatic (broker heartbeats + coordinator) | Manual (self-built janitor loop) |
| Reassigning stuck work | Automatic rebalance | `XCLAIM` / `XAUTOCLAIM`, self-triggered |
| Retry-limit / poison messages | Usually a separate DLQ topic | Delivery counter via `XPENDING`, in-band |
| Fan-out to all consumers | Separate single-member groups per consumer | Separate group per consumer/instance |
| Replication config | Per-topic `replicationFactor` | Whole-instance/cluster concern |
| Topic/stream creation | Explicit admin step (`numPartitions`, RF) | Inline, auto via `mkstream=True` |

**Ties back to Citadel's own Redis-streams sync design**: sharding per-clinic to preserve ordering guarantees is the exact same trade-off as Kafka's per-partition ordering — shard by key to get ordering within the shard, give up ordering across shards. And the PEL/pending-entries idea is the same shape as anything left unacked in that design: a message picked up but never confirmed needs an explicit "give up and reassign" mechanism, not indefinite retry.
