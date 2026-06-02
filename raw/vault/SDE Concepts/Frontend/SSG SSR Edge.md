# SSG/ SSR/ Edge

| **Feature** | **Server Components** | **Client Components** | **Edge Functions** |
| --- | --- | --- | --- |
| Runs On | Server (before reaching browser) | Browser (after page loads) | Edge (CDN, close to user) |
| JavaScript | ❌ No JS in browser | ✅ JS runs in browser | ✅ Runs on CDN servers |
| Best For | Static content, API calls, SEO | Interactivity, event handling | Real-time personalization, low latency |
| Performance | ✅ Fast (no hydration) | ⚠️ Slower (hydration required) | ✅ Super fast (global execution) |

### CDN: **Content Delivery Network (AWS Cloudfront)**

**When Should You Use a CDN?**

•	**Hosting images, videos, static assets (JS, CSS)**

•	**Speeding up website loading times**

•	**Reducing traffic on your main server**
