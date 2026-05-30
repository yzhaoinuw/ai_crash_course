# Stateless Next-Token Prediction

A base language model does not have human-like memory. It does not "remember"
previous chats internally the way a person remembers past conversations.

At inference time, the model receives an input context and generates the next
token repeatedly.

A simplified view:

```text
[system instructions]
[user settings / memories, if provided]
[conversation history or summary]
[retrieved relevant snippets, if any]
[current user message]
→ model predicts next token
→ model predicts next token
→ ...
```

The model only uses what is included in the current input context. Anything
outside that context is invisible unless an external system retrieves or
summarizes it and puts it back into the prompt.

This single fact — that the model is **stateless** and works only from its current
input — is the root of almost everything else in this curriculum:

- Why "memory" has to be faked from the outside
  ([Module 04](../04-memory-and-retrieval/)).
- Why identity and helpfulness come from context and training, not self-awareness
  ([Module 05](../05-assistant-behavior-and-architecture/)).
- Why the context window budget matters so much
  ([Module 01](../01-tokens-and-context/03-the-context-window.md)).
