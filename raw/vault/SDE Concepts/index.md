# SDE Concepts — Index

Claude's quick-reference TOC. Use this to locate notes before reading full files.
Last updated: 2026-06-24

---

## By File

### Hashing Internals/
| File | Key topics |
|---|---|
| Internal Structure of a Hash Table.md | Two-step pipeline, hash function, modulo, load factor, dynamic resizing, doubling, amortised O(1) |
| Conflict Resolution in Hash Tables with Chaining.md | Linked list per slot, prepend insert, BST/Red-Black upgrade, Java HashMap TREEIFY_THRESHOLD, cache locality |
| Conflict Resolution in Hash Tables with Open Addressing.md | Probing function, EMPTY/OCCUPIED/DELETED slot states, soft delete, tombstones, Python dict |
| Conflict Resolution- Linear, Quadratic and Double Hashing.md | Linear probing, primary clustering, quadratic probing, secondary clustering, double hashing, h2(k) constraints |
| Hash Table Resizing and Optimizations.md | Resize stages (alloc/rehash/dealloc), +1 growth O(n²) proof, doubling amortised proof, bitwise AND trick, shrink threshold 1/8 vs 1/4, thrashing |

### Python/
| File | Key topics |
|---|---|
| OS Scheduling, Process and Threads.md | Physical cores, SMT/HT, process vs thread, context switch, CFS, MLFQ, user-level vs kernel-level threads, 1:1 M:1 M:N, asyncio vs ThreadPool vs Multiprocessing, FastAPI async def vs def, Node.js V8+libuv |
| Memory Allocation and Management in Python.md | Variables as references, heap vs stack, reference counting, cyclic GC, immutability, small int cache, string interning, shallow/deep copy, weak references |
| GIL, Threads and Processes.md | GIL purpose, CPU-bound vs I/O-bound impact, GIL release during I/O, multiprocessing workaround, user vs kernel threads, green threads |
| Asyncio.md | Event loop, coroutines, await, create_task, gather, TaskGroup, abandoned task trap, to_thread, ProcessPoolExecutor, Semaphore |
| Node vs Fastapi and asyncio nuances.md | FastAPI vs Node.js, Django WSGI, FastAPI ASGI, GIL + multiple workers, WSGI vs ASGI comparison |
| Python and JS based Server concepts.md | WSGI servers (Gunicorn, uWSGI), ASGI servers (Uvicorn), Node.js as runtime, WebSocket vs SSE, framework vs runtime |
| WSGI and ASGI Deep Dive.md | WSGI callable (environ/start_response), ASGI callable (scope/receive/send), Gunicorn/Uvicorn are Python, dev vs prod servers, production server responsibilities (pre-fork, crash recovery, graceful restart), deployment patterns |
| Language Nuances.md | Python bytecode/PVM, bytecode vs machine code, __pycache__, PyPy, JIT deep dive (type specialization, deoptimization, warmup), V8 pipeline (Ignition/TurboFan/hidden classes), JS vs Python runtime comparison, frontend frameworks, system VM vs process VM, process VM landscape (JVM/CLR/BEAM/YARV/Zend), CPython compiler vs PVM as separate components, AST as universal compiler tool, compiler frontend vs backend, LLVM IR |
| OOP Questions.md | 4 pillars, _var/__var/@property, ABC/@abstractmethod, MRO/C3, mixins, duck typing, Protocol, class/static/instance methods, dunders, composition over inheritance |
| Packaging in python.md | venv/virtualenv, Poetry, pyproject.toml, lock files, wheel vs sdist |
| PANDAS_REFERENCE_GUIDE.md | pandas use cases, why over SQL, internal mechanisms (NumPy, SIMD, index hash tables, copy-on-write, groupby, column-oriented, dtype optimisation, Cython), performance numbers, decision framework |

### NodeJS/
| File | Key topics |
|---|---|
| Event Loop- Role.md | Single thread, call stack blocking, web APIs outside JS, task queue, event loop rule, setTimeout(0), debouncing |
| Event Loop- Deep dive.md | 6 phases (timers/pending/idle/poll/check/close), microtasks vs macrotasks, process.nextTick, execution order |
| Worker threads.md | libuv thread pool (no JS, 4 default), worker threads (own V8, own event loop, OS-level), postMessage, when to use each |

