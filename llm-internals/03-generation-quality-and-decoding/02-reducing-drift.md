# How Systems Reduce Long-Output Drift

> Prerequisite: [Why wrong tokens derail long outputs](01-why-outputs-drift.md).

There is no perfect solution, but several techniques help.

## A. Lower-temperature sampling

Lower temperature makes the model less random and more stable.

Useful for:

- factual answers
- code
- structured reports
- long technical explanations

## B. Strong structure

A structured prompt helps anchor the output:

```text
Write the answer in these sections:
1. Background
2. Main mechanism
3. Caveats
4. Summary
```

Structure reduces drift.

## C. Chunking

Instead of generating one giant answer, an agent can:

```text
plan → write section 1 → write section 2 → review → revise
```

This re-anchors the model repeatedly.

## D. Self-review and refinement

Some systems generate a draft, critique it, and revise it.

## E. Retrieval and citations

For factual tasks, retrieval can reintroduce source material during generation.
(See [Module 04 — Memory and Retrieval](../04-memory-and-retrieval/).)

## F. Beam search / multiple candidates

Some decoding systems keep several candidate continuations and choose among them,
though this is less common in chat-style LLMs.

## G. External tools

For code, math, document editing, and data analysis, tools can verify or compute
results instead of relying only on next-token prediction.
