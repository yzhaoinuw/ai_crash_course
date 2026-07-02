# LLM Internals — Study Roadmap

A self-paced curriculum for understanding how ChatGPT / decoder-only LLMs handle
context, memory, and generation. It is built bottom-up: each module assumes the
one before it. Topics are kept bite-sized so a single sitting can usually finish
one lesson.

This roadmap was distilled from a spontaneous Q&A (originally a single file,
`context-memory-and-generation.md`) and reorganized into ordered modules. The
built-out content ends where the deepest *answered* question sat — the
production-system / drift-control level. Everything past that is sketched under
[Future Modules](#future-modules-not-yet-built-out) so the path forward is clear.

## How To Use This

1. Read modules in order, `01 → 05`.
2. Each module folder has its own `README.md` with learning objectives and the
   reading list for that module.
3. Lessons inside a module are numbered; read them in order too.
4. The original prose has been preserved — these docs are a reorganization, not a
   rewrite.

## The Roadmap

### [01 — Tokens and the Context Window](01-tokens-and-context/)

The unit the model actually counts, and the budget it lives inside.

- Tokens vs words (subword tokenization, morphology)
- Multilingual tokenization (Chinese, mixed-script)
- The context window (input + output budget)
- How much chat history fits before compression

### [02 — How a Decoder-Only LLM Generates](02-decoder-only-generation/)

The core machine: a stateless next-token predictor and the loop it runs.

- Stateless next-token prediction
- Decoder-only models (causal attention, "decoding" the prompt)
- Prefill vs decode
- The KV cache
- Why sequential decoding is still fast
- Why long context is expensive (O(n²) attention)
- Prompt caching across turns (prefix reuse; why permission prompts cost momentum, not tokens)
- Training a decoder-only model (teacher forcing, the causal mask)

### [03 — Generation Quality and Decoding](03-generation-quality-and-decoding/)

Why long outputs go wrong, and the levers that keep them on track.

- Why wrong tokens derail long outputs (exposure bias, error compounding)
- How systems reduce drift (sampling, structure, chunking, self-review, tools)

### [04 — Memory, Context Management, and Retrieval](04-memory-and-retrieval/)

How "memory" is faked from outside the model.

- Memory vs context (the four different things people call "memory")
- RAG, embeddings, and vector databases
- Long-context reliability (lost in the middle)

### [05 — Assistant Behavior and System Architecture](05-assistant-behavior-and-architecture/)

Why a naked model behaves like a helpful, named assistant.

- Identity and instruction-tuned behavior
- The layers of a deployed assistant
- Mental models and key vocabulary

## Future Modules (not yet built out)

These are the rabbit holes flagged during the original Q&A but not yet explored
in depth. They are the natural continuation once Modules 01–05 feel solid.

- **Transformer architecture internals** — query/key/value vectors, why attention
  uses dot products, multi-head attention, residual connections + layer norm,
  encoder-only vs decoder-only vs encoder-decoder.
- **Attention scaling** — why vanilla attention is O(n²), FlashAttention, sparse
  and sliding-window attention, why million-token contexts are hard.
- **KV cache engineering** — GPU memory cost, batch size vs latency/throughput,
  paged attention and vLLM.
- **Decoding strategies (deep)** — greedy, temperature, top-k, top-p, beam,
  contrastive decoding.
- **Speculative decoding** — draft-model acceleration and verification.
- **Agent architecture** — when to retrieve, when to call tools, model vs
  assistant vs agent, planner/executor/critic/memory modules.
- **Memory systems (deep)** — how memories are selected/stored, avoiding
  irrelevant or stale retrievals.
- **Hallucination and verification** — why fluent ≠ true, citations, tool-backed
  math/code, self-consistency.
- **Alignment (deep)** — instruction tuning, RLHF, constitutional AI, self-model
  vs role-conditioned behavior.
- **Non-autoregressive alternatives** — diffusion language models, revising
  earlier tokens.
- **Practical prompt engineering** — anchoring long responses, reducing drift,
  chunking, self-verification without overtrust.

## Consolidated Bookmark List

Every reading also appears in the module it belongs to. This is the flat list for
quick copy-paste.

```text
OpenAI tokens:
https://help.openai.com/en/articles/4936856-what-are-tokens-and-how-to-count-them

OpenAI tokenizer:
https://platform.openai.com/tokenizer

The Illustrated Transformer:
https://jalammar.github.io/illustrated-transformer/

The Illustrated GPT-2:
https://jalammar.github.io/illustrated-gpt2/

3Blue1Brown attention:
https://www.3blue1brown.com/lessons/attention

Hugging Face decoding strategies:
https://huggingface.co/blog/mlabonne/decoding-strategies

AssemblyAI decoding strategies:
https://www.assemblyai.com/blog/decoding-strategies-how-llms-choose-the-next-word

Hugging Face KV caching:
https://huggingface.co/blog/not-lain/kv-caching

Sebastian Raschka KV cache:
https://magazine.sebastianraschka.com/p/coding-the-kv-cache-in-llms

Daily Dose of Data Science KV cache:
https://blog.dailydoseofds.com/p/kv-caching-in-llms-explained-visually

Anthropic prompt caching (API docs):
https://platform.claude.com/docs/en/build-with-claude/prompt-caching

Claude Code prompt caching:
https://code.claude.com/docs/en/prompt-caching

Lessons from building Claude Code (prompt caching is everything):
https://claude.com/blog/lessons-from-building-claude-code-prompt-caching-is-everything

OpenAI prompt caching:
https://platform.openai.com/docs/guides/prompt-caching

The Illustrated Word2vec:
https://jalammar.github.io/illustrated-word2vec/

Vector database overview:
https://www.itpro.com/technology/big-data/what-is-a-vector-database

Lost in the Middle summary:
https://arize.com/blog/lost-in-the-middle-how-language-models-use-long-contexts-paper-reading/

Lost in the Middle paper:
https://arxiv.org/abs/2307.03172

Lilian Weng Transformer Family:
https://lilianweng.github.io/posts/2023-01-27-the-transformer-family-v2/

Annotated Transformer:
https://nlp.seas.harvard.edu/annotated-transformer/
```

## One-Paragraph Summary Of The Whole Curriculum

A ChatGPT-like assistant is best understood as a stateless decoder-only
transformer wrapped inside a larger product system. The core model predicts the
next token from the context it is given. It does not inherently remember prior
chats, know its identity, or search files by itself. Those behaviors come from
system prompts, instruction tuning, memory/retrieval systems, context management,
and tools. The model processes the prompt during a parallelizable prefill phase,
builds a KV cache, then generates output sequentially during decode. Long
contexts and long outputs are powerful but expensive and error-prone, so
production systems use summarization, retrieval, chunking, structured prompting,
and tool verification to keep responses coherent and useful.

## Keyword Index

Alphabetical lookup for this chapter. Each keyword points to the lesson that
**owns** it (the canonical treatment), not every file that mentions it. For the
guided top-down view use [The Roadmap](#the-roadmap) above; for the cross-chapter
map see the [master topic index](../README.md#topic-index) at the repo root.

- **append-only cache** → [KV cache](02-decoder-only-generation/04-kv-cache.md)
- **batching (training: within- vs cross-sequence)** → [Training](02-decoder-only-generation/08-training-a-decoder-only-model.md)
- **beam search / multiple candidates** → [Reducing drift](03-generation-quality-and-decoding/02-reducing-drift.md)
- **cache read vs write cost** → [Prompt caching](02-decoder-only-generation/07-prompt-caching-and-turn-cost.md)
- **causal mask / causal attention** → [Decoder-only models](02-decoder-only-generation/02-decoder-only-models.md) (defined), [KV cache](02-decoder-only-generation/04-kv-cache.md) (why caching is safe), [Training](02-decoder-only-generation/08-training-a-decoder-only-model.md) (parallelism)
- **chunking (long output)** → [Reducing drift](03-generation-quality-and-decoding/02-reducing-drift.md)
- **context window / token budget** → [The context window](01-tokens-and-context/03-the-context-window.md)
- **continuation engine** → [Decoder-only models](02-decoder-only-generation/02-decoder-only-models.md)
- **cross-entropy loss / calibrated distributions** → [Training](02-decoder-only-generation/08-training-a-decoder-only-model.md)
- **decode phase** → [Prefill vs decode](02-decoder-only-generation/03-prefill-vs-decode.md)
- **decoder-only architecture** → [Decoder-only models](02-decoder-only-generation/02-decoder-only-models.md)
- **embeddings** → [RAG and embeddings](04-memory-and-retrieval/02-rag-and-embeddings.md)
- **exposure bias / error compounding** → [Why outputs drift](03-generation-quality-and-decoding/01-why-outputs-drift.md)
- **how much chat history fits** → [How much chat fits](01-tokens-and-context/04-how-much-chat-fits.md)
- **identity / self-model** → [Identity and instruction tuning](05-assistant-behavior-and-architecture/01-identity-and-instruction-tuning.md)
- **instruction tuning / answering vs continuing** → [Identity and instruction tuning](05-assistant-behavior-and-architecture/01-identity-and-instruction-tuning.md)
- **KV cache** → [The KV cache](02-decoder-only-generation/04-kv-cache.md)
- **layers of a deployed assistant** → [Layers of an assistant](05-assistant-behavior-and-architecture/02-layers-of-an-assistant.md)
- **long-context cost / O(n²) attention** → [Long context cost](02-decoder-only-generation/06-long-context-cost.md)
- **lost in the middle** → [Long-context reliability](04-memory-and-retrieval/03-long-context-reliability.md)
- **memory vs context (four meanings)** → [Memory vs context](04-memory-and-retrieval/01-memory-vs-context.md)
- **mental models / glossary / vocabulary** → [Mental models and vocabulary](05-assistant-behavior-and-architecture/03-mental-models-and-vocabulary.md)
- **morphology / subword tokenization / BPE** → [Tokens vs words](01-tokens-and-context/01-tokens-vs-words.md)
- **multilingual / Chinese / mixed-script tokenization** → [Multilingual tokenization](01-tokens-and-context/02-multilingual-tokenization.md)
- **next-token prediction** → [Stateless next-token](02-decoder-only-generation/01-stateless-next-token.md)
- **permission / sandbox escalation cost (momentum vs tokens)** → [Prompt caching](02-decoder-only-generation/07-prompt-caching-and-turn-cost.md)
- **prefill phase** → [Prefill vs decode](02-decoder-only-generation/03-prefill-vs-decode.md)
- **prefix (immutable) / prefix reuse** → [KV cache](02-decoder-only-generation/04-kv-cache.md), [Prompt caching](02-decoder-only-generation/07-prompt-caching-and-turn-cost.md)
- **prompt caching / TTL / warm vs cold cache** → [Prompt caching](02-decoder-only-generation/07-prompt-caching-and-turn-cost.md)
- **RAG (retrieval-augmented generation)** → [RAG and embeddings](04-memory-and-retrieval/02-rag-and-embeddings.md)
- **retrieval-based memory** → [Memory vs context](04-memory-and-retrieval/01-memory-vs-context.md) (concept), [RAG and embeddings](04-memory-and-retrieval/02-rag-and-embeddings.md) (mechanics)
- **self-review / refinement** → [Reducing drift](03-generation-quality-and-decoding/02-reducing-drift.md)
- **statelessness** → [Stateless next-token](02-decoder-only-generation/01-stateless-next-token.md)
- **streaming / why decoding feels fast** → [Why sequential decoding is still fast](02-decoder-only-generation/05-why-decoding-is-fast.md)
- **summarized context / context management** → [Memory vs context](04-memory-and-retrieval/01-memory-vs-context.md)
- **teacher forcing / parallel-position training** → [Training](02-decoder-only-generation/08-training-a-decoder-only-model.md)
- **temperature / sampling** → [Reducing drift](03-generation-quality-and-decoding/02-reducing-drift.md)
- **tokens vs words** → [Tokens vs words](01-tokens-and-context/01-tokens-vs-words.md)
- **tools (verification, code, web, citations)** → [Reducing drift](03-generation-quality-and-decoding/02-reducing-drift.md), [Layers of an assistant](05-assistant-behavior-and-architecture/02-layers-of-an-assistant.md)
- **vector databases** → [RAG and embeddings](04-memory-and-retrieval/02-rag-and-embeddings.md)
