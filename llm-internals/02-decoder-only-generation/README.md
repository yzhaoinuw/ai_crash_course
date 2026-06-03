# Module 02 — How a Decoder-Only LLM Generates

The core of a ChatGPT-style system is a stateless decoder-only transformer that
predicts one token at a time. This module opens up that machine: how it reads the
prompt, how it writes the answer, and the engineering tricks that make it fast.

## Learning Objectives

By the end you should be able to:

- Explain what "stateless" and "decoder-only" actually mean.
- Distinguish the **prefill** phase from the **decode** phase.
- Describe what the KV cache stores and why it makes generation fast.
- Explain why long context is expensive even with these optimizations.
- Describe what a single training sample looks like and why next-token prediction
  trains all positions in parallel.

## Lessons

1. [Stateless next-token prediction](01-stateless-next-token.md)
2. [Decoder-only models](02-decoder-only-models.md)
3. [Prefill vs decode](03-prefill-vs-decode.md)
4. [The KV cache](04-kv-cache.md)
5. [Why sequential decoding is still fast](05-why-decoding-is-fast.md)
6. [Why long context is expensive](06-long-context-cost.md)
7. [Training a decoder-only model](07-training-a-decoder-only-model.md)

## Readings For This Module

### Andrej Karpathy — Let's Build GPT From Scratch

The best hands-on walkthrough of how a decoder-only GPT is trained. Karpathy
codes the entire thing — data loading, causal masking, training loop, generation —
from scratch in PyTorch while explaining every step.

Best for: understanding training samples, teacher forcing, the causal mask in
code, and how inference emerges from the same machinery as training.

```text
https://www.youtube.com/watch?v=kCc8FmEb1nY
```

### Lilian Weng — Generalized Language Models

Covers the evolution of language modeling from n-grams through ELMo, GPT, BERT,
and beyond. Good for understanding how next-token prediction became the dominant
pretraining objective and why it generalizes so well.

Best for: historical context, comparing encoder-only vs decoder-only pretraining
objectives, understanding why "predict the next token" scales so well.

```text
https://lilianweng.github.io/posts/2019-01-31-lm/
```

### Sebastian Raschka — Build a Large Language Model (From Scratch) — online book

Detailed, code-first walkthrough of building and training a GPT-style model.
Raschka explains data preparation, tokenization, the training loop, and
finetuning in clear prose with runnable notebooks.

Best for: a thorough code-level understanding of pretraining, the data pipeline,
cross-entropy loss in context, and how base models become chat models.

```text
https://www.manning.com/books/build-a-large-language-model-from-scratch
```
(free chapters available; full book is paid)

---

### Jay Alammar — The Illustrated Transformer

Classic visual introduction to the Transformer architecture. Even though it
focuses on the original encoder-decoder Transformer, the attention diagrams are
extremely helpful.

Best for: attention intuition, query/key/value concepts, multi-head attention,
encoder vs decoder basics.

```text
https://jalammar.github.io/illustrated-transformer/
```

### Jay Alammar — The Illustrated GPT-2

Closer to ChatGPT-style decoder-only models.

Best for: GPT-style generation, decoder-only architecture, how text prediction
works, how attention is used in language generation.

```text
https://jalammar.github.io/illustrated-gpt2/
```

### 3Blue1Brown — Attention in Transformers, Step by Step

Probably the best visual explanation of attention.

Best for: why attention works, queries/keys/values, how token meaning changes with
context, an intuitive geometric view of attention.

```text
https://www.3blue1brown.com/lessons/attention
```

### Hugging Face — KV Caching Explained

Short, approachable explanation of why generation does not recompute the whole
prompt every token.

Key idea:

```text
Process the prompt once → store key/value tensors → reuse them during token-by-token generation
```

```text
https://huggingface.co/blog/not-lain/kv-caching
```

### Sebastian Raschka — Understanding and Coding the KV Cache in LLMs from Scratch

More technical but very useful.

Best for: seeing KV cache in code, understanding the speed/memory tradeoff,
connecting theory to implementation.

```text
https://magazine.sebastianraschka.com/p/coding-the-kv-cache-in-llms
```

### Daily Dose of Data Science — KV Caching in LLMs, Explained Visually

Visual explanation of KV cache. Good for building intuition before reading
implementation-level material.

```text
https://blog.dailydoseofds.com/p/kv-caching-in-llms-explained-visually
```

### Lilian Weng — The Transformer Family v2.0

Higher-density but excellent.

Best for: encoder-only vs decoder-only vs encoder-decoder models, architecture
variants, positional encoding, Transformer design evolution.

```text
https://lilianweng.github.io/posts/2023-01-27-the-transformer-family-v2/
```

### Harvard NLP — The Annotated Transformer

Code-heavy. Read this when you want to see a Transformer implemented line by line.

Best for: implementation details, PyTorch-style architecture, understanding the
original Transformer from code.

```text
https://nlp.seas.harvard.edu/annotated-transformer/
```
