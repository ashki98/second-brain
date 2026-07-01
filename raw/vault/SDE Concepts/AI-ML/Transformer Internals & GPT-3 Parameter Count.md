# Transformer Internals & the GPT-3 Parameter Count

How a transformer turns tokens into a next-token prediction, and where all 175B of GPT-3's parameters actually live. Foundation built on the 3Blue1Brown transformer series (GPT overview, Attention, MLP chapters).

## The end-to-end pipeline

```
text → token IDs → [embedding lookup] → +positional encoding
     → ( attention → MLP ) × n_layers     ← residual stream flows through
     → [unembedding] → probability over 50,257 tokens → sample next token
```

- **Tokenize**: text split into tokens (word-pieces) from a fixed vocab, each a single integer ID. GPT-3 `n_vocab = 50,257`.
- **Embedding lookup**: a plain lookup table maps each token ID → a vector of width `d_embed = 12,288`. It's **indexing, not computation** — the only reason it counts as params is the stored vectors are *learned*. Think hash table where the token ID *is* the array offset → O(1), no collisions.
- **Positional encoding**: order info added on top (raw embedding = *what* the token is, not *where*).
- **Layer stack**: `n_layers = 96` identical layers, each = attention block + MLP block, both **adding** into the residual stream.
- **Unembedding**: final vector projected back to a score over all 50,257 tokens → softmax → probabilities → sample.

## The residual stream (key mental model)

- The "river of meaning" is a per-token vector of width `d_embed` (12,288).
- Every block **reads from** and **adds back into** it — never replaced, only edited.
- This is why every projection matrix has 12,288 as one dimension — everything must match the stream's width.

## Attention block (tokens share information)

Three **learned** projection matrices turn each token's vector into three roles:

- **Query (Q)** = E × W_Q — "what am I looking for?"
- **Key (K)** = E × W_K — "what do I offer?"
- **Value (V)** = E × W_V — the information actually passed on

(Deliberate database/retrieval analogy: a query is matched against keys, matching keys return values.)

**How a pattern is computed:**
1. Current token's Query · every token's Key (dot product). Big dot = highly relevant.
2. Softmax-normalise per column → attention weights (sum to 1).
3. Output = weighted blend of all the Values.

3B1B "fluffy blue creature" grid: in the `creature` column, `fluffy`=0.42, `blue`=0.58 → creature pulls meaning from both adjectives.

**Causal masking** — a token may attend only to itself + earlier tokens (never future). Enforced by masking the upper triangle to zero → grid is **lower-triangular**. This is the property that makes the KV cache valid: a token's K and V depend only on it and what came before, so once computed they **never change** (immutable, append-only).

**Multi-head** — many heads run in parallel, each with its own W_Q/W_K/W_V, each specialising in a relationship type. GPT-3: **96 heads × 96 layers**.

**Q vs K/V asymmetry** (why it's the "KV" cache, not "QKV"): at generation, the *current* token's Query is compared against *all previous* tokens' Keys/Values. So K and V of every token are needed on every future step (cache them); Q of a token is used once, at its own step, then discarded.

## MLP block (each token processed alone)

Contrast with attention: attention *mixes* tokens, MLP *digests* each token independently (no cross-token interaction).

```
d_embed (12,288) --up-proj--> n_neurons (49,152) --[nonlinearity]--> --down-proj--> d_embed (12,288)
```

- **Up-projection** (`n_neurons × d_embed`): widens to 4× (`12,288 → 49,152`).
- **Nonlinearity** (ReLU/GELU): lets each hidden neuron fire or stay silent — without it, two stacked matrices collapse into one.
- **Down-projection** (`d_embed × n_neurons`): contracts back to stream width.

**Where facts live (3B1B thesis):** each of the ~49,152 hidden neurons can act as a fact/feature detector. Up-projection row = a "question" (neuron fires when a concept is present — e.g. a "Michael Jordan" neuron); down-projection column = an "answer" (when it fires, write an associated concept back — e.g. nudge toward "basketball"). An MLP ≈ a big learned set of *if-concept-then-concept* associations.

- **Caveat — superposition/polysemanticity:** real neurons aren't clean one-fact detectors; they fire for many unrelated features. The "one neuron, one fact" picture is a teaching simplification.

## The GPT-3 parameter table (175,181,291,520)

Dims: `n_vocab=50,257`, `d_embed=12,288`, `d_query=d_value=128`, `n_heads=96`, `n_layers=96`, `n_neurons=49,152`.

| Component | Formula | Count |
|---|---|---|
| Embedding | d_embed × n_vocab | 617,558,016 |
| Key | d_query × d_embed × n_heads × n_layers | 14,495,514,624 |
| Query | d_query × d_embed × n_heads × n_layers | 14,495,514,624 |
| Value | d_value × d_embed × n_heads × n_layers | 14,495,514,624 |
| Output | d_embed × d_value × n_heads × n_layers | 14,495,514,624 |
| Up-projection | n_neurons × d_embed × n_layers | 57,982,058,496 |
| Down-projection | d_embed × n_neurons × n_layers | 57,982,058,496 |
| Unembedding | n_vocab × d_embed | 617,558,016 |
| **Total** | | **175,181,291,520** |

**Patterns to notice:**
- **Embedding == Unembedding** (617M each) — same two numbers, reversed roles (ID→vector vs vector→distribution).
- **K/Q/V/Output equal** (~14.5B each) — same factors, different matrix orientation.
- **Up == Down projection** (~58B each), together **~116B = 66%** of the total. Most parameters — and most stored facts — live in the **MLPs**, not attention.

## Note

Every weight here (embedding table, W_Q/W_K/W_V, up/down projections) is **learned via backpropagation**: forward pass → loss → gradients (blame propagated backward) → nudge each weight. "Learned" everywhere in these notes means this.
