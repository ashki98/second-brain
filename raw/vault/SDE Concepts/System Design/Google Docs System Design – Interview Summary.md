# Google Docs System Design – Interview Summary

Google Docs System Design – Interview Summary

## Overview

System design for a collaborative editing platform like Google Docs that enables multiple users to edit documents in real-time.

## Key Requirements

### Functional Requirements

- Users should be able to create, read, update, and delete documents (CRUD)
- Multiple users should be able to concurrently edit the same document
- Updates should be visible in real-time
- Cursor position should be visible to all users

### Non-Functional Requirements

- Support millions of documents
- Up to 100 concurrent users per document
- Latency ≤ 200 milliseconds for concurrent updates
- Documents should be durable and persistently stored
- Documents should converge and be consistent

## API Design

- `POST /api/documents` - Create document
- `GET /api/documents/{id}` - Fetch document contents
- `PUT /api/documents/{id}` - Update document metadata
- `WS /api/documents/{id}/collaborate` - WebSocket endpoint for real-time collaboration

## High-Level Architecture

### Core Components

1. **Client** - User interface
2. **API Gateway** - Routes requests to appropriate services
3. **Document Service** - Handles CRUD operations and metadata
4. **WebSocket Service** - Handles real-time updates
5. **Object Storage** - Stores document blobs (via CDN)
6. **Database** - Stores document metadata and recent edits

### Data Flow

- Static document reads go through CDN → Object Storage
- Document updates go through API Gateway → WebSocket Service
- Edits are stored in database (not entire document)
- WebSocket service broadcasts edits to all connected clients

## Deep Dives

### Sending Updates

**Approach 1: Send whole document**

- Expensive for large documents
- Creates new version each time

**Approach 2: Send only edits**

- Much better for performance
- Smaller payload size
- But requires conflict resolution

**Approach 3: Operational Transform** (Chosen approach)

- Transforms edits to ensure consistency
- Handles conflicts intelligently
- Example: If inserting at position 5, and another edit inserts 1 character at position 5, transform second edit to position 6

### Conflict Resolution with Operational Transform

**Problem:**

```
String: "Hello!"
Edit 1: Insert "," at position 5
Edit 2: Insert " world" at position 5

Without OT: "Hello world,!" (incorrect)
With OT: "Hello, world!" (correct - Edit 2 transformed to position 6)

```

**Solution:**

- Server performs OT on incoming edits
- Server broadcasts transformed edits to all clients
- Clients also perform OT locally for optimistic updates
- Server is source of truth for edit ordering

### Storage Strategy (Hybrid Approach)

1. **Object Storage** - Stores complete document snapshots
2. **Database (Cassandra)** - Stores recent edits (optimized for heavy writes)
3. **When user opens document:**
    - Fetch base document from Object Storage (via CDN)
    - Fetch recent edits from database
    - Apply edits locally
4. **Compaction:**
    - When WebSocket room reaches 0 connections
    - Trigger compaction worker via message queue
    - Apply all edits to object storage
    - Delete/tombstone edits from database

### Scaling WebSocket Service

**Challenge:** WebSocket services are stateful

- All users editing same document must connect to same instance
- Cannot simply load balance across instances

**Solution: Consistent Hashing**

- Use document ID as key
- Route all users of same document to same WebSocket instance
- Example: Doc IDs 0-99 → Instance 1, Doc IDs 100-199 → Instance 2
- If instance fails, consistent hashing re-routes to another instance

## Additional Considerations

### Performance Optimizations

- **Rate limiting** on client side
- **Debouncing** - Don't send every keystroke
- **Client-side batching** - Batch multiple edits before sending

### Version History

- Keep tombstoned edits in database instead of deleting
- Allows reconstruction of document history
- Can show previous versions to users

### Alternative Approaches

**Last Write Wins (LWW)**

- Simpler than OT
- Last edit always wins
- Can lose data
- Not ideal for collaborative editing

**Conflict-free Replicated Data Types (CRDT)**

- More complex than OT
- Guarantees eventual consistency
- Google Docs may have migrated from OT to CRDT

## Technology Choices

- **WebSocket Protocol** - Real-time bidirectional communication
- **Cassandra** - NoSQL database optimized for heavy writes
- **Object Storage** - For storing large document blobs
- **CDN** - For fast document delivery
- **Message Queue** - For triggering async compaction jobs
- **Consistent Hashing** - For WebSocket service routing

## Key Takeaways

1. Sending only edits (not full documents) is critical for performance
2. Operational Transform is essential for conflict resolution
3. Hybrid storage (Object Storage + Database) provides best performance
4. WebSocket services require special scaling considerations
5. Server is source of truth for edit ordering
6. Compaction keeps storage manageable
7. Client-side optimizations (debouncing, batching) reduce server load
