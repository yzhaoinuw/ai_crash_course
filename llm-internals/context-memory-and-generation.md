# Study Notes: How ChatGPT / Decoder-Only LLMs Handle Context, Memory, and Generation

Date: 2026-05-27

## 1. The core idea: an LLM is stateless by default

A base language model does not have human-like memory. It does not “remember” previous chats internally the way a person remembers past conversations.

At inference time, the model receives an input context and generates the next token repeatedly.

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

The model only uses what is included in the current input context. Anything outside that context is invisible unless an external system retrieves or summarizes it and puts it back into the prompt.

---

## 2. Memory vs context

There are several different things people casually call “memory,” but they are not the same.

### A. Immediate conversation context

This is the current chat history included in the prompt. The model can attend to it directly.

### B. Summarized context

When a chat gets too long, older parts may be summarized. Instead of seeing every old message, the model may receive a compressed summary like:

```text
The user is working on a Dash app for neuroscience data visualization and has had issues with callback state persistence.
```

This is useful but lossy. Details can be lost or distorted.

### C. Retrieval-based memory

A system can search older conversations, documents, or stored memories and inject relevant snippets into the prompt.

This is usually called RAG: retrieval-augmented generation.

Typical flow:

```text
user message
→ convert to embedding
→ search memory / documents
→ retrieve relevant snippets
→ insert snippets into prompt
→ model responds
```

### D. Persistent structured memory

Some platforms store durable facts or preferences about the user, such as:

```text
User prefers concise technical explanations.
User works with Python and MATLAB.
User is building neuroscience data pipelines.
```

The model does not remember this internally. The platform provides it as extra context.

---

## 3. The context window

The context window is the maximum number of tokens the model can see at one time.

It includes both:

```text
input tokens + output tokens
```

So if a model has an 8,000-token context window and your prompt is 7,999 tokens, then in the strict theoretical case, only 1 token remains for output.

In practice, APIs often also define a separate maximum output token limit, but the total still has to fit inside the model’s context window.

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

---

## 4. Tokens vs words

Models count tokens, not words.

A rough English conversion:

```text
1 token ≈ 0.75 words
1 word ≈ 1.3 tokens
```

Approximate context sizes:

| Token window | Approximate words |
|---:|---:|
| 8k tokens | ~6k words |
| 32k tokens | ~24k words |
| 128k tokens | ~95k words |
| 200k tokens | ~150k words |
| 1M tokens | ~700k words |

This is approximate. Code, Chinese text, unusual words, formatting, and punctuation can change the ratio.

---

## 5. How much chat history fits without compression?

Very roughly, if one user+assistant exchange averages 200–350 words:

| Context size | Rough chat turns |
|---:|---:|
| 8k tokens | ~20–30 turns |
| 32k tokens | ~70–100 turns |
| 128k tokens | ~250–400 turns |
| 200k tokens | ~400–600 turns |

This ignores hidden instructions, tool schemas, memory snippets, formatting overhead, and output tokens. Real usable capacity is lower.

---

## 6. Why long context is impressive but not free

Longer prompts cost more and take longer because the model has to process more tokens.

In a vanilla transformer, attention cost scales roughly like:

```text
O(n²)
```

where `n` is the number of tokens.

That means longer context can become expensive fast.

Modern systems use optimizations such as:

- FlashAttention
- KV caching
- grouped-query attention
- sparse/sliding-window attention
- batching
- specialized GPU kernels
- memory-efficient serving infrastructure

But long context still increases latency and cost.

---

## 7. Decoder-only models

ChatGPT-style models are generally decoder-only transformers.

“Decoder-only” means:

- The model uses causal, left-to-right attention.
- Each token can only attend to earlier tokens.
- The model is trained to predict the next token.
- The prompt is treated as a prefix.
- The answer is treated as a continuation.

The model does not have a separate encoder that reads the whole prompt bidirectionally.

The entire interaction is essentially:

```text
[prompt tokens][generated answer tokens]
```

The model keeps extending the sequence.

---

## 8. Prefill vs decode

LLM inference has two major phases.

### A. Prefill phase

The model processes the input prompt.

