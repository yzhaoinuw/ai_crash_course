# Memory vs Context

There are several different things people casually call "memory," but they are not
the same.

## A. Immediate conversation context

This is the current chat history included in the prompt. The model can attend to it
directly.

## B. Summarized context

When a chat gets too long, older parts may be summarized. Instead of seeing every
old message, the model may receive a compressed summary like:

```text
The user is working on a Dash app for neuroscience data visualization and has had issues with callback state persistence.
```

This is useful but lossy. Details can be lost or distorted.

## C. Retrieval-based memory

A system can search older conversations, documents, or stored memories and inject
relevant snippets into the prompt.

This is usually called RAG: retrieval-augmented generation.

Typical flow:

```text
user message
→ convert to embedding
→ search memory / documents
→ retrieve relevant snippets
→ insert snippets into prompt
→ model responds
```

RAG gets its own lesson: [RAG, embeddings, and vector databases](02-rag-and-embeddings.md).

## D. Persistent structured memory

Some platforms store durable facts or preferences about the user, such as:

```text
User prefers concise technical explanations.
User works with Python and MATLAB.
User is building neuroscience data pipelines.
```

The model does not remember this internally. The platform provides it as extra
context.

## Mental model: notes inserted before the model speaks

The model does not open its own memory. The platform may insert relevant notes into
the prompt. Many things that feel like "the model remembering" are actually handled
by layers outside the model — see
[Module 05: the layers of an assistant](../05-assistant-behavior-and-architecture/02-layers-of-an-assistant.md).
