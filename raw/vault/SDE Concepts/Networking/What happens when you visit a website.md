# What happens when you visit a website

What Happens When You Type a URL Into Your Browser? – Concepts Summary (ByteByteGo)

Here's a concise point-by-point summary to help you quickly refresh all key concepts:

1. URL Components
- URL stands for Universal Resource Locator; it's what you enter in your browser.
- A URL has four parts:
    1. Scheme: Specifies the protocol (e.g., http:// or https://). https means encrypted.
    2. Domain: The site name (e.g., [example.com](http://example.com/)).
    3. Path: Like a directory in a file system, pointing to a section of the site.
    4. Resource: The specific file or object (often combined with path).
1. DNS Lookup (Domain Name System)
- When you enter a URL, the browser needs to find the server's IP address via the DNS.
- DNS acts like the internet's phone book, translating domains to IPs.
- Caching:
    - First, browser checks its own cache.
    - If not found, browser checks your computer's (OS) DNS cache.
    - If still not found, the OS queries an external DNS resolver.
    - DNS responses are cached at every level for speed.
1. TCP Connection Establishment
- With the server's IP, the browser establishes a TCP connection.
- TCP handshake:
    - Involves several steps (network round-trips) to start the connection.
- Keep-Alive:
    - Browsers try to reuse existing TCP connections (keep-alive) to improve loading speed.
1. HTTPS and SSL/TLS Handshake
- If the protocol is https, connection setup requires an SSL/TLS handshake (more complex than regular TCP, adds encryption).
- SSL/TLS handshake is resource-intensive, so browsers use optimizations like SSL session resumption to reduce overhead for repeat connections.
1. HTTP Request/Response
- Once the TCP (or HTTPS) connection is ready, the browser sends an HTTP request to the server.
- HTTP is a simple text-based protocol.
- The server processes the request and sends back an HTTP response (often with HTML content).
1. Rendering and Additional Resources
- The browser renders the HTML content.
- Usually, the page references additional resources (e.g., JavaScript files, images, CSS).
- For each resource, the browser repeats:
    - DNS lookup (using cache when possible)
    - TCP connection (reusing connections when possible)
    - HTTP request & response cycles

Quick Review:

- URL structure → DNS lookup → TCP connection (plus SSL/TLS for HTTPS) → HTTP request/response → rendering & fetching more resources.