During prefill:

- All prompt tokens are known.
- The model computes hidden states and attention information.
- It builds the KV cache.
- It may compute logits for every position, but those logits are usually ignored except for the final position.
- This phase can be parallelized across the prompt tokens using causal masking.

Even though the model is “decoder-only,” it can process the prompt in parallel because the full prompt is already available.

### B. Decode phase

The model generates the response one token at a time.

During decode:

```text
current context → predict next token
append token
current context + new token → predict next token
append token
...
```

This is sequential at the token level.

But each token-generation step uses massively parallel GPU computation internally.

---

## 9. Does the model “decode” the prompt?

In a sense, yes.

During prefill, the decoder-only model processes the prompt using the same next-token-prediction machinery it uses for generation.

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

But it does not output those predictions for the prompt tokens. It mostly uses the computation to build internal representations and the KV cache.

Better wording:

```text
Prefill = process known prefix and build cache
Decode = continue the prefix by sampling/choosing new tokens
```

---

## 10. KV cache

The KV cache stores key/value tensors from previous tokens so the model does not need to recompute the whole prompt every time it generates a new token.

Without KV cache:

```text
generate token 1 → process whole prompt
generate token 2 → process whole prompt + token 1 again
generate token 3 → process whole prompt + token 1 + token 2 again
```

That would be very slow.

With KV cache:

```text
process prompt once
store key/value states
for each new token:
    compute only the new token's states
    attend to cached previous states
```

This is one of the main reasons autoregressive decoding is fast enough to feel interactive.

---

## 11. Why sequential decoding is still fast

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

---

## 12. Why wrong tokens can derail long outputs

Autoregressive generation has an error-compounding problem.

The model predicts:

```text
P(token_t | token_1, token_2, ..., token_{t-1})
```

At generation time, the earlier tokens are the model’s own outputs.

So if it makes a bad prediction early, later predictions condition on that bad prefix. This can cause drift, inconsistency, or hallucination.

Related concepts:

- exposure bias
- error compounding
- distribution shift between training and generation
- hallucination drift

---

## 13. How systems reduce long-output drift

There is no perfect solution, but several techniques help:

### A. Lower-temperature sampling

Lower temperature makes the model less random and more stable.

Useful for:

- factual answers
- code
- structured reports
- long technical explanations

### B. Strong structure

A structured prompt helps anchor the output:

```text
Write the answer in these sections:
1. Background
2. Main mechanism
3. Caveats
4. Summary
```

Structure reduces drift.

### C. Chunking

Instead of generating one giant answer, an agent can:

```text
plan → write section 1 → write section 2 → review → revise
```

This re-anchors the model repeatedly.

### D. Self-review and refinement

Some systems generate a draft, critique it, and revise it.

### E. Retrieval and citations

For factual tasks, retrieval can reintroduce source material during generation.

### F. Beam search / multiple candidates

Some decoding systems keep several candidate continuations and choose among them, though this is less common in chat-style LLMs.

### G. External tools

For code, math, document editing, and data analysis, tools can verify or compute results instead of relying only on next-token prediction.

---

## 14. Identity and task behavior

A base model does not intrinsically “know” it is ChatGPT.

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

Given that prefix, the statistically likely continuation is an answer from a helpful assistant.

So “identity” is not self-awareness. It is context-conditioned behavior.

---

## 15. Why the model answers questions instead of merely continuing sentences

A raw language model might simply continue text.

An instruction-tuned assistant model has learned patterns like:

```text
User asks question → assistant answers
User asks for code → assistant provides code
User asks for explanation → assistant explains
User asks unsafe request → assistant refuses or redirects
```

This behavior is learned through instruction tuning and alignment, and reinforced by the chat format.

The model is still predicting next tokens, but the probability distribution has been shaped so helpful assistant behavior is likely.

---

## 16. The important layers to distinguish

A deployed AI assistant is not just a naked language model.

It usually includes multiple layers:

