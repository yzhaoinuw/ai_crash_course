# Module 04 — Memory, Context Management, and Retrieval

The model is stateless ([Module 02](../02-decoder-only-generation/01-stateless-next-token.md)),
so any "memory" it appears to have is constructed from the outside and injected
into the prompt. This module covers the different kinds of memory, how retrieval
(RAG) works, and why simply having a longer context window does not solve
everything.

## Learning Objectives

By the end you should be able to:

- Name the four different things people call "memory" and how they differ.
- Trace the RAG pipeline from user message to injected snippets.
- Explain what embeddings and vector databases do.
- Describe the "lost in the middle" effect and why bigger context ≠ better.

## Lessons

1. [Memory vs context](01-memory-vs-context.md)
2. [RAG, embeddings, and vector databases](02-rag-and-embeddings.md)
3. [Long-context reliability (lost in the middle)](03-long-context-reliability.md)

## Readings For This Module

### Jay Alammar — The Illustrated Word2vec

Not about ChatGPT memory directly, but one of the best visual introductions to
embeddings.

Why it matters: RAG and semantic memory rely on embedding text into vectors, then
searching for similar vectors.

```text
https://jalammar.github.io/illustrated-word2vec/
```

### ITPro — What is a Vector Database?

Readable overview of vector databases. Useful for understanding how
memory/retrieval systems can search semantically rather than by exact keyword.

Core RAG pipeline:

```text
text → embedding → vector database search → retrieved snippets → prompt → LLM answer
```

```text
https://www.itpro.com/technology/big-data/what-is-a-vector-database
```

### Arize — Lost in the Middle: How Language Models Use Long Contexts

Readable summary of the famous long-context limitation.

Key idea: models often use information near the beginning and end of a long
context better than information buried in the middle.

```text
https://arize.com/blog/lost-in-the-middle-how-language-models-use-long-contexts-paper-reading/
```

### Original Paper — Lost in the Middle

Worth skimming the abstract and figures.

Main point: a bigger context window does not mean the model uses all positions
equally well.

```text
https://arxiv.org/abs/2307.03172
```

## Where this goes next

Deeper memory-system questions — how memories are selected and stored, how systems
avoid retrieving irrelevant or stale memories — are sketched in the
[roadmap's future modules](../README.md#future-modules-not-yet-built-out).
