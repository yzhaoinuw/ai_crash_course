# Module 03 — Generation Quality and Decoding

Autoregressive generation conditions each token on the model's own previous
tokens. That is powerful, but it means an early mistake can snowball. This module
covers why long outputs drift and the levers — sampling, structure, tools — that
keep them coherent.

## Learning Objectives

By the end you should be able to:

- Explain exposure bias and error compounding in your own words.
- List the practical techniques that reduce long-output drift and say when each
  applies.
- Connect a decoding strategy (temperature, top-k, top-p, beam) to the
  probability distribution it operates on.

## Lessons

1. [Why wrong tokens derail long outputs](01-why-outputs-drift.md)
2. [How systems reduce drift](02-reducing-drift.md)

## Readings For This Module

### Hugging Face — Decoding Strategies in Large Language Models

Practical explanation of how models choose the next token.

Covers: greedy decoding, beam search, temperature, top-k sampling, top-p / nucleus
sampling.

This connects directly to the question:

```text
If one wrong token can throw off later output, how do we control generation?
```

```text
https://huggingface.co/blog/mlabonne/decoding-strategies
```

### AssemblyAI — Decoding Strategies: How LLMs Choose The Next Word

Readable companion article on decoding. Good for reinforcing the idea that the
model produces a probability distribution, and a decoding strategy turns that
distribution into actual text.

```text
https://www.assemblyai.com/blog/decoding-strategies-how-llms-choose-the-next-word
```

## Where this goes next

Going deeper on the decoding algorithms themselves — and **speculative decoding**,
where a small draft model accelerates a large one — is sketched in the
[roadmap's future modules](../README.md#future-modules-not-yet-built-out).