```text
1. Core LLM
   - stateless next-token predictor

2. Prompt construction system
   - inserts system instructions, user settings, recent chat history

3. Context management
   - truncates or summarizes long history

4. Retrieval system
   - searches previous chats, files, documents, or memory

5. Tool layer
   - web search, code execution, calendar, email, file tools, etc.

6. Safety and policy layer
   - constrains allowed outputs and tool use

7. UI layer
   - streaming, formatting, citations, artifacts
```

Many things that feel like “the model remembering” are actually handled by layers outside the model.

---

## 17. Mental models

### Context window as a whiteboard

The model can only see what fits on the whiteboard.

If the board is full of prompt text, there is little space left for the answer.

### Memory as notes inserted before the model speaks

The model does not open its own memory. The platform may insert relevant notes into the prompt.

### Decoder-only model as continuation engine

The prompt is the prefix. The answer is the continuation.

### Prefill as reading, decode as writing

This is not architecturally exact, but it is a useful intuition.

### KV cache as not rereading the whole book

The model processes the prompt once and caches useful internal states, then generates new tokens efficiently.

---

## 18. Key vocabulary

- Token: A chunk of text used by the model, often smaller than a word.
- Context window: Maximum number of tokens the model can attend to at once.
- Prompt: The input tokens supplied to the model.
- Completion / output: Tokens generated by the model.
- Decoder-only transformer: A transformer that uses causal attention and generates left to right.
- Causal mask: Prevents each token from attending to future tokens.
- Prefill: Processing the known prompt and building the KV cache.
- Decode: Autoregressively generating new tokens.
- KV cache: Stored key/value attention tensors from previous tokens.
- Attention: Mechanism that lets tokens condition on earlier tokens.
- RAG: Retrieval-augmented generation; external search plus prompt injection.
- Embedding: Vector representation of text used for semantic search.
- Instruction tuning: Training that teaches a model to follow user instructions.
- RLHF: Reinforcement learning from human feedback.
- Exposure bias: Mismatch between training on correct prefixes and generating from the model’s own prefixes.
- Error compounding: Early generation errors affecting later output.
- Temperature: Sampling control for randomness.
- Top-p / nucleus sampling: Sampling from a probability mass cutoff.
- Beam search: Maintaining multiple candidate output paths.
- Lost-in-the-middle: Long-context models may underuse information in the middle of a long prompt.
- Attention dilution: Relevant details can become harder to use as context gets very long.

---

## 19. Interesting topics to expand next

### A. Transformer architecture

Questions to study:

- What exactly are query, key, and value vectors?
- Why does attention use dot products?
- What does multi-head attention buy us?
- Why are residual connections and layer normalization important?
- What is the difference between encoder-only, decoder-only, and encoder-decoder transformers?

### B. Attention scaling

Questions to study:

- Why is vanilla attention O(n²)?
- How does FlashAttention reduce memory overhead?
- What are sparse attention and sliding-window attention?
- Why are million-token contexts hard to run?

### C. KV cache engineering

Questions to study:

- How much GPU memory does KV cache consume?
- Why does batch size affect latency and throughput?
- What are paged attention and vLLM?
- Why do long chats get expensive even when output is short?

### D. Long-context reliability

Questions to study:

- What is the lost-in-the-middle effect?
- Why can longer context sometimes make answers worse?
- How do retrieval and summarization compare?
- When should you include full context vs retrieved snippets?

### E. Decoding strategies

Questions to study:

- Greedy decoding vs sampling
- Temperature
- Top-k sampling
- Top-p / nucleus sampling
- Beam search
- Speculative decoding
- Contrastive decoding

### F. Speculative decoding

Questions to study:

- How can a smaller draft model speed up a larger model?
- How does the large model verify proposed tokens?
- Why does this preserve output distribution while improving speed?

### G. Agent architecture

Questions to study:

- How do agents decide when to retrieve memory?
- How do agents decide when to call tools?
- What is the difference between a model, an assistant, and an agent?
- How do planners, executors, critics, and memory modules fit together?

### H. Memory systems

Questions to study:

- How are user memories selected and stored?
- What is semantic search?
- What are embeddings?
- How do vector databases work?
- How do systems avoid retrieving irrelevant memories?
- How do they prevent stale or wrong memories from polluting answers?

### I. Hallucination and verification

Questions to study:

