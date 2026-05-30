# Module 01 — Tokens and the Context Window

The model does not see words or characters directly. It sees **tokens**, and it
can only hold a fixed number of them at once — the **context window**. This module
builds the foundation everything else rests on.

## Learning Objectives

By the end you should be able to:

- Explain why "1 word = 1 token" is the wrong mental model.
- Predict roughly how text (English, Chinese, mixed, code) gets split.
- State what counts against the context window and why output competes with input.
- Estimate how much conversation fits before compression kicks in.

## Lessons

1. [Tokens vs words](01-tokens-vs-words.md)
2. [Multilingual tokenization](02-multilingual-tokenization.md)
3. [The context window](03-the-context-window.md)
4. [How much chat history fits](04-how-much-chat-fits.md)

## Readings For This Module

### OpenAI — What are tokens and how to count them?

Good first read for understanding why LLM input/output limits are measured in
tokens rather than words.

Key ideas:

- Tokens are chunks of text.
- One English token is roughly 4 characters or about 3/4 of a word.
- Both input and output tokens count toward model usage and limits.

Link:

```text
https://help.openai.com/en/articles/4936856-what-are-tokens-and-how-to-count-them
```

### OpenAI Tokenizer

Interactive tool for seeing how your own text is split into tokens.

Useful exercise: paste a paragraph, a code block, and some Chinese text. Compare
how tokenization changes across different kinds of input.

Link:

```text
https://platform.openai.com/tokenizer
```
