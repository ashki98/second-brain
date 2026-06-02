# Trail: Networking & Web Communication

**Objective:** Trace the full path of a web request — from URL to pixels — and understand every protocol layer it passes through.
**Prerequisites:** None
**Review time:** ~25 min
**Nodes:** 5

---

1. [[What happens when you visit a website]]
   > Start concrete: you type a URL and press Enter. This note walks the entire journey — DNS resolution, TCP handshake, TLS negotiation, HTTP request/response, and browser rendering. Read this first so you have the full narrative. Every subsequent note in this trail zooms into one layer of this flow.

2. [[Networking Fundamentals - Protocols, Layers, and W]]
   > Now that you've seen the journey end-to-end, zoom into the layers. This note covers the OSI model, TCP/IP stack, and the key protocols at each layer (Ethernet, IP, TCP, UDP, HTTP). When you read about "Layer 4 load balancers" or "Layer 7 routing," you'll know exactly what layer that refers to.

3. [[Networking Deep Dive — How the Internet Actually Works]]
   > Zoom out beyond your local network to the actual internet: BGP routing, autonomous systems, physical infrastructure (submarine cables, IXPs). This note answers "how does a packet find its way from Sydney to Toronto?" — the infrastructure layer beneath the protocols.

4. [[Status Code]]
   > HTTP status codes are the vocabulary of web communication. With the full HTTP cycle understood from notes 1–3, this note gives you the language servers use to communicate outcomes — and what your own servers should return in each situation.

5. [[CORS]]
   > The browser enforces the same-origin policy on every request — CORS is the HTTP header mechanism that relaxes it in a controlled way. This note closes the loop on browser security: you now understand not just how requests travel (notes 1–3) but how the browser decides which ones to allow.