- Why do next-token models hallucinate?
- Why does sounding fluent not imply truth?
- How do citations and tools reduce hallucination?
- Why are math and code better handled with external execution tools?
- What is self-consistency checking?

### J. Alignment and assistant identity

Questions to study:

- What is instruction tuning?
- What is RLHF?
- What is constitutional AI?
- How does a system prompt shape behavior?
- Does a model have a self-model, or only role-conditioned behavior?

### K. Non-autoregressive alternatives

Questions to study:

- Can language models generate full outputs non-sequentially?
- What are diffusion language models?
- Why are autoregressive models still dominant?
- What would it take for a model to revise earlier tokens naturally?

### L. Practical prompt engineering

Questions to study:

- How do you anchor long responses?
- How do you design prompts that reduce drift?
- Why does asking for an outline first help?
- When should you chunk a task?
- How do you ask a model to verify its own work without overtrusting it?

---

## 20. One-paragraph summary

A ChatGPT-like assistant is best understood as a stateless decoder-only transformer wrapped inside a larger product system. The core model predicts the next token from the context it is given. It does not inherently remember prior chats, know its identity, or search files by itself. Those behaviors come from system prompts, instruction tuning, memory/retrieval systems, context management, and tools. The model processes the prompt during a parallelizable prefill phase, builds a KV cache, then generates output sequentially during decode. Long contexts and long outputs are powerful but expensive and error-prone, so production systems use summarization, retrieval, chunking, structured prompting, and tool verification to keep responses coherent and useful.

---

## Appendix: Suggested Readings and Visual Explainers for LLM Internals

This appendix collects short, visual, and practical resources for going deeper into the key concepts from the study notes: tokens, context windows, transformer attention, decoder-only generation, KV cache, RAG/memory, long-context limitations, and decoding strategies.

---

### Recommended Reading Path

For a smooth learning curve, read in this order:

1. OpenAI token article + OpenAI tokenizer
2. 3Blue1Brown attention video
3. Jay Alammar, *The Illustrated GPT-2*
4. Hugging Face decoding strategies
5. Hugging Face KV caching
6. Sebastian Raschka KV cache from scratch
7. Jay Alammar, *The Illustrated Word2vec*
8. Lost in the Middle summary
9. Lilian Weng, *The Transformer Family v2.0*

This path goes from:

```text
tokens → attention → decoder-only generation → sampling → inference speed → retrieval/memory → long-context failure modes
```

---

### 1. Tokens and Context Windows

#### OpenAI — What are tokens and how to count them?

Good first read for understanding why LLM input/output limits are measured in tokens rather than words.

Key ideas:

- Tokens are chunks of text.
- One English token is roughly 4 characters or about 3/4 of a word.
- Both input and output tokens count toward model usage and limits.

Link:

```text
https://help.openai.com/en/articles/4936856-what-are-tokens-and-how-to-count-them
```

#### OpenAI Tokenizer

Interactive tool for seeing how your own text is split into tokens.

Useful exercise:

Paste a paragraph, a code block, and some Chinese text. Compare how tokenization changes across different kinds of input.

Link:

```text
https://platform.openai.com/tokenizer
```

---

### 2. Transformer and Attention Intuition

#### Jay Alammar — The Illustrated Transformer

Classic visual introduction to the Transformer architecture.

Even though it focuses on the original encoder-decoder Transformer, the attention diagrams are extremely helpful.

Best for:

- attention intuition
- query/key/value concepts
- multi-head attention
- encoder vs decoder basics

Link:

```text
https://jalammar.github.io/illustrated-transformer/
```

#### Jay Alammar — The Illustrated GPT-2

Closer to ChatGPT-style decoder-only models.

Best for:

- GPT-style generation
- decoder-only architecture
- how text prediction works
- how attention is used in language generation

Link:

```text
https://jalammar.github.io/illustrated-gpt2/
```

#### 3Blue1Brown — Attention in Transformers, Step by Step

Probably the best visual explanation of attention.

Best for:

- why attention works
- queries, keys, values
- how token meaning changes with context
- intuitive geometric view of attention

Link:

