# How Much Chat History Fits Without Compression?

> Prerequisite: [The context window](03-the-context-window.md).

Very roughly, if one user+assistant exchange averages 200–350 words:

| Context size | Rough chat turns |
|---:|---:|
| 8k tokens | ~20–30 turns |
| 32k tokens | ~70–100 turns |
| 128k tokens | ~250–400 turns |
| 200k tokens | ~400–600 turns |

This ignores hidden instructions, tool schemas, memory snippets, formatting
overhead, and output tokens. Real usable capacity is lower.

Once a conversation approaches these limits, a product has to do something —
truncate, summarize, or retrieve. That is the subject of
[Module 04 — Memory, Context Management, and Retrieval](../04-memory-and-retrieval/).
