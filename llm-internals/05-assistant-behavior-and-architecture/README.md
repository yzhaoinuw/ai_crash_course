# Module 05 — Assistant Behavior and System Architecture

This is where the curriculum lands: pulling the pieces together into the full
picture of a deployed assistant. A naked language model just continues text. The
helpful, named, tool-using assistant you interact with is that model plus training
plus several surrounding layers.

This module sits at the edge of what the original Q&A explored in depth. Past here,
the [future modules](../README.md#future-modules-not-yet-built-out) take over.

## Learning Objectives

By the end you should be able to:

- Explain where "identity" and helpful behavior come from (and where they don't).
- Distinguish a raw language model from an instruction-tuned assistant.
- List the layers that wrap the core model in a real product.
- Hold the whole system in your head with a few durable mental models.

## Lessons

1. [Identity and instruction-tuned behavior](01-identity-and-instruction-tuning.md)
2. [The layers of a deployed assistant](02-layers-of-an-assistant.md)
3. [Mental models and key vocabulary](03-mental-models-and-vocabulary.md)

## Where this goes next

The deeper alignment questions — instruction tuning vs RLHF vs constitutional AI,
and whether a model has a self-model or only role-conditioned behavior — plus agent
architecture and hallucination/verification are all sketched in the
[roadmap's future modules](../README.md#future-modules-not-yet-built-out).