```text
https://www.3blue1brown.com/lessons/attention
```

---

### 3. Decoder-Only Generation and Sampling

#### Hugging Face — Decoding Strategies in Large Language Models

Practical explanation of how models choose the next token.

Covers:

- greedy decoding
- beam search
- temperature
- top-k sampling
- top-p / nucleus sampling

This connects directly to the question:

```text
If one wrong token can throw off later output, how do we control generation?
```

Link:

```text
https://huggingface.co/blog/mlabonne/decoding-strategies
```

#### AssemblyAI — Decoding Strategies: How LLMs Choose The Next Word

Readable companion article on decoding.

Good for reinforcing the idea that the model produces a probability distribution, and a decoding strategy turns that distribution into actual text.

Link:

```text
https://www.assemblyai.com/blog/decoding-strategies-how-llms-choose-the-next-word
```

---

### 4. KV Cache and Why Generation Is Fast

#### Hugging Face — KV Caching Explained

Short, approachable explanation of why generation does not recompute the whole prompt every token.

Key idea:

```text
Process the prompt once → store key/value tensors → reuse them during token-by-token generation
```

Link:

```text
https://huggingface.co/blog/not-lain/kv-caching
```

#### Sebastian Raschka — Understanding and Coding the KV Cache in LLMs from Scratch

More technical but very useful.

Best for:

- seeing KV cache in code
- understanding the speed/memory tradeoff
- connecting theory to implementation

Link:

```text
https://magazine.sebastianraschka.com/p/coding-the-kv-cache-in-llms
```

#### Daily Dose of Data Science — KV Caching in LLMs, Explained Visually

Visual explanation of KV cache.

Good for building intuition before reading implementation-level material.

Link:

```text
https://blog.dailydoseofds.com/p/kv-caching-in-llms-explained-visually
```

---

### 5. Embeddings, Retrieval, and Memory

#### Jay Alammar — The Illustrated Word2vec

Not about ChatGPT memory directly, but it is one of the best visual introductions to embeddings.

Why it matters:

RAG and semantic memory rely on embedding text into vectors, then searching for similar vectors.

Link:

```text
https://jalammar.github.io/illustrated-word2vec/
```

#### ITPro — What is a Vector Database?

Readable overview of vector databases.

Useful for understanding how memory/retrieval systems can search semantically rather than by exact keyword.

Core RAG pipeline:

```text
text → embedding → vector database search → retrieved snippets → prompt → LLM answer
```

Link:

```text
https://www.itpro.com/technology/big-data/what-is-a-vector-database
```

---

### 6. Long Context and “Lost in the Middle”

#### Arize — Lost in the Middle: How Language Models Use Long Contexts

Readable summary of the famous long-context limitation.

Key idea:

Models often use information near the beginning and end of a long context better than information buried in the middle.

Link:

```text
https://arize.com/blog/lost-in-the-middle-how-language-models-use-long-contexts-paper-reading/
```

#### Original Paper — Lost in the Middle

This is a paper, but it is worth skimming the abstract and figures.

Main point:

A bigger context window does not mean the model uses all positions equally well.

Link:

```text
https://arxiv.org/abs/2307.03172
```

---

### 7. Slightly Deeper but Worth Keeping

#### Lilian Weng — The Transformer Family v2.0

Higher-density but excellent.

Best for:

- encoder-only vs decoder-only vs encoder-decoder models
- architecture variants
- positional encoding
- Transformer design evolution

Link:

```text
https://lilianweng.github.io/posts/2023-01-27-the-transformer-family-v2/
```

#### Harvard NLP — The Annotated Transformer

Code-heavy.

Read this when you want to see a Transformer implemented line by line.

Best for:

- implementation details
- PyTorch-style architecture
- understanding the original Transformer from code

Link:

```text
https://nlp.seas.harvard.edu/annotated-transformer/
```

---

### 8. Suggested Follow-Up Topics

These are the next rabbit holes worth exploring.

#### Transformer architecture

Questions:

- What exactly are query, key, and value vectors?
- Why does attention use dot products?
- What does multi-head attention buy us?
- Why are residual connections and layer normalization important?
- What is the difference between encoder-only, decoder-only, and encoder-decoder models?

