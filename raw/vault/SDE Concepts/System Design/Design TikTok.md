# Design TikTok

YT: 

[https://www.youtube.com/watch?v=Z-0g_aJL5Fw](https://www.youtube.com/watch?v=Z-0g_aJL5Fw)

![[image 8.png]]

## 1. **Clarifying Problem & Functional Requirements**

- **Goal:** Design the backend for the TikTok mobile app (video sharing, viewing, interactions).
- **Core Features to Support:**
    - Upload videos (short: max 1 minute)
    - Attach text/captions to each video
    - View video feed (scroll through one at a time)
    - Follow users; see followed users’ videos
    - Like (favorite), comment, and optionally forward videos

---

## 2. **Non-Functional Requirements & Scale**

- **Availability:** Target high availability—99.999% uptime for user reliability
- **Latency:**
    - Must be low for viewing; can use client-side caching and background data pulls
    - Latency impact differs for uploads vs. downloads (discuss trade-offs)
- **Scale Estimation:**
    - Designed for ~1 million daily active users
    - Video size: ~5 MB per 1-minute compressed H.264 video
    - Storage estimation: Each user uploads ~2 videos/day (~10 MB/user/day)
    - User metadata low (~1 KB/user/day)

---

## 3. **High-Level API Endpoints**

- **`/upload_video`** – Accepts video, user data, and optional text
- **`/view_feed`** – Returns up to 10 preselected video links for user’s feed
- **`/user_activity`** – Handles follows, likes, comments (interaction tracking)

---

## 4. **Database Schema & Storage Layer**

- **Relational Database (Postgres or similar)**
    - **User Table**: user_id (UUID), other profile info
    - **Videos Table**:
        - video_id (UUID)
        - user_id (foreign key)
        - blob storage URL
        - metadata/caption
    - **User Activity Table**: tracks likes, follows (uses foreign keys to users/videos)
- **Blob Storage (e.g., S3):**
    - Stores actual video files; DB keeps metadata + storage link

---

## 5. **Feed Generation & Caching**

- **Feed Data:**
    - Combination of videos from followed users and algorithmic recommendations
    - Initial implementation: Focus on followed users' videos
- **Caching:**
    - Use Redis (or similar) to cache top 10 videos per user for instant feed loading
    - Precache service compiles playlists ahead of user access
    - Cache warm-up triggered by background jobs or on-demand (when user is active)
- **Read Pattern:**
    - Read-heavy system: Use read replicas/secondary DBs to handle scale, offload read pressure from primary DB

---

## 6. **Scaling Strategies**

- **Horizontal Scaling for Traffic Spikes:**
    - **CDN (Content Delivery Network):**
        - Distribute video files geographically
        - Caches viral videos to relieve blob storage traffic
    - **Load Balancers:**
        - Distribute API requests among multiple backend instances
        - Support zero-downtime deploys (direct traffic to A/B versioning)
    - **Database Sharding:**
        - Partition DB by region/user segments to balance write load
- **Auto Scaling Groups:**
    - Used for both DB read replicas and caching services (expands capacity as needed)

---

## 7. **Follow-up & Bottlenecks**

- **Potential Bottlenecks**:
    - Read-heavy traffic, especially during viral events
    - Write bottlenecks: DB scaling/sharding for uploads & user activity
    - Geographic scale: Region-specific CDN nodes, data center selection
- **Mitigation Techniques:**
    - CDN placement for heavy content
    - Efficient cache and read replicas
    - Load balancer in front of APIs and DB shards

---

## 8. **Areas for Extra Depth** (if time allows)

- **Feed/Recommendation Algorithm:** Overview alluded to; complex logic not deeply explored (precache service may require its own DB structure)
- **User Metadata Management:** Skipped for brevity; assumed standard
- **Interaction Table Details:** Foreign keys and linking for scalable queries

---

## 9. **Interview Feedback & Learnings**

- **Structure covered:** API design, database schemas, services, scaling strategies (load balancing, CDN, sharding)
- **Advice:** Always clarify scope, make trade-offs explicit, highlight scalability and reliability points

---

## **Quick Review Points:**

- Core features: upload, feed, interaction endpoints
- Backend uses relational DB + blob storage
- Caching and precache for quick feed loading
- Scaled via CDN, load balancers, DB sharding
- Focus on read-heavy design for viral use cases
- Non-functional goals: high availability, low latency, scalability
