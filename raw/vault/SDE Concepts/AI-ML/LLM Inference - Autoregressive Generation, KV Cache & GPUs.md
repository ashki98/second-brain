# LLM Inference: Autoregressive Generation, the KV Cache & GPUs

How a trained model actually *runs* to produce tokens — and the GPU/VRAM facts that explain why it's engineered the way it is. This is the bridge from "what the model is" to "how we serve it."

## Autoregressive generation

An LLM generates **one token at a time**, feeding the entire sequence so far back through the model each step. A 100-token answer = 100 forward passes.

```
pass 1:  [The]            → predicts "cat"
pass 2:  [The cat]        → predicts "sat"
pass 3:  [The cat sat]    → predicts "on"
...
```

**The waste this creates:** to process `[The cat sat]`, attention needs the K and V vectors of `The`, `cat`, `sat`. But `The`'s and `cat`'s K/V were already computed in earlier passes — and they **never change** (`The`'s Key is the same no matter what follows, thanks to causal masking). Recomputing them every pass is pure waste, and it gets quadratically worse as the sequence grows.

## The KV cache

**Cache each token's K and V once computed, reuse on every later step.** After processing a token, stash its Key and Value. Next pass, only compute K/V for the *one new* token; read all previous from cache.

```
pass 1  [The]          compute K,V for "The"
pass 2  [The cat]      "The" from cache, compute K,V for "cat"
pass 3  [The cat sat]  "The","cat" from cache, compute K,V for "sat"
        cache grows by ONE entry per step
```

- It's a space-time tradeoff: spend memory to avoid recomputation (memoization / materialized view instinct).
- **DDIA echo:** cache entries are **immutable once written** (causal masking guarantees a token's K/V never change) → like an append-only log. No invalidation problem → cheap and safe.

**Why K and V but not Q** (it's the *KV* cache, not QKV): the current token's Query is compared against *all previous* Keys/Values. So K,V of every token are needed on every future step → cache. But Q of a token is used **once**, at its own step, then never read again → compute fresh, discard. Old Queries are dead weight.

**Why not just cache the layer output `e+`** (a sharp wrong turn worth recording): the post-attention output `e2+` for `cat` *is* stable and doesn't change — true. But attention for a new token doesn't read other tokens' *outputs*; it reads their **K and V**, which are *projections computed before* the attention blend (upstream), while `e+` is *downstream*. So `e2+` isn't `cat`'s Key or Value — caching it gives attention nothing useful; you'd still have to re-project. **A finished token contributes to future tokens only through its K and V, never through its post-attention output** — so that's the only thing worth caching.

**Per-layer caches:** each layer has its **own** W_K/W_V, so a 2-layer model keeps **two separate KV caches**. `forest` arriving at layer 2's attention needs the others' *layer-2* K/V (different vectors from layer 1). The new token flows forward through the stack; at each layer it reaches back into *that layer's* cache. Meaning of past tokens lives **in the caches**, not absorbed into the newest token's vector.

**The attention grid view:** each column = one token's Query scored against all Keys (rows). Causal masking → lower-triangular. Per decode pass, only **one new column** appears (new Q vs all cached Keys); every Key it needs is already cached. So per-pass cost is **linear** (new Q against N cached Keys), not the **quadratic** cost of rebuilding the whole triangle. That collapse is the entire payoff.

## Prefill vs decode (two phases, different bottlenecks)

| Phase | What happens | Bound by | Sets metric |
|---|---|---|---|
| **Prefill** | Whole prompt processed in ONE parallel pass; fills KV cache in bulk; emits first token | **Compute** | TTFT |
| **Decode** | Generate one token per pass, sequentially (can't parallelize — token N+1 needs token N) | **Memory bandwidth** | TPS |

- **TTFT (Time To First Token)** = prefill latency, dominated by prompt length. Governs perceived responsiveness ("is it working?").
- **TPS (Tokens Per Second)** = decode speed. Governs how long a long answer takes to stream.
- They're separate because you can be fast at one, slow at the other (huge prompt → bad TTFT even if TPS is great). Chatbot lives on TTFT; batch summarization cares about TPS.
- The cache bridges them: prefill fills it in bulk, decode extends it one column per step.

## GPUs & VRAM (why all the above is shaped this way)

**CPU vs GPU:**
- CPU: a few powerful cores, great at complex sequential/branchy logic ("one genius, fast").
- GPU: thousands of weak simple cores doing the **same op on different data** in parallel ("ten thousand interns in lockstep").
- A transformer is matrix multiplications (every row of the GPT-3 param table). Matrix multiply = "same multiply-add on thousands of numbers" → exactly the GPU's shape. That's why GPUs run LLMs hundreds of× faster.

**VRAM = the GPU's own dedicated memory**, holding weights, activations, KV cache. Two distinct properties:

- **Capacity** (GB) — a **hard wall**. 70B at FP16 = 140 GB; if the GPU has 80 GB, it physically won't load. First question in self-hosting. Quantization shrinks weights to fit under it.
- **Bandwidth** (bytes/sec to cores) — moving data from VRAM to cores is *slower* than the cores compute, so cores often sit **idle waiting** = "memory-bandwidth-bound." This is decode's bottleneck.

**How earlier claims trace to these two:**
- "Quantization helps it fit" → attacks **capacity**.
- "Decode is bandwidth-bound" → reads every weight from VRAM per token → limited by **bandwidth**, cores starved.
- "Quantization ~doubles decode throughput" → fewer bytes/weight → less to move → starved cores fed faster.
- "KV cache is the real memory bottleneck" → competes with weights for the same **capacity**.

- **DDIA echo:** capacity-vs-bandwidth = disk *size* vs disk *throughput* — one decides what fits, the other how fast you scan it. Twist: VRAM is the *only* tier the cores use directly, so "does it fit" is unforgiving (can't spill to a slower tier without big penalty).

**One-line anchor:** a GPU is thousands of simple cores doing matrix math in parallel, fed by VRAM whose *capacity* decides what fits and whose *bandwidth* often decides how fast you go.
