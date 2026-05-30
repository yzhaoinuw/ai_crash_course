# The Context Window

The context window is the maximum number of tokens the model can see at one time.

It includes both:

```text
input tokens + output tokens
```

So if a model has an 8,000-token context window and your prompt is 7,999 tokens,
then in the strict theoretical case, only 1 token remains for output.

In practice, APIs often also define a separate maximum output token limit, but the
total still has to fit inside the model's context window.

Formula:

```text
prompt_tokens + generated_tokens <= context_window
```

Example:

```text
context window = 8,000 tokens
prompt = 7,000 tokens
max output = 512 tokens

actual output allowed = min(512, 8,000 - 7,000) = 512
```

But:

```text
context window = 8,000 tokens
prompt = 7,999 tokens
max output = 512 tokens

actual output allowed = min(512, 1) = 1
```

## Mental model: the whiteboard

The model can only see what fits on the whiteboard. If the board is full of prompt
text, there is little space left for the answer.
