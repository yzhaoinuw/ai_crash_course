# The KV Cache

The KV cache stores key/value tensors from previous tokens so the model does not
need to recompute the whole prompt every time it generates a new token.

Without KV cache:

```text
generate token 1 → process whole prompt
generate token 2 → process whole prompt + token 1 again
generate token 3 → process whole prompt + token 1 + token 2 again
```

That would be very slow.

With KV cache:

```text
process prompt once
store key/value states
for each new token:
    compute only the new token's states
    attend to cached previous states
```

This is one of the main reasons autoregressive decoding is fast enough to feel
interactive.

## Why it is safe to cache (your mental model is right)

A natural question: how do we know the cached states are still valid after we add
a new token? The answer is **causal masking**.

In a decoder-only model, every token only attends to itself and earlier tokens —
never to later ones. So token *i*'s key/value vectors at every layer depend only
on tokens `≤ i`. Appending a new token at the end cannot reach backward and change
anything before it.

That gives two facts, both of which match the intuition you described:

- The key/value vectors of already-cached tokens are **frozen** once computed.
  The "attention among the old tokens" was settled during prefill and is never
  recomputed.
- Each decode step does exactly one token's worth of new attention: the new token
  forms a single query and attends to **all** cached keys plus its own.

So the cache is not a lossy shortcut or an approximation. It stores precisely the
states that *provably cannot change*, which is why reusing them gives identical
results to recomputing from scratch.

One consequence worth remembering: the cache is only valid as long as the prefix
is **immutable**. If anything early changes — you edit the system prompt, trim an
old message, swap a token — every cached state after that point is invalid and
must be recomputed. Caches are tied to an exact prefix.

### "But don't the earlier words need to re-read the expanded context?"

This is the most common stumbling block, so it is worth stating directly: **no.**
The cache is **append-only** — each new token adds its own K/V, and nothing
already in the cache is ever updated. It is written once per token and never
rewritten.

The reason this is not a loss of accuracy comes from how we *read* versus how the
model works. When a human reads, a later word can change the meaning of an earlier
one ("the **bank** … of the river") — that is **bidirectional** reading.
Decoder-only models are deliberately **not** bidirectional: token *i* is only ever
allowed to predict token *i+1* from tokens `≤ i`, both during training and at
inference. So an earlier token "seeing" the new context would not be *more*
accurate — it would be a **different computation than the one the model was
trained to do.** Freezing the old K/V reproduces exactly the masked computation
from training; there is nothing to correct.

Context *does* still get integrated — but always at the **frontier**, never
backward. Each new token forms one query, attends over the whole cache, and the
result becomes its own cached entry for the next step. Information flows
new-reads-old, marching forward; it never flows back into the old tokens.

### Illustration: information flow and the append step

The cache is a per-layer list of K/V entries, one per token. Reading direction is
always **left-to-right** — a new token reads everything to its left, and nothing
reads it from the left:

```text
            cached (frozen)                      new
        ┌───────────────────────┐                │
KV:     [The] [cat] [sat] [on] [the]            [ mat ]   ← step's new token
                                                   │
query from "mat" ─── attends over ──▶ all 5 cached K/V + its own
                                                   │
                                                   ▼
                                          produces K/V for "mat"
                                                   │
                                                   ▼
KV:     [The] [cat] [sat] [on] [the] [mat]   ◀── appended, now frozen too
```

Notice what did *not* happen: none of the arrows point backward. `[The]…[the]`
were not recomputed; only `[mat]` was added. Step by step it is pure append:

```text
step 1   KV: [The]
step 2   KV: [The][cat]                 # appended [cat], read [The]
step 3   KV: [The][cat][sat]            # appended [sat], read [The][cat]
step 4   KV: [The][cat][sat][on]        # appended [on],  read [The][cat][sat]
...                                      # earlier entries never change
```

The arrow of information flow only ever points one way:

```text
[The] ──▶ [cat] ──▶ [sat] ──▶ [on] ──▶ [the] ──▶ [mat] ──▶ (next token)
  ▲                                                  │
  └──────────── never flows back ───────────────────┘
```

This is also why **bidirectional (BERT-style encoder) models cannot use a KV cache
this way**: there, every token attends to future tokens too, so adding a token
*would* change all the earlier ones and force a full recompute. Decoder-only
models give up that backward flow, and the append-only cache is the reward.

## So what is the catch? (compute for memory)

The speedup is not free. You are trading **recomputation for memory**: instead of
redoing the work each step, you keep every token's K/V tensors sitting in GPU
memory. That cost shows up in two ways.

**1. The cache is large and grows linearly.** Its size is roughly:

```text
2 (K and V) × n_layers × n_tokens × n_kv_heads × head_dim × bytes × batch_size
```

For long contexts or big batches this can rival or exceed the size of the model
weights themselves. It is the hard limit on how long a context — and how many
concurrent users — a given GPU can serve.

**2. Decode becomes memory-bandwidth bound, not compute bound.** Every single
decode step has to *read the entire cache back* from GPU memory to attend over it.
So per-token latency grows with context length: not because there is more math
(there is only one new query), but because there is more memory to stream each
step. A 100k-token context is slow to decode mainly because the cache is huge to
re-read, not because the arithmetic is hard.

So the trade is: you avoid O(n) recomputation per step, but you pay O(n) memory
*and* O(n) memory traffic per step instead. For autoregressive decoding that is
almost always a great deal — but it is why most of the hard engineering in LLM
serving (paged attention, grouped-query attention, quantized caches) is aimed
squarely at making the KV cache smaller and cheaper to read.

## Mental model: not rereading the whole book

The model processes the prompt once and caches useful internal states, then
generates new tokens efficiently.

## Why this matters for cost

The KV cache is also why long chats get expensive even when the output is short —
the cache grows with the conversation and consumes GPU memory. That tension is
covered in [Why long context is expensive](06-long-context-cost.md), and
"KV cache engineering" is one of the future modules in the
[roadmap](../README.md#future-modules-not-yet-built-out).
