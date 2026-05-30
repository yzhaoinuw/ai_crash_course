# Why Sequential Decoding Is Still Fast

Output generation is sequential, but not slow in human terms because:

- Each token step runs huge matrix operations in parallel on GPUs.
- KV caching avoids recomputing the prompt.
- Servers batch requests across users.
- Streaming displays tokens as they are produced.
- Human reading speed is much slower than model token generation speed.

The outer loop is sequential:

```python
for token in output:
    generate_next_token()
```

But the body of the loop is massively parallel tensor math.
