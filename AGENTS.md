# Guidelines and Tips for Agents

The shared treaty for any agent (Claude, Codex, or other) working on this repo.
Read it first; don't auto-read every Markdown file — use the
[Documentation](#documentation) map to pick what's relevant to the task.

Keep this file under 150 lines. Shared mechanics live in
[`treaty_docs/treaty_conventions.md`](treaty_docs/treaty_conventions.md); project-specific rules live here.

## What This Repo Is

A personal, docs-only **knowledge base** on AI / LLMs: ordered Markdown lessons, no
runtime. There is **nothing to build, run, or test** — no app, test suite, formatter,
linter, or pre-commit. "Work" here means writing and organizing notes, so skip
hunting for a test or activation command; there isn't one.

## Before Committing

- Prepend a dated entry to `treaty_docs/work_log.md` for any substantive session (see below),
  noting model + version and effort/thinking mode.
- If a lesson was added, renamed, renumbered, or re-scoped, update the root
  `README.md` topic index and the chapter `README.md` keyword index to match
  (see [Keyword and Topic Indexes](#keyword-and-topic-indexes)).
- Commit and push only when the user asks.
- Run `git diff --check` and `treaty validate .` with a current Treaty CLI;
  check relative links and anchors when document paths or headings change.
- Do not create a pull request unless the user explicitly asks.

## Keeping The Treaty Docs Current

- **`treaty_docs/work_log.md`** — at the end of a substantive session (file edits, a technical
  decision or reversal, a reusable finding, or unfinished follow-up), prepend a dated
  entry. Skip casual Q&A and trivial one-offs. The file's own header documents the
  date-rotation and archiving rules. Follow [Work Log Discipline](treaty_docs/treaty_conventions.md#work-log-discipline)
  for entry structure and check the log before an authorized push or merge too.
- **`treaty_docs/next_steps.md`** — when a session creates or resolves future work, add or remove
  items and keep the "Currently Hot" pointer accurate.

## Branching

The repo is committed directly to `main`. If you do use a feature branch, finish and
push (or explicitly park) its work before switching away — don't leave important
changes stranded locally or unverified.
See [Branch Handoff](treaty_docs/treaty_conventions.md#branch-handoff) for the shared procedure.

## Updating The Treaty

Only update the adopted template when the user asks. Upgrade the CLI first, inspect
`treaty --version` and `treaty diff`, preview with `treaty update --dry-run`, then
apply and resolve conflicts while preserving these project rules. Full procedure:
[Updating The Treaty](treaty_docs/treaty_conventions.md#updating-the-treaty).

Working Treaty docs live in `treaty_docs/` (`docs_dir: treaty_docs`); `AGENTS.md`,
`project_overview.md`, and `.copier-answers.yml` stay at the root. This repo has no
releases or pre-commit configuration; use targeted `--data` answers for new questions.

## Documentation

Read these only as needed:

- `treaty_docs/treaty_conventions.md` — upstream-maintained mechanics; read before logging,
  rotating dates, handing off branches, or updating the template. Avoid local edits.
- `project_overview.md` — course structure and authored/derived file ownership.
- `treaty_docs/work_log.md` — recent implementation history and verification breadcrumbs. Find
  date anchors with `rg -n '^## [0-9]{4}-[0-9]{2}-[0-9]{2}' treaty_docs/work_log.md` and read only
  the slice you need; older dates live under `treaty_docs/work_log_archive/`.
- `treaty_docs/next_steps.md` — unfinished work; the "Currently Hot" pointer names active threads.
- `README.md` — the reader-facing landing page and its collapsible topic index.

For any structured Markdown doc, `grep -n '^## ' <file>` gives the section map before
a targeted read.

## Commit Messages

- Short title line; add a flat bullet body only when the commit bundles several
  user-requested changes.
- Describe high-level, user-facing behavior (the notes added or changed), not tests,
  memory updates, or other behind-the-scenes work — unless that internal work is
  itself the point of the commit.

## Git Ownership

If Git warns about "dubious ownership," mark the repo safe:
`git config --global --add safe.directory "$HOME/python_projects/ai_crash_course"`.

## Note-Writing Style

This is a learning project. When writing or expanding notes:

- **Use `text`-block illustrations when they genuinely aid understanding** — a causal
  mask grid, training vs inference token-by-token, information flow through a cache.
  Prefer labeled diagrams for anything spatial or sequential, but don't add one just
  to have one. One good diagram beats three mediocre ones.
- **At the end of a substantive learning session, suggest 2–4 further readings**
  (blogs/articles over papers), each with a one-line note on what it's best for.

## Keyword and Topic Indexes

Two hand-maintained navigation indexes, curated as **canonical-location maps**, not
exhaustive keyword dumps — ripgrep already covers the long tail, so these earn their
keep only as human-browsable navigation and the "which lesson *owns* this concept?"
map grep can't give. Keep them lean.

- Root `README.md` → **Topic Index**: a collapsible map grouped by theme; the topic in
  the `<summary>`, its keywords as the list inside, each pointing to the canonical
  lesson.
- Each chapter's `README.md` → **Keyword Index**: an alphabetical keyword → owner lookup.

Maintenance rule: whenever a lesson is added, renamed, renumbered, or re-scoped, update
both indexes in the same commit. Point a keyword to where a concept is *taught*, not
every file that mentions it. If an index and the lessons ever disagree, the lessons win.
