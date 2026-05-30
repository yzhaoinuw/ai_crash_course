# RAG, Embeddings, and Vector Databases

> Prerequisite: [Memory vs context](01-memory-vs-context.md), section C.

Retrieval-augmented generation (RAG) is the mechanism behind retrieval-based
memory. Instead of relying on what the model internally "knows," the system
searches an external store and injects relevant text into the prompt before the
model answers.

## The pipeline

```text
text → embedding → vector database search → retrieved snippets → prompt → LLM answer
```

Step by step, for a user message:

```text
user message
→ convert to embedding
→ search memory / documents
→ retrieve relevant snippets
→ insert snippets into prompt
→ model responds
```

## Embeddings

An **embedding** is a vector representation of text used for semantic search. Text
with similar meaning maps to nearby vectors, so "find related text" becomes "find
nearby vectors" — even when the wording is different and exact keywords do not
match.

The [Illustrated Word2vec](https://jalammar.github.io/illustrated-word2vec/)
reading is the best visual intro to why this works.

## Vector databases

A **vector database** stores embeddings and supports fast nearest-neighbor search.
It is what lets a memory/retrieval system search *semantically* rather than by
exact keyword. See the
[vector database overview](https://www.itpro.com/technology/big-data/what-is-a-vector-database).

## The hard parts (future work)

Retrieval is not magic. Real systems still have to avoid pulling in irrelevant
snippets and prevent stale or wrong memories from polluting answers. Those
questions are part of the "Memory systems (deep)" future module in the
[roadmap](../README.md#future-modules-not-yet-built-out).
