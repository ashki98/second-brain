# Design URL Shortner

YT: 

[https://www.youtube.com/watch?v=qSJAvd5Mgio](https://www.youtube.com/watch?v=qSJAvd5Mgio)

![[image 9.png]]

**URL Shortener System Design (Bitly) — Comprehensive Notes & Concept Refresh**

**Purpose:** Use this as a quick, structured reference for reviewing all system design concepts covered in the NeetCodeIO Bitly/URL shortener video.

---

## 1. **Question Approach & Structure**

- System design is open-ended and ambiguous; structure your answer for clarity.
- Typical process:
    - **Requirements Gathering**
    - **API Design**
    - **High-Level Architecture**
    - **Deep Dives/Scaling/Efficiency & Edge Cases**

---

## 2. **Functional & Non-Functional Requirements**

## **Functional**

- **URL Shortening:** Map a long URL to a unique short URL.
- **URL Redirection:** Map the short URL back to the original long URL; must be quick.
- **Link Analytics (bonus):** Track how many times a URL is accessed.
- (Optional: Support custom slugs, expiration, user authentication — not covered deeply.)

## **Non-Functional**

- **Low Latency:** Especially on redirection.
- **High Throughput:** E.g., support up to 100M daily active users, 1B read requests/day (~10,000 RPS).
- **High Availability**
- **Scalability**
- Sufficient capacity (1–5B URLs lifetime).
- Writes scale more slowly (~millions only).

---

## 3. **API Design**

- **POST /api/urls/shorten**
    - Request: `{ long_url }`
    - Response: `{ short_url }`
- **GET /api/urls/{short_url}**
    - Request: `{ short_url }` (via URL param)
    - Response: HTTP 302/301 redirect to `{ long_url }`
    
    **Key status codes:**
    
    - **301:** Permanent redirect (browser caches, may skip server in future requests—affects analytics).
    - **302:** Temporary, not cached—good for analytics as every hit goes to the server.

---

## 4. **Basic Architecture**

- **Client (browser)**
- **URL Service (handles API endpoints)**
- **Database:** Stores `{short_url, long_url, user_id, created_at, used_count}`.

**Initial Toy Version:** Local hashmap only (for demonstration)—NOT production ready!

---

## 5. **Scalable High-Level Design**

- **Introduce API Gateway:** (usually excessive for just two endpoints, but common in larger systems).
    - Allows splitting into:
        - **URL Shortening Service**
        - **URL Redirection Service**
    - API Gateway handles routing, authentication, etc.
- **Decouple database for scalability** (not tightly coupled to business logic/service).

---

## 6. **Unique Short URL Generation Strategies**

- **Counter (monotonic integer):**
    - Use e.g., Redis to store & increment a global counter; encode with base62 (0–9, a–z, A–Z).
    - **Base62, 6 chars:** ~56B combos; enough for billions of URLs; 7 chars = trillions.
    - Redis is single-threaded, helps with atomicity.
    - **Drawback:** Counter is a single point of failure; replication adds complexity.
- **Random String Generation:**
    - Generate 6/7-length random base62 string.
    - Check for collision in DB; if collision, retry.
    - With 5B URLs and 3T possible values, collision probability is low.
    - Slightly slower due to DB lookups, but shortening latency is NOT critical.
- **Hashing:**
    - Hash (`MD5`, `SHA-256`) of input, take first 6-7 chars as short URL.
    - Also check for collision; if so, retry.
    - No need for global counter, easier to scale horizontally.
- **(Mentioned in comments, not video) Distributed generation:** Twitter snowflake-like patterns.
- **Note:** Main trade-offs are simplicity, reliability, and scalability vs. collision-resistance.

---

## 7. **Key Database Design Points**

- Store mapping: `short_url`, `long_url`, `user_id`, `created_at`, `usage_count`.
- Each record: ~1KB; a billion URLs ≈ 1TB, manageable on modern hardware.

---

## 8. **Latency & Throughput Optimization: Read Path**

- **Index primary key (short_url)** for fast lookup.
- **Read Replicas:** To horizontally scale reads (not writes).
- **Caching with Redis:**
    - Most-used short URLs in Redis (memory is ~10x faster than disk).
    - Greatly reduces DB load and improves redirect latency.
    - Usually, traffic is highly skewed (few URLs used a lot).
- **Avoid sharding** unless scale is extreme (adds complexity).

---

## 9. **Analytics/Write Path & Optimization**

- **Analytics service:** update usage count per redirect.
    - **Don’t hit main DB with every request!**
    - Use in-memory store (like Redis) for fast, frequent increments.
    - Periodically flush counts to DB (e.g., via cron job every 60s).
    - Loss of a few seconds data is acceptable in most scenarios.
    - Alternative: Batch write or use a message queue for decoupling.

---

## 10. **Trade-offs & Key Points**

- **301 vs 302 Redirect:**
    - 301 is faster for recurring user visits, but bypasses server (hurts analytics).
    - 302 ensures every hit is registered but increases server load.
- **DB Scaling:**
    - Start with a single-node RDBMS; add cache/read replicas as needed.
    - Only shard if absolutely necessary.
- **Service Splitting:**
    - Not required for small/mid-scale, demonstrated for interview completeness.
- **Failure/Resilience:**
    - Watch out for single points of failure (e.g., Redis in counter mode).
    - Acceptable to keep things simple unless interview specifically asks for distributed failover or high resilience.

---

## 11. **Interview Tips Emphasized**

- Always clarify/explain your trade-offs and reasoning.
- Structure your answer, don’t jump in with architecture immediately.
- Don’t over-engineer unless asked (“right-sized” solution).
- Talk about bottlenecks (DB, cache, network, etc.).
- Focus on what matters: scaling reads, handling analytics without impacting performance, generating truly unique short URLs.

---

## 12. **Your Specific Doubts/What Was Covered Here**

- **URL Generation (Counter vs Random vs Hash):** Approaches and tradeoffs explained clearly; how to handle collisions emphasized.
- **Redirect Status Codes (301 vs 302):** Explicitly discussed. Why analytics don’t work with 301 and how 302 solves it (extra load vs data completeness).
- **Analytics Path:** Use of in-memory store (Redis), periodic flushing, and why DB direct-writes are a performance anti-pattern.
- **Database Indexing:** Mentioned as important for latency but not dwelled on.
- **Caching:** Clearly highlighted as biggest win for real-world scale and latency.
- **Microservices & API Gateway:** When and why to split, and when it’s overkill.
- **Scaling Path:** Start simple, add complexity/reactive scale.

---

**This note is structured for “refresh before interview” purposes.** Refer back to any section as needed, and extend any point with implementation details in Python/FastAPI/Node.js as per your stack.

1. [https://www.youtube.com/watch?v=qSJAvd5Mgio](https://www.youtube.com/watch?v=qSJAvd5Mgio)
