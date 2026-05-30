# Long-Context Reliability (Lost in the Middle)

A bigger context window is not a free upgrade. Beyond the cost issue covered in
[Module 02](../02-decoder-only-generation/06-long-context-cost.md), there is a
*quality* issue: models do not use every position in a long context equally well.

## Lost in the middle

Models often use information near the beginning and end of a long context better
than information buried in the middle.

Key idea: a bigger context window does not mean the model uses all positions
equally well. Stuffing everything into the prompt can actually make answers worse
than retrieving a few well-placed, relevant snippets.

Two related terms from the vocabulary:

- **Lost-in-the-middle** — long-context models may underuse information in the
  middle of a long prompt.
- **Attention dilution** — relevant details can become harder to use as context
  gets very long.

## Practical consequence

This is a big part of why retrieval ([RAG](02-rag-and-embeddings.md)) and
summarization exist even when a model technically has room for the full history.
Often it is better to include a few relevant snippets than the entire document.

Open questions worth studying next (future module): how retrieval and
summarization compare, and when to include full context vs retrieved snippets. See
the [roadmap](../README.md#future-modules-not-yet-built-out).

## Readings

- [Arize — Lost in the Middle (summary)](https://arize.com/blog/lost-in-the-middle-how-language-models-use-long-contexts-paper-reading/)
- [Original paper — Lost in the Middle](https://arxiv.org/abs/2307.03172)
