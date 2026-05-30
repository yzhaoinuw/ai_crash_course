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

## Mental model: not rereading the whole book

The model processes the prompt once and caches useful internal states, then
generates new tokens efficiently.

## Why this matters for cost

The KV cache is also why long chats get expensive even when the output is short —
the cache grows with the conversation and consumes GPU memory. That tension is
covered in [Why long context is expensive](06-long-context-cost.md), and
"KV cache engineering" is one of the future modules in the
[roadmap](../README.md#future-modules-not-yet-built-out).
