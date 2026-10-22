# Project Overview

## What This Repo Is

A personal, docs-only AI knowledge base. Readers follow ordered Markdown lessons
about LLM internals and practical AI systems. There is no application, runtime,
dependency environment, build, test suite, or CI configuration in this repository.

## Active Runtime Path

The reading path starts at the root [README](README.md), continues through the
[LLM Internals roadmap](llm-internals/README.md), then a module README and its
numbered lessons. Markdown is read directly in GitHub or a local viewer.

## Repo Structure Map

```text
ai_crash_course/
|- README.md                    # landing page and Topic Index
|- llm-internals/
|  |- README.md                 # roadmap, future modules, Keyword Index
|  |- 01-tokens-and-context/
|  |- 02-decoder-only-generation/
|  |- 03-generation-quality-and-decoding/
|  |- 04-memory-and-retrieval/
|  `- 05-assistant-behavior-and-architecture/
|- AGENTS.md                    # project guidance
|- CLAUDE.md                    # imports shared AGENTS.md
|- project_overview.md
|- treaty_conventions.md        # upstream-maintained treaty mechanics
|- next_steps.md
|- work_log.md
|- work_log_archive/
`- .copier-answers.yml           # template version and recorded answers
```

## What Looks Active vs. Legacy

The five numbered module folders are current. Module 02 includes prompt caching
and training internals as lessons 07 and 08. Future modules are proposals in the
roadmap, not implemented chapters.

The former monolith `llm-internals/context-memory-and-generation.md` was split
into these modules and removed. Historical log references to it, or to lesson 07
before training was renumbered to 08, describe past paths; do not recreate them.
Sibling chapters such as `agents/` are planned in [next_steps.md](next_steps.md).

## Authored vs. Derived

Lessons, module READMEs, both navigation indexes, `AGENTS.md`, this overview,
planning, and work-log entries are hand-maintained. Lessons are the source of
truth if a navigation index disagrees with them. Update both indexes when a
lesson is added, renamed, renumbered, or re-scoped.

`treaty_conventions.md` comes from the upstream template; refresh it with
`treaty update` rather than editing shared mechanics locally.
`.copier-answers.yml` records Copier's template pin and answers; use the CLI's
targeted answer overrides instead of hand-editing metadata. There are no
generated lesson pages, compiled artifacts, or lockfiles in this checkout.

## Tests And Fixtures

No application tests or fixtures exist. Check `git diff --check`, run
`treaty validate .` using a current installed CLI, and inspect Markdown links
and anchors after navigation changes. Treaty is maintenance tooling; its
installation does not establish an application runtime for this repo.

## User Data Expectations

No runtime inputs, datasets, or private user records are needed to read the notes.

## Practical Mental Model

Start with [AGENTS.md](AGENTS.md) for editing rules, the root
[README](README.md) for topic ownership, and the chapter
[roadmap](llm-internals/README.md) for module order and keyword ownership.
Use [next_steps.md](next_steps.md) for unfinished curriculum work.

## Questions Worth Clarifying Later

Choose the next module or sibling chapter with the user. Expand the root chapter
table of contents when a second chapter exists; preserve the current two-level
course/chapter structure and the navigation-index maintenance rule.
