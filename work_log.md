# Work Log

Prepend new session notes to the top of this file.

Rotation policy: the live log holds at most the **5 most recent unique calendar dates**. When a new date would push the file past 5 unique dates, move the oldest 5 dates as a chunk into a new file at `work_log_archive/work_log_<earliest>_to_<latest>.md`. The live file always holds at most 5 unique dates; each archive file always holds exactly 5.

If today's date already has a `## YYYY-MM-DD` header at the top, add a new `###` session subsection under it rather than starting a second `## YYYY-MM-DD` header for the same date.

Update this log at the end of any substantive work session unless the user explicitly asks not to document it. Substantive work includes file edits, meaningful validation or debugging, technical decisions or reversals, reusable discoveries, branch/PR/release state changes, or follow-up work that future agents need. Log useful experiments even when the code was reverted; skip casual Q&A, trivial one-off commands, and pure scratch work with no future coordination value.

## 2026-06-03

### Added training internals lesson and note-writing conventions (claude-sonnet-4-6, high effort)

- Conversation covered: what a training sample looks like (raw text chunks, not sentences), teacher forcing, why all positions train in parallel, the causal mask, cross-entropy as distribution learning (not point-guess), batching semantics (within-sequence vs across-sequence).
- Created `llm-internals/02-decoder-only-generation/07-training-a-decoder-only-model.md` with `text` block illustrations for all major concepts.
- Updated `02-decoder-only-models.md` to add a causal mask grid where the file had previously only described the rule in prose.
- Updated module `README.md`: added lesson 7, one new learning objective, and three new training-focused readings (Karpathy "Let's Build GPT", Lilian Weng generalized LMs, Sebastian Raschka's LLM book).
- Updated `AGENTS.md` Project-Specific Reminders: added note-writing style guidance — use `text` illustrations when they add real value (not by default), suggest 2–4 further readings at end of substantive learning sessions, prefer blogs/articles over papers.

## 2026-05-31

### Added treaty adoption badge + fixed broken README Topics link (claude-opus-4-8, default)

- User noticed no Agent Collab Treaty badge on the GitHub page. Clarified the badge is opt-in (offered by `treaty init`, not added by merely following the treaty) and was never added here.
- Sourced the real badge markup from the template repo (`gh:yzhaoinuw/agent_collab_treaty`, `copier.yml`) instead of hand-writing a URL. Did NOT run `treaty init` in-repo (would re-template the customized AGENTS.md/work_log.md); cloned to a temp dir, extracted markup, removed temp dir.
- User chose the tri-color SVG variant. Added it to `README.md`: `assets/treaty-adopted.svg` raw URL linking to the treaty repo. Caveat flagged to user: the template warns raw-SVG badges may not render through GitHub's camo proxy for all viewers; shields.io variant is the reliable fallback (one-line swap if it shows broken).
- Also fixed a pre-existing broken link: README "Topics" pointed at the deleted monolith `llm-internals/context-memory-and-generation.md`; repointed to `llm-internals/README.md` (the roadmap index from the restructure).

### Added an information-flow illustration to the KV cache note (claude-opus-4-8, default)

- User asked for a simple illustration of information-flow direction and what happens when a new KV entry is created. Added an "Illustration: information flow and the append step" subsection to `llm-internals/02-decoder-only-generation/04-kv-cache.md` using ASCII/`text` diagrams: (1) a new token ("mat") forming a query, attending over the frozen cache, producing its own K/V, and being appended; (2) a step-by-step append trace showing earlier entries never change; (3) a one-way arrow-of-flow diagram (new-reads-old, never backward).
- Content-only change; no tests/runtime in this repo.

## 2026-05-30

### Expanded the KV cache note with the speedup trade-off (claude-opus-4-8, default)

- User asked "what's the catch with the KV cache" and proposed a mental model (cached tokens' attention to each other is frozen; only the new token attends over the cache). Confirmed the model is correct and grounded it in causal masking.
- Added two sections to `llm-internals/02-decoder-only-generation/04-kv-cache.md`:
  - "Why it is safe to cache" — frozen K/V for past tokens, one new query per decode step, and the immutable-prefix caveat (edit anything early → cache invalid downstream). Emphasizes the cache is exact, not an approximation.
  - "So what is the catch? (compute for memory)" — the trade is recomputation for memory: (1) cache size grows linearly and can exceed model weights, capping context length + concurrency; (2) decode is memory-bandwidth bound, since each step re-reads the whole cache, so per-token latency grows with context length even though the per-step math is constant. Connects to why paged attention / GQA / quantized caches exist (the "KV cache engineering" future module).
- Follow-up: user pushed back with the common misconception that earlier tokens must be re-attended to the expanded context to "stay accurate." Added a subsection "But don't the earlier words need to re-read the expanded context?" to the same note: cache is append-only (written once per token, never rewritten); decoder-only models are causal by design/training so earlier tokens are *not supposed* to see later context; context integrates at the frontier (new-reads-old), never backward; and this is why bidirectional BERT-style encoders cannot use an append-only KV cache.
- Content-only change; no tests/runtime in this repo.

### Recorded course-structure decision; committed + pushed the restructure (claude-opus-4-8, default)

- Decision: repo is two-level — root = course (chapter table of contents), each chapter (e.g. `llm-internals/`) holds ordered module folders. Do not flatten module folders to root; `agents/` is the expected next parallel chapter.
- Added a "Course structure" thread to `next_steps.md`: a root-level TOC README is deferred until a second chapter lands (so it isn't a one-item list).
- Committed the full restructure + treaty-doc updates and pushed to `main`.

### Restructured llm-internals into an ordered study roadmap (claude-opus-4-8, default)

- Read the single monolithic Q&A note `llm-internals/context-memory-and-generation.md` to characterize the topics actually explored.
- Designed a 5-module roadmap (bite-sized lessons) ending where the deepest *answered* question sat — production-system architecture / drift control. Aspirational `to-expand` topics (§19 + appendix §8) were captured as "Future Modules" in the roadmap index rather than built out.
- Created module folders under `llm-internals/`: `01-tokens-and-context`, `02-decoder-only-generation`, `03-generation-quality-and-decoding`, `04-memory-and-retrieval`, `05-assistant-behavior-and-architecture`.
- Split the monolith's 20 sections into ~16 lesson docs grouped by module; each module has a `README.md` with objectives + its slice of the appendix readings. Top-level `llm-internals/README.md` is the roadmap index (module map, future modules, consolidated bookmark list, one-paragraph summary).
- Distributed all 16 appendix readings to the module they belong to (verified none dropped). Removed the original monolith via `git rm` (recoverable from history; content fully redistributed).
- Decision: nested module folders under `llm-internals/` rather than repo root, since the whole curriculum is currently LLM-internals; agents/memory-systems become future modules.
- Verification: `find llm-internals -type f | sort` confirms 24 files (5 module READMEs + 1 index + ~18 lessons) and the original monolith is gone. Content-only change; no tests/runtime in this repo.

<!--
Each session entry follows this shape:

## YYYY-MM-DD

### Short title for what was done (model + version, effort/thinking mode, token budget if known)

- bullet describing what was added or changed
- another bullet — keep them high-level and user/agent-facing, not implementation play-by-play
- if relevant, intended profiling signal or measurement:
  - what to look for in logs / output
  - what numbers were observed
- Verification:
  - the exact command(s) that were actually run
  - what passed / what was confirmed

Model / effort / token info goes in the parentheses after the `###` title when available from the system. Use whatever the model or interface actually reports — do not estimate or hallucinate. Omit any field that the interface does not surface.

- **Model**: the version string the interface reports (e.g. `grok-4.3`, `gpt-4o`, `claude-opus-4-7`).
- **Effort / thinking mode**: the effort knob the interface reports (e.g. `high`, `low`, `extended thinking`). Omit if no such knob exists or its setting is not surfaced.
- **Token budget**: **output tokens for the session** (output + thinking/reasoning tokens for models that report them separately, e.g. Claude with extended thinking). This is the cleanest cross-agent proxy for "amount produced." Omit if the interface does not surface a count.

Purely human-driven work can use `(human)`. Mixed human + agent sessions can combine them, e.g. `(human + grok-4.3, high)`.

Keep the parenthetical compact. Examples:
- `(grok-4.3, high, ~18k out)`
- `(gpt-4o, high, ~22k out)`
- `(claude-opus-4-7, extended thinking, ~30k out)`
- `(grok-4.3, low)`

Newest entry goes on top. If the session did multiple distinct pieces of work, use multiple `###` subsections under one `##` date header.
-->
