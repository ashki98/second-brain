# CDN Concepts & AWS CloudFront Developer Flow

CDN Concepts & AWS CloudFront Developer Flow

## CDN Concepts & CloudFront Developer Flow

**What is a CDN? Why use one for web apps?**

- A **Content Delivery Network (CDN)** is a distributed system of servers (PoPs, edge servers) that delivers web content to users globally, routing requests to the nearest server for performance.
- CDNs accelerate delivery of static (and increasingly, dynamic) HTTP content to users; originally for HTML, now for all web service traffic.

**How does a CDN work?**

- **PoPs (Points of Presence):** Edge servers installed in hundreds of global locations mean users get their content from the nearest PoP.
- **Routing methods:**
    - *DNS-based Routing:* Each PoP has a unique IP, DNS directs users to the nearest.
    - *Anycast:* All PoPs share a single IP, network determines "nearest" for any request.
- **Edge Servers:**
    - *Reverse Proxy & Caching:* Content is cached at the edges, minimizing trips to origin server, reducing bandwidth load, and boosting speed.
    - *Optimizations:* Can minify and convert assets to optimal formats (e.g., JS minification, image format upgrades to WebP/AVIF).
    - *TLS Termination:* Secure connections established at the edge, lowering user latency (expensive handshakes occur closer to the user).

**Benefits of CDNs:**

- **Performance:** Speeds up delivery by serving cached/static content from edge.
- **Security:** Edge network absorbs DDoS attacks (especially with Anycast), provides robust security.
- **Availability:** Content is resilient against server/location failures (redundancy at multiple PoPs).

**How does CloudFront fit into a modern web development workflow?**

- CloudFront is AWS's CDN—here's the typical flow for deploying a React app (in GitHub) with static assets (S3):
    1. **Build deployment:**
        - Build React app locally (minified files), upload build folder to S3.
    2. **Configure S3:**
        - Enable static web hosting, set `index.html` as entry/error document.
    3. **Set up CloudFront:**
        - Create a distribution with your S3 bucket as origin.
        - CloudFront fetches and caches all assets, serving globally (fast!).
    4. **Update/Invalidate cache:**
        - When updating the React app, use "invalidations" to refresh CDN content.
    5. **HTTPS/domain setup:**
        - Use Amazon Certificate Manager for SSL; attach your custom domain to CloudFront.
    6. **Security:**
        - Restrict direct public S3 access using Origin Access Control (OAC) or Origin Access Identity (OAI); ensure assets served only via CloudFront.

**Summary of developer tasks for a working, production-grade setup:**

- Build and upload React static files to S3.
- Configure S3 for web hosting and correct access permissions.
- Set up CloudFront with S3 as origin, configure SSL certificates for HTTPS.
- Point your DNS to the CloudFront distribution.
- Invalidate CloudFront cache on updates/releases.