### Networking/
| File | Key topics |
|---|---|
| What happens when you visit a website.md | URL structure, DNS lookup chain, TCP handshake, TLS handshake, HTTP request/response, Keep-Alive, sub-resource fetching |
| Networking Fundamentals - Protocols, Layers, and W.md | OSI model, encapsulation, DNS resolution steps, TCP vs UDP, port ranges, HTTP/2 vs HTTP/3/QUIC, web server role |
| Networking Deep Dive — How the Internet Actually Works.md | 7-stage end-to-end flow, socket as kernel struct, TCP 3-way handshake, sequence numbers, ACKs, retransmit, MSS, 4 browser syscalls, routers layers 1-3 only |
| Status Code.md | 1xx/2xx/3xx/4xx/5xx categories, key codes (200/201/202/204/301/302/304/400/401/403/404/409/422/429/500/502/503/504), 301 vs 302, 400 vs 422 |

### Security/
| File | Key topics |
|---|---|
| CORS.md | SOP, what triggers CORS, simple vs preflight requests, CORS headers, backend fix, proxy approach, CORS vs CSRF |
| XSS Attacks.md | Stored/reflected/DOM-based XSS, Samy Worm, PayPal bug, input sanitisation, output escaping, CSP headers, DOMPurify |

### FastAPI/
| File | Key topics |
|---|---|
| Fast API (vs Django).md | Django vs Flask vs FastAPI comparison, Pydantic, async-first, yield for cleanup, when to choose each |
| FastAPI Dependency Injection (Depends) - Comprehen.md | Depends marker, execution flow (route→deps→endpoint), Query/Body/Header/Path params, magic trick, dependency chains, benefits (testability/DRY/security) |

### Database/
| File | Key topics |
|---|---|
| SQL vs NoSQL - Tradeoffs.md | CAP theorem, SQL vs NoSQL comparison, built-in scaling in NoSQL, when to choose |
| DB Indexing.md | B-tree index, when to index, when not to, heap/clustered/covering index, OLTP vs OLAP, column-oriented, bitmap encoding |
| DB Sharding.md | Horizontal partitioning, range/hash/directory sharding, consistent hashing, cross-shard join trade-offs |
| DB Deadlocks.md | Circular wait, wait-for graph, same-order access prevention, SELECT FOR UPDATE (pessimistic locking), retry logic |

### DDIA/
| File | Key topics |
|---|---|
| Part 1 Foundation of data systems.md | Reliability/Scalability/Maintainability, relational vs document model, normalisation, SQL declarative, OLTP vs OLAP, column-oriented + bitmap, Protobuf/Thrift/Avro |
| Replication.md | Single-leader, sync vs async, semi-sync, failover risks (split brain, lost writes, GitHub incident), replication lag anomalies (read-your-writes/monotonic/consistent prefix), multi-leader conflicts, leaderless/Dynamo, quorum w+r>n, sloppy quorum, hinted handoff, version vectors |
| Transactions.md | ACID (atomicity/consistency/isolation/durability), read committed, snapshot isolation, MVCC, lost updates, write skew, phantoms, serial execution, two-phase locking (2PL), predicate locks, index-range locks, serializable snapshot isolation (SSI) |

