# Prefill vs Decode

LLM inference has two major phases.

## A. Prefill phase

The model processes the input prompt.

During prefill:

- All prompt tokens are known.
- The model computes hidden states and attention information.
- It builds the KV cache.
- It may compute logits for every position, but those logits are usually ignored
  except for the final position.
- This phase can be parallelized across the prompt tokens using causal masking.

Even though the model is "decoder-only," it can process the prompt in parallel
because the full prompt is already available.

## B. Decode phase

The model generates the response one token at a time.

During decode:

```text
current context → predict next token
append token
current context + new token → predict next token
append token
...
```

This is sequential at the token level.

But each token-generation step uses massively parallel GPU computation internally.

## Mental model: prefill as reading, decode as writing

This is not architecturally exact, but it is a useful intuition. The
[KV cache](04-kv-cache.md) is what carries information from the reading phase into
the writing phase without re-reading.
