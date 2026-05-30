# Tokens vs Words

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

This is approximate. Code, Chinese text, unusual words, formatting, and
punctuation can change the ratio.

## English examples: words are not the unit

A word can be one token, several tokens, or share a token with nearby punctuation
or whitespace. Exact splits depend on the tokenizer, but the pattern is important:

| Text | Approximate token intuition | Why |
|---|---|---|
| `cat` | `cat` | A common short word is often a single token. |
| `the cat` | `the`, ` cat` | Many tokenizers learn common word-with-leading-space chunks. |
| `unhappy` | `un`, `happy` or `unhappy` | A common whole word may be kept intact; otherwise a prefix plus stem may be used. |
| `unhappiness` | `un`, `happiness` or `un`, `happy`, `ness` | Morphology matters: prefix, stem, and suffix can become useful reusable pieces. |
| `anti-inflammatory` | `anti`, `-`, `inflammatory` or smaller pieces | Hyphens and specialist words often create extra boundaries. |
| `ChatGPT-like` | `Chat`, `GPT`, `-`, `like` or similar | Mixed capitalization, acronyms, and punctuation often split. |
| `walk`, `walked`, `walking` | often related but not identical splits | Inflectional endings like `-ed` and `-ing` may be separate when that helps reuse patterns. |

This is why "1 word = 1 token" is the wrong mental model. A tokenizer is not doing
dictionary lookup in the human sense. It is compressing text into reusable pieces
that appeared often enough in training data to be useful.

Morphology comes into play because English words are built from meaningful parts:

```text
reusable        = re + usable, or reusable
unusable        = un + usable, or unusable
unbelievably    = un + believe + ably, or unbelievable + ly
tokenization    = token + ization, or tokenization
antidisestablishmentarianism = anti + dis + establishment + arian + ism, or other chunks
```

Common forms are more likely to be stored as larger chunks. Rare forms, newly
coined words, long technical terms, misspellings, and unusual compounds are more
likely to be broken into smaller subword pieces.

## Same sub-part, different treatment

The same visible string can be handled differently depending on context and
frequency.

For example, `ing` might be a separate suffix-like token in one word:

```text
walking -> walk + ing
debugging -> debug + ging, or debugging
singing -> singing, or sing + ing
```

The tokenizer is not applying a grammar rule that says "`ing` is always a suffix."
It has learned a vocabulary of frequent character sequences. If `singing` is
common enough, it may be kept whole. If a related form is less common, the
tokenizer may fall back to smaller pieces.

Leading spaces are another non-obvious case:

```text
cat
 cat
```

Many tokenizers treat these as different token candidates. This helps the model
learn that words usually appear after spaces in English, while still handling word
starts, punctuation, and line breaks.
