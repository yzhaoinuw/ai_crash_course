# Work Log

Prepend new session notes to the top of this file.

Rotation policy: the live log holds at most the **5 most recent unique calendar dates**. When a new date would push the file past 5 unique dates, move the oldest 5 dates as a chunk into a new file at `work_log_archive/work_log_<earliest>_to_<latest>.md`. The live file always holds at most 5 unique dates; each archive file always holds exactly 5.

If today's date already has a `## YYYY-MM-DD` header at the top, add a new `###` session subsection under it rather than starting a second `## YYYY-MM-DD` header for the same date.

Update this log at the end of any substantive work session unless the user explicitly asks not to document it. Substantive work includes file edits, meaningful validation or debugging, technical decisions or reversals, reusable discoveries, branch/PR/release state changes, or follow-up work that future agents need. Log useful experiments even when the code was reverted; skip casual Q&A, trivial one-off commands, and pure scratch work with no future coordination value.

## 2026-05-30

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
