# The Layers of a Deployed Assistant

A deployed AI assistant is not just a naked language model.

It usually includes multiple layers:

```text
1. Core LLM
   - stateless next-token predictor

2. Prompt construction system
   - inserts system instructions, user settings, recent chat history

3. Context management
   - truncates or summarizes long history

4. Retrieval system
   - searches previous chats, files, documents, or memory

5. Tool layer
   - web search, code execution, calendar, email, file tools, etc.

6. Safety and policy layer
   - constrains allowed outputs and tool use

7. UI layer
   - streaming, formatting, citations, artifacts
```

Many things that feel like "the model remembering" are actually handled by layers
outside the model.

## How this ties the curriculum together

Each layer maps back to an earlier module:

- **Layer 1 (core LLM)** — [Module 02](../02-decoder-only-generation/).
- **Layers 2–3 (prompt construction, context management)** —
  [Module 01](../01-tokens-and-context/) (the budget) and
  [Module 04](../04-memory-and-retrieval/01-memory-vs-context.md)
  (summarized context).
- **Layer 4 (retrieval)** —
  [Module 04](../04-memory-and-retrieval/02-rag-and-embeddings.md).
- **Layer 5 (tools)** — the verification lever in
  [Module 03](../03-generation-quality-and-decoding/02-reducing-drift.md), and the
  "Agent architecture" future module.
- **Layer 6 (safety/policy)** —
  [identity and instruction tuning](01-identity-and-instruction-tuning.md).
- **Layer 7 (UI)** — streaming, which is part of why generation
  [feels fast](../02-decoder-only-generation/05-why-decoding-is-fast.md).
