---
course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 (Udemy, Stéphane Maarek)
---

# Section 3 — Getting Started with AWS

## Global infrastructure
- **Region** — a cluster of data centers, mostly independent from other regions (own laws/rules, isolation, latency to nearest users). Choosing a region depends on: compliance/data governance, proximity to users (latency), available services (not all services in all regions), and pricing (varies by region).
- **Availability Zone (AZ)** — one or more discrete data centers with redundant power/networking, inside a region, connected with high-bandwidth/low-latency links so a region can offer HA and fault tolerance.
- **Edge Locations / Points of Presence** — 400+ locations globally where AWS can serve content faster to end users (CloudFront, Global Accelerator).
- **Global services** (no region selection needed) vs. **region-scoped services** (most of them — must pick a region every time).

## Console tour
- The AWS Console groups services by category; a region selector in the top bar changes which resources you see for region-scoped services.
