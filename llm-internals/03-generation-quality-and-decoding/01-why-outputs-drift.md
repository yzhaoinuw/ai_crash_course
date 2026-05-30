# Why Wrong Tokens Can Derail Long Outputs

Autoregressive generation has an error-compounding problem.

The model predicts:

```text
P(token_t | token_1, token_2, ..., token_{t-1})
```

At generation time, the earlier tokens are the model's own outputs.

So if it makes a bad prediction early, later predictions condition on that bad
prefix. This can cause drift, inconsistency, or hallucination.

Related concepts:

- **exposure bias** — the model was trained on correct prefixes but generates from
  its own (possibly flawed) prefixes.
- **error compounding** — early generation errors affect later output.
- **distribution shift** between training and generation.
- **hallucination drift**.

The mitigations for this problem are the subject of the
[next lesson](02-reducing-drift.md).
