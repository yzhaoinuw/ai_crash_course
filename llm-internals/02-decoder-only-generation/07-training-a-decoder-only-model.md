# Training a Decoder-Only Model

Understanding training resolves several intuitions about how these models actually
work — and explains why inference works the way it does.

## What a Training Sample Looks Like

A training sample is not a sentence. It is a raw text chunk — however many tokens
fit into the model's context window.

The pipeline takes a large corpus (books, web pages, code), concatenates documents
together separated by an end-of-document token, then slices the result into
fixed-length chunks:

```text
# Conceptual data pipeline

document_stream = "<|endoftext|>The cat sat on the mat...<|endoftext|>The sky is blue..."

# Slice into context-length chunks (e.g., 2048 tokens each)
chunks = [
    document_stream[0    : 2048],   # may start mid-sentence, span multiple docs
    document_stream[2048 : 4096],
    document_stream[4096 : 6144],
    ...
]

# Each chunk becomes one training example
# input:   chunk[0 : N-1]   ← tokens the model sees
# labels:  chunk[1 : N]     ← tokens the model must predict (shifted by 1)
```

Sentence boundaries do not matter. A chunk may start mid-sentence, end
mid-sentence, and span multiple paragraphs. The `<|endoftext|>` separator teaches
the model that text after it is independent of text before it.

## One Prediction Per Position, All Positions at Once

At every position in the chunk, the model predicts the next token. All 2047
predictions in a 2048-token chunk are computed in a single forward pass — not
one by one.

```text
# Inference: one token per step (sequential loop)

step 1:  model sees "The"                → samples → "cat"
step 2:  model sees "The cat"            → samples → "sat"
step 3:  model sees "The cat sat"        → samples → "on"
# ...

# Training: all positions in one forward pass (parallel)

model sees "The cat sat on the mat"
  pos 0:  "The"              → predict "cat"    ✓ loss computed
  pos 1:  "The cat"          → predict "sat"    ✓ loss computed
  pos 2:  "The cat sat"      → predict "on"     ✓ loss computed
  pos 3:  "The cat sat on"   → predict "the"    ✓ loss computed
  pos 4:  "The cat sat on the" → predict "mat"  ✓ loss computed

# Result: 2047 prediction signals from one chunk — all at once
```

This is why Transformers train so much faster than recurrent networks, where
each position had to wait for the previous one to finish.

The training procedure is called **teacher forcing**: the model always receives
the true previous tokens as input, not its own (potentially wrong) predictions.
This keeps gradients clean and training stable.

## The Causal Mask Makes Parallelism Safe

If all positions are computed simultaneously, how does position 2 not cheat by
peeking at position 3?

The answer is the **causal mask**: a lower-triangular mask applied to the
attention scores before the softmax. Masked positions are set to −∞, which
becomes 0 after softmax — effectively zero attention weight.

```text
# Attention mask for a 5-token sequence
# Rows = query token (what is predicting), Columns = key token (what it can see)
# ✓ = allowed to attend,  · = masked out (−∞ → 0 after softmax)

              The   cat   sat   on    mat
         The [  ✓     ·     ·    ·     ·  ]   ← "The" sees only itself
         cat [  ✓     ✓     ·    ·     ·  ]   ← "cat" sees The + itself
         sat [  ✓     ✓     ✓    ·     ·  ]   ← "sat" sees The, cat + itself
          on [  ✓     ✓     ✓    ✓     ·  ]
         mat [  ✓     ✓     ✓    ✓     ✓  ]   ← "mat" sees everything before it
```

Position `sat` can only see `The`, `cat`, and itself. It cannot peek at `on` or
`mat`. This guarantees that each prediction is honest: the model never had access
to the token it was predicting.

This same mask is what makes the [KV cache](04-kv-cache.md) safe during
inference: because a token's key/value vectors depend only on tokens to its left,
those vectors never need to change when new tokens are appended.

## Cross-Entropy Loss Rewards Calibrated Distributions

The loss function is **cross-entropy** at every position. This has a non-obvious
implication: the model is not trained to output one "correct" token — it is
trained to output a probability distribution where the true next token gets
high probability.

```text
# The model sees: "The"
# Many tokens could plausibly follow:
#
#   "cat"   → 0.12
#   "dog"   → 0.09
#   "quick" → 0.08
#   "sky"   → 0.07
#   ...     → many small probabilities
#
# Cross-entropy only penalizes low probability on the TRUE token.
# It does not penalize high probability on other plausible completions.
#
# So the model learns: be uncertain when many continuations are valid,
# be confident when only one continuation is plausible.
```

With billions of training examples, each possible context is observed with many
different continuations in practice. The model converges toward the true empirical
distribution. There is no special mechanism needed — the math of cross-entropy
handles it automatically.

A consequence: models are naturally well-calibrated on common patterns and
uncertain on rare or ambiguous ones. The limited context at the start of a sentence
is not a weakness that needs to be patched — the model simply learns to spread
probability mass across plausible next tokens, which is the correct behavior.

## Batching: Same Sequence vs Different Sequences

A training **batch** contains multiple independent sequences:

```text
# A batch of 4 sequences, each of length 2048 tokens

batch = [
    chunk_from_wikipedia_article,    # sequence A
    chunk_from_python_code_repo,     # sequence B
    chunk_from_a_novel,              # sequence C
    chunk_from_news_article,         # sequence D
]

# All 2047 positions within sequence A are computed together (one forward pass).
# Sequences A, B, C, D are completely independent — no cross-sequence attention.
# Gradients are averaged across all positions in all sequences.
```

Key distinction:

- **Within one sequence**: all positions share the same forward pass and the same
  KV computation. This is efficient.
- **Across sequences in a batch**: completely independent. Gradient signals from
  a Wikipedia article and a Python file are averaged together in the same
  parameter update.

This is also why shuffling matters: if every batch contained chunks from the same
document, the model would overfit to local style rather than general language.

---

*Previous: [Why sequential decoding is still fast](05-why-decoding-is-fast.md)*
