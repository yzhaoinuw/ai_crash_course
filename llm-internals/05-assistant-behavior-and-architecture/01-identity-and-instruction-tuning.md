# Identity and Instruction-Tuned Behavior

## Identity and task behavior

A base model does not intrinsically "know" it is ChatGPT.

Identity and role behavior come from:

- system instructions
- conversation formatting
- instruction tuning
- supervised fine-tuning
- reinforcement learning from human feedback
- safety and alignment training

The model sees something like:

```text
System: You are ChatGPT, a helpful AI assistant.
User: What is photosynthesis?
Assistant:
```

Given that prefix, the statistically likely continuation is an answer from a
helpful assistant.

So "identity" is not self-awareness. It is context-conditioned behavior.

## Why the model answers questions instead of merely continuing sentences

A raw language model might simply continue text.

An instruction-tuned assistant model has learned patterns like:

```text
User asks question → assistant answers
User asks for code → assistant provides code
User asks for explanation → assistant explains
User asks unsafe request → assistant refuses or redirects
```

This behavior is learned through instruction tuning and alignment, and reinforced
by the chat format.

The model is still predicting next tokens, but the probability distribution has
been shaped so helpful assistant behavior is likely.

## Where this goes next

The distinction between instruction tuning, RLHF, and constitutional AI — and the
deeper question of whether a model has a self-model or only role-conditioned
behavior — is part of the "Alignment (deep)" future module in the
[roadmap](../README.md#future-modules-not-yet-built-out).
