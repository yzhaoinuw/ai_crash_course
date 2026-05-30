# Multilingual Tokenization

> Prerequisite: [Tokens vs words](01-tokens-vs-words.md).

## Chinese tokenization

Chinese is a good contrast because written Chinese usually does not put spaces
between words:

```text
我喜欢机器学习。
```

A human might segment this as:

```text
我 / 喜欢 / 机器学习 / 。
I / like / machine learning / .
```

But the tokenizer does not need a perfect word segmentation step. It can represent
the sentence using characters, common multi-character words, punctuation, or mixed
chunks:

```text
我 / 喜欢 / 机器 / 学习 / 。
```

or sometimes:

```text
我 / 喜 / 欢 / 机器学习 / 。
```

The exact split depends on the tokenizer and how common each sequence is. Very
common Chinese words such as `中国`, `学习`, `北京`, or `人工智能` may be kept as
larger chunks by some tokenizers. Less common names, rare characters,
domain-specific terms, or mixed Chinese-English text may split more finely.

## Mixed-script text

This is one reason multilingual tokenization is tricky. English uses spaces,
Chinese usually does not, and both can appear in the same prompt:

```text
请解释 transformer attention 的核心思想。
```

A multilingual tokenizer can still handle this because its vocabulary contains
pieces from many writing systems: Latin letters, Chinese characters, punctuation,
spaces, digits, and frequent cross-language chunks. The model then learns
relationships among those token sequences during training. It is not limited to
one language's idea of what a "word" is.

The tradeoff is efficiency. Some languages or scripts may require more tokens per
idea than English, depending on the tokenizer. Newer multilingual tokenizers try
to reduce this imbalance by including better chunks for many languages, not just
English.
