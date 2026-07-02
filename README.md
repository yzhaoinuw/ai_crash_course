# AI Knowledge Base

[![Agent Collab Treaty](https://raw.githubusercontent.com/yzhaoinuw/agent_collab_treaty/main/assets/treaty-adopted.svg)](https://github.com/yzhaoinuw/agent_collab_treaty)

Personal study notes and reference material on artificial intelligence, large language models, and practical AI systems.

## Topics

- [LLM Internals: Context, Memory, and Generation](llm-internals/README.md)

## Topic Index

A collapsible master map of everything the notes cover, grouped by theme. Each
entry points to the lesson that **owns** the concept. Looking up a specific keyword
instead? Each chapter's README ends with an alphabetical keyword index (e.g. the
[LLM Internals keyword index](llm-internals/README.md#keyword-index)).

### Chapter 1 — [LLM Internals](llm-internals/README.md)

<details>
<summary><b>Tokens &amp; the context window</b> — the unit the model counts and the budget it lives in</summary>

- tokens vs words, subword tokenization, morphology → [Tokens vs words](llm-internals/01-tokens-and-context/01-tokens-vs-words.md)
- multilingual, Chinese, mixed-script tokenization → [Multilingual tokenization](llm-internals/01-tokens-and-context/02-multilingual-tokenization.md)
- context window, input + output budget → [The context window](llm-internals/01-tokens-and-context/03-the-context-window.md)
- how much chat history fits before compression → [How much chat fits](llm-internals/01-tokens-and-context/04-how-much-chat-fits.md)
</details>

<details>
<summary><b>How a decoder-only LLM generates</b> — the stateless next-token machine and its serving economics</summary>

- statelessness, next-token prediction → [Stateless next-token](llm-internals/02-decoder-only-generation/01-stateless-next-token.md)
- decoder-only, causal attention, continuation engine → [Decoder-only models](llm-internals/02-decoder-only-generation/02-decoder-only-models.md)
- prefill vs decode → [Prefill vs decode](llm-internals/02-decoder-only-generation/03-prefill-vs-decode.md)
- KV cache, immutable prefix, append-only, compute-for-memory → [The KV cache](llm-internals/02-decoder-only-generation/04-kv-cache.md)
- why sequential decoding still feels fast → [Why decoding is fast](llm-internals/02-decoder-only-generation/05-why-decoding-is-fast.md)
- long-context cost, O(n²) attention → [Long context cost](llm-internals/02-decoder-only-generation/06-long-context-cost.md)
- prompt caching, prefix reuse, cache read/write, TTL, permission/escalation cost → [Prompt caching](llm-internals/02-decoder-only-generation/07-prompt-caching-and-turn-cost.md)
- training, teacher forcing, causal mask, cross-entropy, batching → [Training a decoder-only model](llm-internals/02-decoder-only-generation/08-training-a-decoder-only-model.md)
</details>

<details>
<summary><b>Generation quality &amp; decoding</b> — why long outputs drift and how to keep them on track</summary>

- output drift, exposure bias, error compounding → [Why outputs drift](llm-internals/03-generation-quality-and-decoding/01-why-outputs-drift.md)
- temperature, structure, chunking, self-review, beam search, tools → [Reducing drift](llm-internals/03-generation-quality-and-decoding/02-reducing-drift.md)
</details>

<details>
<summary><b>Memory, context management &amp; retrieval</b> — how "memory" is faked from outside the model</summary>

- memory vs context (the four meanings), summarized + structured memory → [Memory vs context](llm-internals/04-memory-and-retrieval/01-memory-vs-context.md)
- RAG, embeddings, vector databases → [RAG and embeddings](llm-internals/04-memory-and-retrieval/02-rag-and-embeddings.md)
- lost in the middle, long-context reliability → [Long-context reliability](llm-internals/04-memory-and-retrieval/03-long-context-reliability.md)
</details>

<details>
<summary><b>Assistant behavior &amp; architecture</b> — why a naked model acts like a named, helpful assistant</summary>

- identity, instruction tuning, answering vs continuing → [Identity and instruction tuning](llm-internals/05-assistant-behavior-and-architecture/01-identity-and-instruction-tuning.md)
- the layers of a deployed assistant (prompt construction → tools → safety → UI) → [Layers of an assistant](llm-internals/05-assistant-behavior-and-architecture/02-layers-of-an-assistant.md)
- mental models + key vocabulary (glossary) → [Mental models and vocabulary](llm-internals/05-assistant-behavior-and-architecture/03-mental-models-and-vocabulary.md)
</details>

> Maintenance: when a lesson is added, renamed, or re-scoped, update this map and
> the chapter keyword indexes in the same commit — see
> [AGENTS.md → Keyword and Topic Indexes](AGENTS.md#keyword-and-topic-indexes).
