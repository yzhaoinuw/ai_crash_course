# Next Steps

Use this checklist alongside `work_log.md`.

## Currently Hot

Active threads — read these first to know what work is in flight:

- LLM-internals curriculum — Modules 01–05 are built; the natural next action is to author one of the "Future Modules" listed in `llm-internals/README.md` (see thread below).
- Course structure — add a root-level table-of-contents README framing `llm-internals/` as chapter 1, to be created when a second chapter (likely `agents/`) lands (see thread below).

When an agent (or human) creates or significantly updates a thread/plan here, include model + version, effort/thinking mode, and token budget (if known) in parentheses after the thread name or at the end of the status line, using the same compact convention as `work_log.md`.

Other sections below are background or paused; treat them as reference unless a new request reopens them.

## LLM-internals curriculum (claude-opus-4-8, default)

Status: in progress — foundation built, future modules not yet started.

The single Q&A note was reorganized into an ordered roadmap at `llm-internals/`
(Modules 01–05 with per-module READMEs and bite-sized lessons). Built-out content
ends at the production-system / drift-control level — the deepest question the
original Q&A actually answered.

Remaining work (each is a self-contained future module, sketched in
`llm-internals/README.md` → "Future Modules"):

- Transformer architecture internals (Q/K/V, multi-head, residual + layer norm).
- Attention scaling (O(n²), FlashAttention, sparse/sliding-window).
- KV cache engineering (GPU memory, batching, paged attention, vLLM).
- Decoding strategies (deep), then speculative decoding.
- Agent architecture; memory systems (deep); hallucination/verification;
  alignment (deep); non-autoregressive alternatives; practical prompt engineering.

When picking one up: add a numbered folder (e.g. `06-...`), follow the existing
module pattern (README with objectives + readings, numbered lessons), and promote
it from "Future Modules" to "The Roadmap" in `llm-internals/README.md`.

## Course structure: chapters and a root table of contents (claude-opus-4-8, default)

Status: planned — not started.

The repo is two-level: the root is the **course** (a table of contents of
chapters), and each chapter (currently only `llm-internals/`) holds its own
ordered module folders. Keep this seam — do not flatten module folders to the
root — because the repo is "agent/llm" notes and at least one parallel chapter is
expected.

Likely future sibling chapters (peers of `llm-internals/`):

- `agents/` — tool use, planning/ReAct, multi-agent, agent memory, evaluation.
  Strongest candidate; "Agent architecture" currently sits as a future module
  inside llm-internals and would graduate to its own chapter once filled in.
- `training-and-alignment/` — instruction tuning, RLHF, constitutional AI. Some
  llm-internals "future modules" (alignment-deep) could move here when they get
  heavy.
- `building-with-llms/` — applied: prompt engineering, RAG-as-engineering, model
  APIs, evals.

Remaining work:

- When the second chapter lands, add a **root-level `README.md` table of contents**
  with one line per chapter linking to that chapter's roadmap README (framing
  `llm-internals/` as chapter 1). Holding off until then so the TOC isn't a
  one-item list.

## Background / Paused

Sections below this line are older threads kept for context. They're not the current focus, but recording the state they were left in saves the next agent from re-deriving it.

### [Paused thread name]

[Last-known state, why it's paused, what would trigger resuming it.]
