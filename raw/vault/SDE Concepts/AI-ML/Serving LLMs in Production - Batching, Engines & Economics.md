# Serving LLMs in Production: Batching, Engines & Economics

How a model is run for many concurrent users at high throughput and low cost. The deployment layer on top of LLM Inference (KV cache + GPUs). GPU specs/prices drift — figures are early-2026.

## Batching — where throughput comes from

**The problem:** during decode, producing one token for one user reads the *entire* model's weights from VRAM (the slow, bandwidth-bound part) and then the thousands of cores do a tiny amount of work and go idle. Expensive cost paid, almost nothing gained. (Freight truck delivering one envelope.)

**The insight:** the weights you hauled out of VRAM are the same for every request. So process **many requests at once** — read weights once, have the idle cores compute the next token for *all* requests simultaneously. The expensive weight-read is **amortized** across the batch; the extra compute was basically free.

**Two things get read from VRAM per decode pass — and they split:**

| | Weights | KV caches |
|---|---|---|
| Copies | ONE, shared by all requests | ONE PER request (private) |
| Read | once, shared | each read separately |
| Cost vs batch | **amortizes** (÷ batch) ✓ | **does NOT amortize** — grows linearly ✗ |

- **DDIA echo:** weights = shared read-only data segment (amortizes across concurrency); KV caches = per-connection session state (grows with live connections). Same wall as connection pooling — cap concurrency to fit a fixed memory budget.

**Why bigger batch isn't infinitely better:**
1. Weight-read cost per request falls as 1/batch (1→2 halves it; 31→32 barely moves it) — diminishing returns. The un-amortizable KV-cache read becomes dominant → shifts from "weight-bound" to "KV-cache-bound."
2. Batch and KV caches compete for the **same fixed VRAM**. `total VRAM = weights (fixed) + Σ per-request KV cache`. More requests = more cache = ceiling on batch size.

**Static vs continuous batching:**
- **Static**: wait for a fixed group to ALL finish, then start the next. Wasteful — short requests finish early, their slot sits idle until the longest one is done.
- **Continuous** (in-flight): scheduler works at token level — the moment any request finishes, evict it and slot a waiting one in mid-flight. Keeps the GPU saturated. Biggest lever for throughput-per-dollar. (Used by vLLM, TensorRT-LLM.)
- It's a resource-pool scheduler (connection-pool / Redis consumer-group instinct): fixed-capacity pool, variable-length work arriving/departing, keep utilization high without oversubscribing. Continuous batching = avoid head-of-line blocking.

## Serving engine — what it actually is

The production program between your app and the raw model. The model is an inert pile of weights; the engine loads it into VRAM and serves many users at max throughput.

```
requests (HTTP) → [ SERVING ENGINE ] ⇄ GPU + VRAM → tokens stream back
                   ├ scheduler (continuous batching)
                   ├ KV-cache manager (PagedAttention)
                   ├ optimized GPU kernels (FlashAttention, fused matmuls)
                   └ API server + token streaming
```

- It implements every mechanism from these notes: holds weights resident, runs the batching scheduler, manages KV caches, holds hand-tuned kernels that do the matrix math far faster than naive PyTorch, exposes an (OpenAI-compatible) API, streams tokens.
- **DDIA framing:** a serving engine is to a model what a **database engine is to a data file**. You don't hand-parse files, you run Postgres (query planner + buffer pool + connection layer + execution engine). Weights = data file; engine = Postgres wrapped around it.
- **Is vLLM "just an HTTP server"?** No. It *contains* an optional OpenAI-compatible HTTP front end, but that's a thin, swappable shell. You can `import vllm` and call `LLM.generate()` in-process with no socket at all — the scheduler, PagedAttention, and kernels still run. Like calling Postgres "a TCP server" — there's a listener inside, but you've named the wrapper, not the engine. It's an **inference engine** with an optional HTTP adapter.
- Alternatives: **TensorRT-LLM** (NVIDIA's, fastest on their HW, less flexible), **SGLang** (strong on structured/agent workloads), **TGI** (Hugging Face). Same job; differ in optimization aggressiveness and HW/features.

**PagedAttention** (vLLM's core innovation): you don't know a response's length in advance, so reserving max-length KV space wastes VRAM. PagedAttention splits the cache into fixed-size **blocks** (pages), non-contiguous, allocated on demand — **OS virtual-memory paging applied to the KV cache**. Flip side: when VRAM is exhausted it swaps blocks to host memory and decode latency spikes (~30ms → 200ms+/token) — thrashing, same as OS page swapping.

## The VRAM budget & deployment math

```
total VRAM = model weights (fixed) + Σ per-request KV cache (grows with batch × context)
```

- **Weights** = params × bytes/param. 70B: 140 GB (FP16), 70 GB (FP8), 35 GB (INT4).
- **KV cache per request** ≈ `2 × layers × kv_heads × head_dim × seq_len × bytes`. The `2` = K and V. **Linear in context length AND batch size.**

**Worked example — Llama-70B on one 80 GB H100 (FP8):** weights ≈ 70 GB → only ~10 GB left for ALL KV caches.

| Context | KV per request | Requests that fit in ~10 GB |
|---|---|---|
| 128K | ~13 GB | < 1 (!) |
| 32K | ~3.2 GB | ~3 |
| 4K | ~0.4 GB | ~24 |

- **The central production tradeoff:** context length and batch size trade directly against each other inside the fixed VRAM wall. A workload fine at 8K can OOM at 32K — same model, same GPU.

## GPU landscape (early 2026)

| GPU | VRAM | Bandwidth | ~Cloud $/hr |
|---|---|---|---|
| H100 | 80 GB | 3.35 TB/s | ~$2.00 |
| H200 | 141 GB | 4.8 TB/s | ~$2.60 |
| B200 | ~180-192 GB | ~8 TB/s | ~$4-5 |

- H200 = H100's compute die + more/faster memory → pure inference win (esp. long context, up to ~3.4× H100). H100 needs 2-way tensor parallelism for 70B; H200 fits it on one GPU.
- B200 = new architecture, ~2× H200 bandwidth + FP4 → ~4× Llama-70B throughput, ~$0.17/M tokens vs ~$0.50 on H200.

## Metrics on every dashboard

- **TTFT** — prefill latency (prompt length). Interactive target: sub-second.
- **TPS/user** — decode speed; interactive ≈ 50 tok/s (faster than reading).
- **Throughput (tok/s/GPU)** — aggregate across the batch; divides into cost.
- **$/million tokens** — what finance cares about.

## Multi-GPU parallelism (when one isn't enough)

- **Tensor parallelism** — split each layer's matrices across GPUs; they work on the same token together, communicating every layer. Needs fast interconnect (NVLink). (= sharding *within* an operation.)
- **Pipeline parallelism** — different layers on different GPUs, assembly-line. (= partitioning *by stage*.)
- Fitting on ONE GPU is preferred — crossing GPUs adds per-token communication latency.
- **MoE wrinkle:** models like DeepSeek V3 (671B total) route only ~37B active per token — cheap to *compute*, but the whole expert table must live in VRAM → total *capacity* dominates.

## The whole picture (one line)

Quantize weights to fit and free room → fill leftover VRAM with as big a batch as latency allows → let context length and batch fight over that leftover → use vLLM to keep the batch saturated → read it out as $/token. The central knob: bigger batch = cheaper per token, until latency breaks or VRAM runs out.
