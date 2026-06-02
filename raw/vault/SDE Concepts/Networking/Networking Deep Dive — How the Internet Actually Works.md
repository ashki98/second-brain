# Networking Deep Dive — How the Internet Actually Works

> **Purpose:** These are comprehensive notes from a deep-dive conversation covering how a request flows from your browser to a server and back. Every concept builds on the previous one. Read sequentially the first time, use the table of contents for reference later.
> 

---

## Table of Contents

1. [The Big Picture — End-to-End Request Flow](https://claude.ai/chat/83cdeb20-331b-4abb-bdea-357355ca6186#1-the-big-picture)
2. [The OSI Layer Model — What Each Layer Actually Does](https://claude.ai/chat/83cdeb20-331b-4abb-bdea-357355ca6186#2-the-osi-layer-model)
3. [Who Does What — Browser vs Kernel vs NIC vs Routers](https://claude.ai/chat/83cdeb20-331b-4abb-bdea-357355ca6186#3-who-does-what)
4. [Sockets — What They Actually Are](https://claude.ai/chat/83cdeb20-331b-4abb-bdea-357355ca6186#4-sockets)
5. [TCP — The Reliability Engine](https://claude.ai/chat/83cdeb20-331b-4abb-bdea-357355ca6186#5-tcp-the-reliability-engine)
6. [Ports — How the OS Routes Packets to Processes](https://claude.ai/chat/83cdeb20-331b-4abb-bdea-357355ca6186#6-ports)
7. [HTTP — The Application Protocol](https://claude.ai/chat/83cdeb20-331b-4abb-bdea-357355ca6186#7-http)
8. [Encapsulation — The Onion Model](https://claude.ai/chat/83cdeb20-331b-4abb-bdea-357355ca6186#8-encapsulation)
9. [Ethernet, MAC Addresses, and the NIC](https://claude.ai/chat/83cdeb20-331b-4abb-bdea-357355ca6186#9-ethernet-mac-addresses-and-the-nic)
10. [How Packets Find Their Way — ARP, DHCP, Routing](https://claude.ai/chat/83cdeb20-331b-4abb-bdea-357355ca6186#10-how-packets-find-their-way)
11. [Connection Reuse — Keep-Alive and HTTP/2](https://claude.ai/chat/83cdeb20-331b-4abb-bdea-357355ca6186#11-connection-reuse)
12. [What We Deliberately Skipped](https://claude.ai/chat/83cdeb20-331b-4abb-bdea-357355ca6186#12-what-we-deliberately-skipped)
13. [Connection to Voice/Telephony (RTP, WebRTC)](https://claude.ai/chat/83cdeb20-331b-4abb-bdea-357355ca6186#13-connection-to-voice-telephony)

---

## 1. The Big Picture

When you type `google.com` and press Enter, here's the complete journey in 7 stages:

```
STAGE 1: Browser (user space)
  → socket()      → Kernel allocates empty socket struct
  → connect()     → Kernel picks ephemeral port, does TCP 3-way handshake
  → write()       → Kernel copies HTTP text into send buffer

STAGE 2: Kernel (still inside your machine)
  → TCP segments the send buffer, adds ports + seq numbers
  → IP wraps it with source/destination IPs
  → Ethernet wraps it with source/destination MACs (next hop = router)
  → Places frame in NIC's transmit queue

STAGE 3: NIC converts frame to electrical signals → onto the wire

STAGE 4: Router-to-router hops (10-20 routers)
  → Each router: strip Ethernet → read IP destination → consult routing table
  → Build NEW Ethernet frame for next hop → send
  → IP addresses stay the same. MAC addresses change at every hop.
  → Routers NEVER look at TCP or HTTP.

STAGE 5: Google's machine (reverse of stages 1-3)
  → NIC receives signals → reconstruct Ethernet frame
  → Kernel strips Ethernet → strips IP → reads TCP 5-tuple
  → Matches to the correct socket → delivers payload to receive buffer
  → Web server calls read() → gets "GET / HTTP/1.1..."
  → Generates response → calls write() → "HTTP/1.1 200 OK..." into send buffer

STAGE 6: Response travels back (same router hops, possibly different path)
  → 52KB response split into ~36 TCP segments
  → Each segment independently routed back to your machine
  → Your kernel reassembles in order using sequence numbers
  → Sends ACKs back to Google confirming receipt

STAGE 7: Browser renders
  → read() → gets complete HTTP response
  → Parses HTML → discovers CSS/JS/images → fetches via same socket (keep-alive)
  → DOM + CSSOM → render tree → layout → paint → pixels on screen
```

**Key insight:** The flow is symmetrical. Your machine's stages 1-3 mirror Google's stages 5-6. The browser only ever calls four functions: `socket()`, `connect()`, `write()`, `read()`. Everything else is the kernel's job.

---

## 2. The OSI Layer Model

| Layer | What It Does | Who Runs It | Example Protocols |
| --- | --- | --- | --- |
| **Application** | Formats the actual message (HTTP request, DNS query) | Browser / app process | HTTP, DNS, RTP |
| **Transport** | Reliable (TCP) or fast (UDP) delivery between endpoints | OS kernel on both ends | TCP, UDP, QUIC |
| **Network** | Routes packets across the internet by IP address | OS kernel + every router | IPv4, IPv6 |
| **Data Link** | Delivers frames to the next physical device by MAC | OS kernel + NIC firmware | Ethernet, Wi-Fi |
| **Physical** | Converts bits to electrical signals / radio waves | NIC hardware | Cables, radio |

### Critical distinction: which devices have which layers

```
Your machine:     All 5 layers
Routers:          Only layers 1-3 (Physical, Data Link, Network)
Google's machine: All 5 layers
```

**Routers never see TCP headers or HTTP content.** They only read the IP destination address, make a forwarding decision, and send the packet to the next hop. This is what makes routing fast — millions of packets per second, each requiring only a single IP lookup.

### Why the layers are separate (not merged)

The transport and network layers are both handled by the kernel, but they solve fundamentally different problems:

- **Network layer (IP):** "How does this packet get from machine A to machine B?" — involves every router in between.
- **Transport layer (TCP/UDP):** "How do I make this delivery reliable/fast?" — only the two endpoints care.

They're separate because they **change independently:**

- Switching from Ethernet to Wi-Fi? Only the data link layer changes. IP and TCP don't care.
- Inventing a new transport protocol (like QUIC)? It slots on top of existing IP. Every router in the world keeps working without changes.
- TCP vs UDP? Two different transport strategies, both running on the same IP layer.

**Transport layer diversity is cheap** — you only need to update the two endpoints. **Network layer diversity is expensive** — every router in the world needs to understand the new protocol. This is why IPv6 adoption took 20+ years while QUIC spread in a few years.

### What exists at the network layer

IP essentially won. The main protocols are:

- **IPv4** (142.250.195.46 style) — still dominant
- **IPv6** (2404:6800:4007:0811::200e style) — gradually replacing IPv4
- **ICMP** (ping, traceroute) — diagnostics, not data transfer

Historical competitors like IPX (Novell) and AppleTalk (Apple) are dead.

---

## 3. Who Does What

### Browser (user-space application)

The browser's entire network vocabulary is four system calls:

| System Call | What It Does |
| --- | --- |
| `socket(TCP)` | Asks kernel to allocate an empty socket data structure |
| `connect(ip, port)` | Asks kernel to do TCP handshake with the server |
| `write(data)` | Pushes bytes into the socket's send buffer |
| `read()` | Reads bytes from the socket's receive buffer |
| `close()` | Asks kernel to tear down the connection (FIN) |

**The browser never:** constructs a TCP header, picks a sequence number, sends an ACK, chooses an ephemeral port, or touches an Ethernet frame. It doesn't even know its own ephemeral port — the kernel picked that silently during `connect()`.

### OS Kernel

Handles everything between the system call and the NIC:

- TCP: segmentation, sequence numbers, ACKs, retransmission, flow control, congestion control
- IP: source/destination addressing
- Ethernet: frame construction, MAC addresses (via ARP)
- Socket table: maps 5-tuples to sockets, delivers incoming data to the right process
- NIC driver: places frames in transmit queue, reads from receive queue

### NIC (Network Interface Card)

The physical hardware at the boundary between software and the wire:

- **Outbound:** reads frames from transmit queue → converts to electrical signals / radio waves
- **Inbound:** detects signals → reconstructs Ethernet frame → checks "is destination MAC mine?" → if yes, places in receive queue and interrupts kernel
- The NIC filters out frames not addressed to it — this is the first filter before the kernel even knows a frame exists

### Routers

Operate at layers 1-3 only:

1. NIC receives signals → reconstructs Ethernet frame
2. Strips Ethernet header → reads IP destination
3. Consults routing table → determines next hop
4. Uses ARP to get next hop's MAC address
5. Wraps IP packet in a NEW Ethernet frame → sends out another NIC

A router is essentially a device with multiple NICs: one facing your network, another facing the ISP, etc.

---

## 4. Sockets

### What a socket is NOT

A socket is not a wire, not a connection on the network, not a physical thing. The wire doesn't know about "connections." Every packet travels independently through routers.

### What a socket IS

A socket is a **data structure in kernel memory** — a state machine that tracks everything about one conversation:

```
Socket object (file descriptor) in kernel memory:
┌─────────────────────────────────────────────────────┐
│ Identity (5-tuple):                                  │
│   (TCP, 192.168.1.42, 52384, 142.250.195.46, 80)   │
│                                                      │
│ Send buffer:     data waiting to be sent out         │
│ Receive buffer:  data arrived, waiting for app read  │
│                                                      │
│ Sequence tracking:                                   │
│   Next seq# to send: 401                             │
│   Next seq# expected from remote: 14601              │
│                                                      │
│ Congestion window, RTT estimates, retransmit timers  │
│                                                      │
│ State: ESTABLISHED                                   │
└─────────────────────────────────────────────────────┘
```

### Why sockets exist — what would happen without them

Without the socket's persistent state, TCP can't do its three key jobs:

| Problem | Without socket | With socket |
| --- | --- | --- |
| Segment #14 arrives before #12 | Nobody holds #14 or waits for #12 | Kernel holds #14 in buffer until #12 arrives, delivers both in order |
| Segment #7 is lost | Nobody notices, nobody retransmits | Timer expires, no ACK received → retransmit |
| Sender is too fast | Receiver overflows, data lost | Window size advertised → "I can only accept 16KB more, slow down" |

Without sockets, you essentially have UDP — fire and forget, no state tracking.

### The socket lifecycle

```
Browser:  socket()           → Kernel creates empty struct. State: CLOSED
Browser:  connect(ip, port)  → Kernel sends SYN. State: SYN_SENT
                             ← Kernel receives SYN-ACK. State: ESTABLISHED
                             → Kernel sends ACK.
Browser:  write(data)        → Data goes into send buffer. Kernel segments & sends.
Browser:  read()             ← Data from receive buffer delivered to browser.
Browser:  close()            → Kernel sends FIN. State: FIN_WAIT. Eventually freed.
```

### Server-side sockets — the listening socket

Google's web server doesn't create a socket when your SYN arrives. It **already has one:**

```
Server startup:
  socket()  → create socket
  bind(80)  → attach to port 80
  listen()  → mark as "ready for incoming connections"
              This is the LISTENING SOCKET.

When your SYN arrives:
  Kernel sees "someone is listen()ing on port 80"
  → Creates a BRAND NEW socket for your specific connection
  → SYN-ACK sent back
  → accept() returns the new socket to the web server process

Result: TWO sockets on Google's side:
  1. Listening socket — still waiting for more connections
  2. Your connection socket — dedicated to talking to you
```

If no one were listening on port 80, the kernel would send RST (reset/rejection) instead of SYN-ACK.

---

## 5. TCP — The Reliability Engine

### The 3-way handshake (SYN → SYN-ACK → ACK)

SYN literally means "synchronize" — it's where both sides agree on **starting sequence numbers:**

```
Browser → Google:   SYN, seq=1000
                    "My first data byte will be #1001"

Google → Browser:   SYN-ACK, seq=5000, ack=1001
                    "My first data byte will be #5001.
                     I'm ready for your byte #1001."

Browser → Google:   ACK, ack=5001
                    "Ready for your byte #5001."

Connection: ESTABLISHED
```

**Why random starting numbers?** If every connection started at seq=0, leftover packets from a previous connection could be confused with a new one. Random starting numbers prevent this.

### How sequence numbers and ACKs track data

Each segment says "I start at byte #N and carry L bytes." Each ACK says "I've received everything up to byte #N."

**Example — browser sends 500-byte HTTP request (fits in one segment):**

```
Browser → Google:   seq=1001, len=500  (bytes 1001–1500)
Google → Browser:   ACK 1501           "Got it all. Send me #1501 next."
```

**Example — Google sends back 52KB response (split into segments):**

```
Google → Browser:   seq=5001, len=1460  (bytes 5001–6460)
Google → Browser:   seq=6461, len=1460  (bytes 6461–7920)
Google → Browser:   seq=7921, len=1460  (bytes 7921–9380)
Browser → Google:   ACK 9381            "Got all three. Next: #9381"
```

ACKs are **cumulative** — ACK 9381 means "I have every byte from 5001 through 9380."

### What happens when a segment is lost

```
Google → Browser:   seq=5001, len=1460     ✓ Received
Google → Browser:   seq=6461, len=1460     ✗ LOST!
Google → Browser:   seq=7921, len=1460     ✓ Received (held in buffer, out of order)

Browser → Google:   ACK 6461               "I still need byte #6461!"
Browser → Google:   ACK 6461               (duplicate — same number)

Google sees duplicate ACKs → RETRANSMIT:
Google → Browser:   seq=6461, len=1460     ✓ Retransmitted!

Browser now has all three → delivers in order to the application
Browser → Google:   ACK 9381               "Got everything now."
```

The browser **can't advance the ACK past the gap.** It keeps repeating the same ACK number until the missing segment arrives. The socket holds out-of-order segments in its receive buffer until the gap is filled.

### SYN is handshake-only; ACK is continuous

- **SYN flag:** one-time event at connection setup. Never appears again.
- **ACK flag:** on virtually **every segment** for the rest of the connection. It piggybacks on data segments:

```
Google's data segment:  seq=5001, len=1460, ACK=1501, flags=[ACK, PSH]
```

This simultaneously says "here's my data" AND "I confirm I received your data up to byte 1500."

Pure ACKs (no data, just confirmation) only happen when one side has nothing to send but needs to confirm receipt.

### TCP segment size

- MSS (Maximum Segment Size) ≈ **1,460 bytes**
- Derived from: Ethernet frame (1,500 bytes) − IP header (20 bytes) − TCP header (20 bytes)
- A small HTTP request (~400 bytes) fits in **one segment** — TCP doesn't split it further
- A large response (52KB) gets split into ~36 segments
- The kernel fills each segment as full as possible to reduce overhead (each segment carries 40 bytes of headers regardless of payload size)

### What "tracks which bytes have been ACKed" means

The sender's socket maintains a **sliding window:**

- Everything below the ACK number → confirmed delivered → freed from send buffer
- Everything above → still in-flight → kept in send buffer for possible retransmission
- Retransmit timer per batch → if no ACK within estimated round-trip time, resend

---

## 6. Ports

### How the OS maps ports to processes

Each active socket is an entry in the kernel's table:

```
(Protocol, LocalIP, LocalPort, RemoteIP, RemotePort) → owning process
```

When a packet arrives, the kernel reads the 5-tuple from the packet headers and looks it up:

```
(TCP, 78.15.200.3,  49812, google_ip, 80)  → socket #4201 (German user)
(TCP, your_ip,      52384, google_ip, 80)  → socket #4202 (YOU)
(TCP, 201.68.4.17,  52384, google_ip, 80)  → socket #4203 (Brazilian user)
(TCP, *,            *,     google_ip, 80)  → listening socket (catches new SYNs)
```

**Your source port is what makes you unique.** A million people all connect to Google port 80, but each has a different source IP + source port combination.

### Port ranges

- **0–1023:** Well-known (HTTP=80, HTTPS=443, DNS=53). Require root/admin to bind.
- **1024–49151:** Registered/common applications.
- **49152–65535:** Ephemeral (temporary). The OS picks one for you when you call `connect()`.

### TCP and UDP port namespaces are separate

TCP port 80 and UDP port 80 are completely independent — they go to different lookup tables. This is because the protocol is part of the 5-tuple.

### Browsers use multiple ports simultaneously

- Each connection needs its own socket → its own ephemeral port
- Under HTTP/1.1, browsers open **6-8 parallel connections** to the same domain for parallel resource fetching
- 10 tabs open → potentially 60+ ephemeral ports active simultaneously
- You can see this with `ss -tn` or `netstat -tn` on Linux

---

## 7. HTTP

### HTTP is plain text

The browser literally writes this ASCII string into the TCP socket:

```
GET / HTTP/1.1\r\n
Host: www.google.com\r\n
User-Agent: Mozilla/5.0 (Linux; ...) Chrome/126.0\r\n
Accept: text/html,application/xhtml+xml,*/*\r\n
Accept-Language: en-US,en;q=0.9\r\n
Accept-Encoding: gzip, deflate, br\r\n
Connection: keep-alive\r\n
\r\n
```

No binary encoding, no compression of headers. Just a string following RFC rules.

### What each header does

| Header | Purpose |
| --- | --- |
| `GET / HTTP/1.1` | Method (GET), path (/), protocol version |
| `Host: www.google.com` | Which site (one IP can host hundreds of sites) |
| `User-Agent` | "I'm Chrome on Linux" |
| `Accept` | "Send me HTML preferably" |
| `Accept-Language` | "I prefer English" |
| `Accept-Encoding` | "I can decompress gzip/brotli" |
| `Connection: keep-alive` | "Don't close the socket after responding" |

**Why Host matters:** One IP can host hundreds of websites (shared hosting, CDNs). The Host header tells the server which site you want. This is how Nginx decides where to route — virtual host matching.

### How the server sees it

The server receives the exact same text as raw bytes on the TCP stream. `0D 0A` is `\r\n` (carriage return + newline) — that's how the server knows where one header ends and the next begins. Two `\r\n` in a row = end of headers, start of body (empty for GET).

### The response

```
HTTP/1.1 200 OK\r\n
Content-Type: text/html; charset=UTF-8\r\n
Content-Length: 52847\r\n
Content-Encoding: gzip\r\n
Cache-Control: private, max-age=0\r\n
Connection: keep-alive\r\n
\r\n
<!doctype html><html>...52KB of HTML...</html>
```

### After the HTML arrives — the rendering cascade

1. **Parse HTML → build DOM** (the document structure tree)
2. **Discover resources** — `<link href="styles.css">`, `<script src="app.js">`, `<img src="logo.png">`
3. **Fetch resources** — parallel GET requests over the same socket (keep-alive)
4. **Parse CSS → build CSSOM** (the styling rules). CSS is render-blocking — page won't paint until CSS is ready.
5. **DOM + CSSOM → render tree** (what's visible and how it looks)
6. **Layout** → calculate exact pixel positions
7. **Paint** → draw pixels to screen

---

## 8. Encapsulation

Each layer wraps the data with its own header, forming nested layers:

```
┌──────────────────────────────────────────────────────┐
│ Ethernet frame                                        │
│  Src MAC: aa:bb:cc:dd:ee:ff (your NIC)               │
│  Dst MAC: 11:22:33:44:55:66 (your ROUTER — not Google)│
│ ┌──────────────────────────────────────────────────┐  │
│ │ IP packet                                         │  │
│ │  Src IP: 192.168.1.42 (you)                       │  │
│ │  Dst IP: 142.250.195.46 (Google)                  │  │
│ │ ┌──────────────────────────────────────────────┐  │  │
│ │ │ TCP segment                                   │  │  │
│ │ │  Src port: 52384 → Dst port: 80               │  │  │
│ │ │  Seq: 1001 | Flags: PSH, ACK                  │  │  │
│ │ │ ┌──────────────────────────────────────────┐  │  │  │
│ │ │ │ HTTP data                                 │  │  │  │
│ │ │ │  GET / HTTP/1.1\r\nHost: google.com...    │  │  │  │
│ │ │ └──────────────────────────────────────────┘  │  │  │
│ │ └──────────────────────────────────────────────┘  │  │
│ └──────────────────────────────────────────────────┘  │
│  [Checksum trailer]                                   │
└──────────────────────────────────────────────────────┘
```

**Critical detail:** The Ethernet destination MAC is your **router's**, not Google's. Your machine only knows how to reach the next hop. Each router strips and rebuilds the Ethernet frame for the next hop.

### What stays the same vs what changes during transit

| Field | Stays the same? | Why |
| --- | --- | --- |
| HTTP data | ✓ | Endpoints only |
| TCP ports, seq numbers | ✓ | Endpoints only |
| IP source & destination | ✓ | Needed for end-to-end routing |
| Ethernet MAC addresses | ✗ Changes at every hop | Only valid for one physical link |

### De-encapsulation (at Google's end)

The reverse process: strip Ethernet → strip IP → strip TCP → deliver HTTP payload to the process bound to the destination port.

---

## 9. Ethernet, MAC Addresses, and the NIC

### What is Ethernet?

A **data link layer protocol** — the rules for how devices physically connected to the same local network communicate. It handles delivery **for one hop at a time**, using MAC addresses.

### What is an Ethernet frame?

The package format for hop-level delivery:

- **Header:** destination MAC, source MAC, EtherType (says "this contains IPv4/IPv6")
- **Payload:** the IP packet
- **Trailer:** checksum to verify frame wasn't corrupted in transit

### MAC addresses

Every NIC has a unique MAC address burned in at the factory (e.g., `aa:bb:cc:dd:ee:ff`). These only matter **locally** — they identify devices on the same physical network segment. They have no meaning across the internet.

### How bits travel on the wire

The NIC converts each Ethernet frame into electrical signals (voltage changes on copper), light pulses (in fiber), or radio waves (Wi-Fi). Frames are sent **sequentially** — one complete frame, then a brief inter-frame gap, then the next frame.

Modern Ethernet uses encoding schemes where voltage patterns represent groups of bits, but conceptually it's a sequential stream of bits per frame.

### Wi-Fi vs Ethernet

Both are data link protocols solving the same problem (local hop delivery via MAC addresses), but:

- **Ethernet (wired):** dedicated point-to-point link via a switch. No collisions.
- **Wi-Fi (wireless):** shared radio airspace. Devices "listen before talking" — if the air is busy, wait a random time and retry. Collisions can happen; both devices back off and retry. This is why Wi-Fi slows down with many devices.

### How the NIC manages multiple processes sending at once

Inside your machine: the NIC has a **transmit queue** (first-come-first-served). Frames from different processes are sent sequentially. At gigabit speeds, a 1,500-byte frame takes ~12 microseconds — appears simultaneous but it's actually taking turns.

---

## 10. How Packets Find Their Way

### DHCP — How your machine gets its network configuration

When you connect to a network, your machine broadcasts: "I just joined, I need an address."

Your router responds with:

- **Your IP:** 192.168.1.42
- **Subnet mask:** 255.255.255.0
- **Default gateway:** 192.168.1.1 (the router's IP)
- **DNS server:** 1.1.1.1 or your ISP's resolver

### ARP — How your machine gets the router's MAC from its IP

Your machine knows it needs to send to 192.168.1.1 (router), but Ethernet needs MAC addresses. ARP bridges the gap:

```
Your machine broadcasts:  "Who has IP 192.168.1.1? Tell me your MAC."
Router responds:          "That's me. My MAC is 11:22:33:44:55:66."
```

Your machine caches this mapping. Same ARP process happens at every router hop.

### Routing — How routers know the next hop

- **Your home machine:** simple rule — "if destination isn't local, send to default gateway." Your machine only knows one next hop.
- **Your home router:** also simple — "send everything to ISP's router."
- **Backbone routers:** maintain massive routing tables (hundreds of thousands of entries) built by **BGP (Border Gateway Protocol)** — routers gossip with each other about which networks they can reach and how many hops away.

---

## 11. Connection Reuse

### The problem keep-alive solves

Without keep-alive (old HTTP/1.0):

```
GET /            → new socket, new handshake → response → SOCKET CLOSED
GET /styles.css  → new socket, new handshake → response → SOCKET CLOSED
GET /logo.png    → new socket, new handshake → response → SOCKET CLOSED
```

30 resources = 30 TCP handshakes. Slow.

With keep-alive (HTTP/1.1 default):

```
GET /            → same socket → response → socket stays open
GET /styles.css  → same socket → response → socket stays open
GET /logo.png    → same socket → response → socket stays open
```

One handshake, many requests. The `Connection: keep-alive` header tells the server not to close the socket after responding.

**But:** HTTP/1.1 is still **one request at a time per socket.** To fetch 6 resources in parallel, the browser opens 6 sockets (6 different ephemeral ports).

### HTTP/2 — Multiplexing

HTTP/2 solves both problems: **one socket, many simultaneous requests** via multiplexed streams. No new handshakes, no multiple ports for parallelism.

|  | HTTP/1.0 | HTTP/1.1 | HTTP/2 |
| --- | --- | --- | --- |
| Connections per page | One per resource | Reused (keep-alive) | Single connection |
| Parallel requests | One per connection | One per connection (need multiple connections) | Many via multiplexed streams |
| Handshakes | Per resource | Per connection | Once |

### Python equivalent

```python
# HTTP/1.0 style — new connection each time
requests.get("https://google.com/page1")
requests.get("https://google.com/page2")

# HTTP/1.1 style — reuse connection (keep-alive)
session = requests.Session()
session.get("https://google.com/page1")
session.get("https://google.com/page2")  # same TCP socket, no new handshake
```

---

## 12. What We Deliberately Skipped

### TLS / Encryption (HTTPS)

In reality, a **TLS handshake** happens between the TCP handshake and the HTTP request. The browser and server exchange encryption keys and verify certificates. After TLS, all data is encrypted. Port 443 (HTTPS) instead of 80 (HTTP). We skipped this to focus on the plain communication flow.

### NAT (Network Address Translation)

Your machine's IP (192.168.1.42) is a **private address** that doesn't exist on the public internet. Your router performs NAT: rewrites your source IP to the router's public IP (e.g., 103.42.7.89) and remembers the mapping. When Google's response comes back, the router reverses the translation. This is why many devices can share one public IP.

### DNS Resolution

Covered in previous notes. The chain: browser cache → OS resolver → Root server (.com?) → TLD server (google.com?) → Authoritative server (142.250.195.46). Uses UDP port 53.

---

## 13. Connection to Voice / Telephony

Everything in this document applies to voice communication too — the same internet infrastructure carries both web pages and voice calls. The key differences are only at the top two layers:

| Aspect | Web (HTTP) | Voice (RTP) |
| --- | --- | --- |
| Application protocol | HTTP | RTP (Real-time Transport Protocol) |
| Transport protocol | TCP | UDP |
| Reliability | Every byte must arrive, in order | Lost packets are gone forever |
| Why | Missing HTML byte = broken page | Late voice packet = worse than silence |
| Retransmission | Yes (ACKs, seq numbers) | No |
| Layers 1-3 (IP, Ethernet, Physical) | Identical | Identical |

The SIP protocol (used in telephony for call setup) parallels TCP's handshake:

- TCP: SYN → SYN-ACK → ACK (then data flows)
- SIP: INVITE → 200 OK → ACK (then audio flows via RTP)

Both are "let's agree to talk before we talk" — just for different purposes.

**In Ripples' architecture:**

- Old path (Twilio): WebSocket (TCP) carrying audio → reliable but adds latency
- New path (LiveKit/WebRTC): RTP over UDP → no retransmission, lower latency, small glitches tolerated

Same routers, same IP routing, same Ethernet hops. Different transport choice based on what the application needs.

---

## Quick Reference — The Complete Mental Model

```
YOU TYPE google.com AND PRESS ENTER
           │
           ▼
    ┌─────────────┐
    │   Browser    │  Knows: HTTP, URLs, rendering
    │  (user space)│  Does: socket(), connect(), write(), read()
    └──────┬──────┘
           │ system calls
           ▼
    ┌─────────────┐
    │   Kernel     │  Knows: TCP, IP, Ethernet, sockets, ports
    │              │  Does: handshake, segment, wrap, ACK, retransmit
    └──────┬──────┘
           │ transmit queue
           ▼
    ┌─────────────┐
    │    NIC       │  Knows: electrical signals
    │  (hardware)  │  Does: bits → voltage/light/radio
    └──────┬──────┘
           │ wire / radio
           ▼
    ┌─────────────┐
    │  Router 1   │  Knows: IP only (layers 1-3)
    │  (home)     │  Does: strip Ethernet, read IP, new Ethernet, forward
    └──────┬──────┘
           │
           ▼
    ┌─────────────┐
    │  10-20 more │  Same thing: read IP, forward, new Ethernet each hop
    │   routers   │  MAC changes at every hop. IP stays the same.
    └──────┬──────┘
           │
           ▼
    ┌─────────────┐
    │  Google NIC  │  Signals → frame → kernel
    ├─────────────┤
    │  Google      │  De-encapsulate → 5-tuple lookup → find your socket
    │  Kernel      │  Deliver to receive buffer → send ACK back
    ├─────────────┤
    │  Web Server  │  read() → parse HTTP → generate response → write()
    └──────┬──────┘
           │
           ▼
    RESPONSE TRAVELS BACK (same process, reversed)
           │
           ▼
    BROWSER: read() → parse HTML → fetch resources → render → PIXELS ON SCREEN
```

---

*These notes cover the complete end-to-end flow of internet communication: from keypress to pixels, from application layer to physical signals, from browser system calls to router-by-router forwarding. The foundation here applies to any internet communication — web, voice, video, or anything else.*
