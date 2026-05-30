# Decoder-Only Models

ChatGPT-style models are generally decoder-only transformers.

"Decoder-only" means:

- The model uses causal, left-to-right attention.
- Each token can only attend to earlier tokens.
- The model is trained to predict the next token.
- The prompt is treated as a prefix.
- The answer is treated as a continuation.

The model does not have a separate encoder that reads the whole prompt
bidirectionally.

The entire interaction is essentially:

```text
[prompt tokens][generated answer tokens]
```

The model keeps extending the sequence.

## Does the model "decode" the prompt?

In a sense, yes.

During prefill, the decoder-only model processes the prompt using the same
next-token-prediction machinery it uses for generation.

For a prompt like:

```text
The capital of France is
```

the model internally computes predictions at positions like:

```text
The → predict capital
The capital → predict of
The capital of → predict France
The capital of France → predict is
The capital of France is → predict next token
```

But it does not output those predictions for the prompt tokens. It mostly uses the
computation to build internal representations and the KV cache.

Better wording:

```text
Prefill = process known prefix and build cache
Decode = continue the prefix by sampling/choosing new tokens
```

Prefill and decode are the subject of the
[next lesson](03-prefill-vs-decode.md).

## Mental model: continuation engine

The prompt is the prefix. The answer is the continuation.
