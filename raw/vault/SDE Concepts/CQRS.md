# CQRS

YT: 

[https://www.youtube.com/watch?v=ID-_ic1fLkY&t=402s](https://www.youtube.com/watch?v=ID-_ic1fLkY&t=402s)

Event Sourcing in this video is presented as a way to turn application state into a time‑machine by storing every change as an immutable event, then deriving current and past state by processing those events, often together with CQRS for scalable, auditable systems.

---

## Core idea of Event Sourcing

- Event Sourcing is an architectural pattern where every state change is stored as an immutable event in an append‑only log, instead of only storing the latest state in a typical relational table.
- The “truth” of the system is the ordered event stream (event store), and any current state (like an account balance or product price) is derived by replaying these events.

---

## Why use Event Sourcing

- It provides a perfect audit trail: every action, who did it, and when, can be reconstructed, which is useful for compliance, debugging, and understanding business flows (for example, why an order was cancelled).
- It allows rebuilding state at any point in time (e.g., “what did this customer look like last month?”) by replaying events up to a timestamp, and it naturally supports CQRS by separating events from the read models that consume them.

---

## Sourcing, Hydration, and Replay

- **Sourcing**: The overarching idea that state is “sourced” from events rather than stored directly; the latest state is always a function of the event history.
- **Hydration**: Rebuilding the current state of a single entity (e.g., one product) on demand by applying all its events sequentially, typically when serving a query or loading an aggregate in memory.
- **Replay**: Processing past events (often all events, or a large subset) to regenerate system state, validate changes, migrate to a new model/schema, or rebuild corrupted read models.

---

## Performance: Snapshots and Materialized Views

- **Snapshots**: Periodic captures of the current state of an entity (e.g., every 100 events) so that, on future hydration, only events after the last snapshot need to be replayed; this optimizes the “write/rehydrate” side.
- **Materialized views**: Dedicated read‑side projections stored in a separate, query‑optimized database that are incrementally updated as new events arrive, enabling fast queries without replay on every read.

---

## CQRS and Event Sourcing together

- CQRS separates the **command (write) side**, which handles commands that change state, from the **query (read) side**, which handles data retrieval; Event Sourcing fits naturally into this by having commands emit events instead of directly mutating state.
- The command side validates business rules and appends events to the event store, while the query side subscribes to those events, maintains read models (materialized views), and serves user queries from those denormalized, query‑optimized stores.

---

## End‑to‑end example (product price)

- A seller issues an `UpdateProductPrice` command (e.g., from 100 to 120); the command service validates rules and generates a `PriceUpdated` event appended to the event log (event store).
- The query service subscribes to this event (via broker or direct event‑store subscription), updates the product’s read model (e.g., current price in a read DB), and future product detail queries read directly from this updated model.

---

## Event propagation

- Event propagation is the process of distributing events from the command side to all interested components/services that need to react, often via message brokers like Kafka, RabbitMQ, or AWS SNS.
- The query side typically subscribes to these topics or directly to the event store (e.g., EventStoreDB), processes new events to update read models, and can rebuild them from scratch by replaying the full event history if corruption or schema changes occur.