#### Attention scaling

Questions:

- Why is vanilla attention O(n²)?
- How does FlashAttention reduce memory overhead?
- What are sparse attention and sliding-window attention?
- Why are million-token contexts hard to run?

#### KV cache engineering

Questions:

- How much GPU memory does KV cache consume?
- Why does batch size affect latency and throughput?
- What are paged attention and vLLM?
- Why do long chats get expensive even when output is short?

#### Long-context reliability

Questions:

- What is the lost-in-the-middle effect?
- Why can longer context sometimes make answers worse?
- How do retrieval and summarization compare?
- When should you include full context vs retrieved snippets?

#### Decoding strategies

Questions:

- Greedy decoding vs sampling
- Temperature
- Top-k sampling
- Top-p / nucleus sampling
- Beam search
- Speculative decoding
- Contrastive decoding

#### Speculative decoding

Questions:

- How can a smaller draft model speed up a larger model?
- How does the large model verify proposed tokens?
- Why can this preserve output quality while improving speed?

#### Agent architecture

Questions:

- How do agents decide when to retrieve memory?
- How do agents decide when to call tools?
- What is the difference between a model, an assistant, and an agent?
- How do planners, executors, critics, and memory modules fit together?

#### Memory systems

Questions:

- How are user memories selected and stored?
- What is semantic search?
- What are embeddings?
- How do vector databases work?
- How do systems avoid retrieving irrelevant memories?
- How do they prevent stale or wrong memories from polluting answers?

#### Hallucination and verification

Questions:

- Why do next-token models hallucinate?
- Why does sounding fluent not imply truth?
- How do citations and tools reduce hallucination?
- Why are math and code better handled with external execution tools?
- What is self-consistency checking?

#### Alignment and assistant identity

Questions:

- What is instruction tuning?
- What is RLHF?
- What is constitutional AI?
- How does a system prompt shape behavior?
- Does a model have a self-model, or only role-conditioned behavior?

#### Non-autoregressive alternatives

Questions:

- Can language models generate full outputs non-sequentially?
- What are diffusion language models?
- Why are autoregressive models still dominant?
- What would it take for a model to revise earlier tokens naturally?

#### Practical prompt engineering

Questions:

- How do you anchor long responses?
- How do you design prompts that reduce drift?
- Why does asking for an outline first help?
- When should you chunk a task?
- How do you ask a model to verify its own work without overtrusting it?

---

### 9. Compact Bookmark List

```text
OpenAI tokens:
https://help.openai.com/en/articles/4936856-what-are-tokens-and-how-to-count-them

OpenAI tokenizer:
https://platform.openai.com/tokenizer

The Illustrated Transformer:
https://jalammar.github.io/illustrated-transformer/

The Illustrated GPT-2:
https://jalammar.github.io/illustrated-gpt2/

3Blue1Brown attention:
https://www.3blue1brown.com/lessons/attention

Hugging Face decoding strategies:
https://huggingface.co/blog/mlabonne/decoding-strategies

AssemblyAI decoding strategies:
https://www.assemblyai.com/blog/decoding-strategies-how-llms-choose-the-next-word

Hugging Face KV caching:
https://huggingface.co/blog/not-lain/kv-caching

Sebastian Raschka KV cache:
https://magazine.sebastianraschka.com/p/coding-the-kv-cache-in-llms

Daily Dose of Data Science KV cache:
https://blog.dailydoseofds.com/p/kv-caching-in-llms-explained-visually

The Illustrated Word2vec:
https://jalammar.github.io/illustrated-word2vec/

Vector database overview:
https://www.itpro.com/technology/big-data/what-is-a-vector-database

Lost in the Middle summary:
https://arize.com/blog/lost-in-the-middle-how-language-models-use-long-contexts-paper-reading/

Lost in the Middle paper:
https://arxiv.org/abs/2307.03172

Lilian Weng Transformer Family:
https://lilianweng.github.io/posts/2023-01-27-the-transformer-family-v2/

Annotated Transformer:
https://nlp.seas.harvard.edu/annotated-transformer/
```

