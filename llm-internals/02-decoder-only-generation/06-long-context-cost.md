# Why Long Context Is Impressive But Not Free

Longer prompts cost more and take longer because the model has to process more
tokens.

In a vanilla transformer, attention cost scales roughly like:

```text
O(n²)
```

where `n` is the number of tokens.

That means longer context can become expensive fast.

Modern systems use optimizations such as:

- FlashAttention
- KV caching
- grouped-query attention
- sparse/sliding-window attention
- batching
- specialized GPU kernels
- memory-efficient serving infrastructure

But long context still increases latency and cost.

## Where this goes next

Two deeper threads branch off from here, both sketched as future modules in the
[roadmap](../README.md#future-modules-not-yet-built-out):

- **Attention scaling** — why attention is O(n²), FlashAttention, sparse and
  sliding-window attention, why million-token contexts are hard.
- **KV cache engineering** — GPU memory cost, batch size vs latency/throughput,
  paged attention and vLLM.

A separate consequence — that long context can also hurt *quality*, not just
cost — is covered in
[Module 04: long-context reliability](../04-memory-and-retrieval/03-long-context-reliability.md).