### Redis/
| File | Key topics |
|---|---|
| Redis Internals.md | IO multiplexing vs multi-threading, epoll, event loop steps, no locking overhead, exploiting network latency, AOF, TTL, eviction, transactions |
| Wire Protocols - RESP.md | Why not JSON, RESP types (+/:/$/*/-), bulk string binary safety, prefix length benefit, Redis command wire format |
| TCP Echo Server Implementation.md | Blocking listener.Accept(), blocking readCommand(), why Client 2 can't connect, motivates IO multiplexing |

### Scaling/
| File | Key topics |
|---|---|
| Load Balancer.md | LB purpose, round-robin/least-connections/IP-hash, health checks, auto-scaling, Layer 4 vs Layer 7 |
| API Throttling.md | Rate limiting, quota limiting, token bucket, where to enforce (gateway/app/Redis), DRF + slowapi |
| Handling 1M Requests.md | Managed API cost table, scaling tiers, write-behind pattern, memory vs disk numbers, PM2 cluster, Redis clustering |

### System Design/
| File | Key topics |
|---|---|
| System Design Primer.md | Back-of-envelope estimation, leader-follower read scaling, read-your-writes, trade-off table for interviews |
| Design URL Shortner.md | 301 vs 302, URL generation (counter/random/hash), Base62, Redis cache + read replicas, analytics write-behind |
| Design TikTok.md | Scale estimation, upload + feed architecture, Redis precache, CDN for videos, DB sharding, fan-out trade-off |
| Google Docs System Design – Interview Summary.md | Operational Transform, OT example, WebSocket stateful scaling, consistent hashing by doc_id, hybrid storage (S3+Cassandra), compaction |

### Frontend/
| File | Key topics |
|---|---|
| SSG SSR Edge.md | CSR/SSR/SSG/Edge comparison, Next.js getStaticProps/getServerSideProps, App Router server components |
| React Basic Learnings.md | UI=f(state), virtual DOM diffing, useState/useEffect, unidirectional flow, hooks reference |
| React Redux Redux-Saga.md | Layer stack, complete data flow, watcher+worker saga pattern, storage types comparison |
| FE Download.md | React vs Vanilla JS, React vs Next.js, TypeScript benefit, functional vs class components, ES6 features |

### AI-ML/
| File | Key topics |
|---|---|
| Confusion Matrix, L1, L2.md | Confusion matrix, TP/FP/TN/FN, accuracy/precision/recall/F1/specificity, L1(Lasso) vs L2(Ridge), lambda |
| LLMs- Fine Tuning, RAG.md | Fine-tuning (weights), RAG (offline index + online retrieval), CAG (context window cache), comparison table |
| AI Agents.md | Monolithic → Compound AI → Agents, reasoning/acting/memory, ReAct pattern, autonomy slider |
| MCP.md | Client-server design, resources/tools/prompts, LLM in client not server, vending machine analogy, JSON-RPC 2.0 |
| Langgraph Flow.md | Why LangGraph (vs while loop), StateGraph/nodes/edges/state, ToolNode, compile(), 6-level roadmap, streaming/checkpointing/human-in-the-loop |
| Transformer Internals & GPT-3 Parameter Count.md | Pipeline (embed→layers→unembed), residual stream, attention Q/K/V + causal masking + multi-head, MLP up/down projection + fact storage, full 175B param table, backprop note |
| Distillation & Quantization.md | Distillation (soft targets, classic vs data/task flavors, ToS), quantization (bucket+scale, precision ladder FP32→INT4, PTQ vs QAT, outliers/GPTQ/AWQ), the two as orthogonal levers |
| LLM Inference - Autoregressive Generation, KV Cache & GPUs.md | Autoregressive decode, KV cache (why K/V not Q, why not e+, per-layer caches, grid view), prefill vs decode (TTFT/TPS), CPU vs GPU, VRAM capacity vs bandwidth |
| Serving LLMs in Production - Batching, Engines & Economics.md | Batching (weights amortize, KV don't), static vs continuous batching, serving engine = DB engine analogy, vLLM/PagedAttention, VRAM budget math, GPU landscape, tensor/pipeline parallelism, MoE |

### Root level
| File | Key topics |
|---|---|
| CDN Concepts & AWS CloudFront Developer Flow.md | CDN PoPs, DNS vs Anycast routing, TLS termination at edge, CloudFront+S3 workflow, cache invalidation strategies |
| CQRS.md | Event Sourcing, sourcing/hydration/replay, snapshots, materialized views, CQRS architecture (command vs query side), event propagation, industry use cases (e-commerce, finance, ride-sharing, healthcare, SaaS, gaming), when to adopt |
| Docker Container vs Registry.md | Container vs VM, image/container/registry, Docker Compose |
| GRPC.md | (Empty file — covered by RPC.md context) gRPC vs REST, Protobuf binary, HTTP/2 multiplexing, 4 streaming modes |
| Github Gitlab CI CD.md | YAML pipelines, GitLab CI stages, Terraform plan+apply, secure variables, Jenkins vs GH vs GitLab |
| JWT.md | Header.payload.signature structure, Base64 encoding, RS256 signature verification, security pitfalls |
| Jenkins & CI CD Walkthrough.md | CI vs CD, Jenkins pipeline stages, 6-step setup, Jenkinsfile |
| Kafka Distributed Messaging System (Video Summary).md | Producer→broker→consumer, topics+partitions, ordering guarantees, consumer groups, at-least/at-most/exactly-once, batching, zero-copy |
| Kubernetes.md | Cluster architecture (master+workers), Pod/Service/Ingress/ConfigMap/Secret/Volume/Deployment/StatefulSet, self-healing, etcd |
| Load Balancer vs API Gateway.md | LB (where to send) vs API Gateway (what to do), architecture patterns (LB+GW, CDN+LB+GW, NLB→GW→ALB) |
| MVC.md | Model/View/Controller roles, View=JSON in APIs, Service Layer enhancement, framework naming map |
| MongoBleed.md | CVE-2025-14847, unauthenticated memory disclosure, zlib buffer over-read, risk levels, patch/workaround |
| Performance Improvement Concepts.md | Connection pooling, DB indexing for performance, profiling types (CPU/memory/DB/I/O), performance checklist |
| RPC.md | RPC concept (local vs remote), Lambda as RPC, Lambda vs gRPC comparison table |
| SOLID.md | SRP/OCP/LSP/ISP/DIP using PaymentProcessor example, composition over inheritance |
| Software Is Changing (Again) by Andrej Karpathy.md | Software 1.0/2.0/3.0, LLMs as utilities/fabs/OS, LLM psychology (superpowers + deficits), 4 properties of LLM apps, vibe coding |

---

## Topic Index (quick lookup)

### Concurrency & Async
- Python GIL → `Python/GIL, Threads and Processes.md`
- Python asyncio → `Python/Asyncio.md`
- Python threads/processes → `Python/OS Scheduling, Process and Threads.md`
- FastAPI vs Django concurrency → `Python/Node vs Fastapi and asyncio nuances.md`
- WSGI vs ASGI → `Python/Python and JS based Server concepts.md`
- Node.js event loop (role) → `NodeJS/Event Loop- Role.md`
- Node.js event loop (phases) → `NodeJS/Event Loop- Deep dive.md`
- Node.js worker threads → `NodeJS/Worker threads.md`

### Data Structures
- Hash table internals → `Hashing Internals/Internal Structure of a Hash Table.md`
- Hash table chaining → `Hashing Internals/Conflict Resolution in Hash Tables with Chaining.md`
- Hash table open addressing → `Hashing Internals/Conflict Resolution in Hash Tables with Open Addressing.md`
- Hash table probing strategies → `Hashing Internals/Conflict Resolution- Linear, Quadratic and Double Hashing.md`
- Hash table resizing + bitwise trick → `Hashing Internals/Hash Table Resizing and Optimizations.md`

### Networking
- URL to browser journey → `Networking/What happens when you visit a website.md`
- OSI model, TCP/UDP, DNS, ports → `Networking/Networking Fundamentals - Protocols, Layers, and W.md`
- Sockets, TCP internals, routers → `Networking/Networking Deep Dive — How the Internet Actually Works.md`
- HTTP status codes → `Networking/Status Code.md`
- CORS, SOP, preflight → `Security/CORS.md`

### Databases
- SQL vs NoSQL, CAP theorem → `Database/SQL vs NoSQL - Tradeoffs.md`
- Indexing, OLTP vs OLAP → `Database/DB Indexing.md`
- Sharding strategies → `Database/DB Sharding.md`
- Deadlocks, pessimistic locking → `Database/DB Deadlocks.md`
- DDIA foundations (data models, encoding) → `DDIA/Part 1 Foundation of data systems.md`
- DDIA replication (leader/leaderless/quorum) → `DDIA/Replication.md`
- Redis IO multiplexing → `Redis/Redis Internals.md`
- Redis RESP protocol → `Redis/Wire Protocols - RESP.md`
- TCP blocking problem → `Redis/TCP Echo Server Implementation.md`

### Distributed Systems & Scaling
- Load balancer algorithms → `Scaling/Load Balancer.md`
- LB vs API Gateway → `Load Balancer vs API Gateway.md`
- Rate limiting / throttling → `Scaling/API Throttling.md`
- CDN, CloudFront → `CDN Concepts & AWS CloudFront Developer Flow.md`
- Kafka messaging → `Kafka Distributed Messaging System (Video Summary).md`
- 1M RPS architecture → `Scaling/Handling 1M Requests.md`
- Connection pooling, profiling → `Performance Improvement Concepts.md`

### System Design
- Estimation, leader-follower reads → `System Design/System Design Primer.md`
- URL shortener design → `System Design/Design URL Shortner.md`
- TikTok design → `System Design/Design TikTok.md`
- Google Docs / Operational Transform → `System Design/Google Docs System Design – Interview Summary.md`

### Backend Patterns
- MVC, Service Layer → `MVC.md`
- SOLID principles → `SOLID.md`
- RPC concept → `RPC.md`
- gRPC, Protobuf, HTTP/2 → `GRPC.md`
- JWT structure + security → `JWT.md`
- CQRS + Event Sourcing → `CQRS.md`
- FastAPI vs Django → `FastAPI/Fast API (vs Django).md`
- FastAPI Depends, DI → `FastAPI/FastAPI Dependency Injection (Depends) - Comprehen.md`

### DevOps
- Docker containers → `Docker Container vs Registry.md`
- Jenkins CI/CD → `Jenkins & CI CD Walkthrough.md`
- GitHub/GitLab CI/CD, Terraform → `Github Gitlab CI CD.md`
- Kubernetes architecture → `Kubernetes.md`

### Frontend
- SSG/SSR/Edge/CSR → `Frontend/SSG SSR Edge.md`
- React basics, hooks, virtual DOM → `Frontend/React Basic Learnings.md`
- Redux + Redux-Saga → `Frontend/React Redux Redux-Saga.md`
- Next.js, TypeScript, ES6 → `Frontend/FE Download.md`

### Security
- XSS types + prevention → `Security/XSS Attacks.md`
- CORS, SOP, preflight → `Security/CORS.md`
- MongoBleed CVE-2025-14847 → `MongoBleed.md`

### AI / LLMs
- Confusion matrix, L1/L2 → `AI-ML/Confusion Matrix, L1, L2.md`
- RAG vs fine-tuning vs CAG → `AI-ML/LLMs- Fine Tuning, RAG.md`
- AI Agents, ReAct → `AI-ML/AI Agents.md`
- MCP protocol → `AI-ML/MCP.md`
- LangGraph → `AI-ML/Langgraph Flow.md`
- Software 1.0/2.0/3.0 (Karpathy) → `Software Is Changing (Again) by Andrej Karpathy.md`
- Transformer internals, attention/MLP, GPT-3 175B param count → `AI-ML/Transformer Internals & GPT-3 Parameter Count.md`
- Distillation (soft targets, flavors) + quantization (precision ladder, PTQ/QAT) → `AI-ML/Distillation & Quantization.md`
- Autoregressive decode, KV cache, prefill vs decode, GPUs/VRAM → `AI-ML/LLM Inference - Autoregressive Generation, KV Cache & GPUs.md`
- Batching, continuous batching, PagedAttention, vLLM, VRAM math, GPU landscape → `AI-ML/Serving LLMs in Production - Batching, Engines & Economics.md`

### Python Specifics
- Memory management, reference counting → `Python/Memory Allocation and Management in Python.md`
- OOP, MRO, dunders, composition → `Python/OOP Questions.md`
- Language nuances, bytecode, __pycache__, PyPy, JIT, V8 pipeline, hidden classes → `Python/Language Nuances.md`
- Packaging, Poetry, lock files → `Python/Packaging in python.md`
- Pandas internals → `Python/PANDAS_REFERENCE_GUIDE.md`
