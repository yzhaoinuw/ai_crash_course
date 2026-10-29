# Work Log

Prepend new session notes to the top of this file.

Rotation policy: the live log holds at most the **5 most recent unique calendar dates**. When a new date would push the file past 5 unique dates, move the oldest 5 dates as a chunk into a new file at `work_log_archive/work_log_<earliest>_to_<latest>.md`. The live file always holds at most 5 unique dates; each archive file always holds exactly 5.

If today's date already has a `## YYYY-MM-DD` header at the top, add a new `###` session subsection under it rather than starting a second `## YYYY-MM-DD` header for the same date.

Update this log at the end of any substantive work session unless the user explicitly asks not to document it. **The log records decisions about the project, not the content of the work produced** — the diff already says what changed. See [Work Log Discipline](treaty_conventions.md#work-log-discipline) for what counts as substantive and what to skip.

## 2026-10-29

### Deliver the reviewed Treaty update and folder migration (Codex, GPT-6)

- The user authorized committing and pushing the reviewed v0.10.2 update and `treaty_docs/` migration to `main`. Include only the Treaty metadata, project guidance, overview, relocated working docs, and archived history. No PR was requested.
- Keep the correct relocated `../project_overview.md` reference and the paused upstream #35 follow-up; the reported template mismatch does not block this validated migration.
- The staged whitespace check caught the separator blank line at the archive's end, which the unstaged check did not inspect while the archive was untracked. Removed that trailing blank line only; all historical entry content remains unchanged.
- After delivery as `521f84e`, posted and read back the requested real-adopter follow-up on [upstream #22](https://github.com/yzhaoinuw/agent_collab_treaty/issues/22#issuecomment-6503209013): confirmed the v0.10.2 force guidance and staging handoff, explicitly distinguished pre-existing work-log edits from relocation rewrites, and left external-reference line numbers unverified because this adopter produced no such rows. Used authenticated host `gh issue comment --body-file`; the connector readback matched the submitted body exactly.
- Verification:
  - `verify_treaty_relocation.py` passed all 167 relative-link/anchor checks, prior-session and exact archive preservation, unchanged unrelated files, and absence of unresolved artifacts before staging.
  - `treaty validate .` and `git diff --check` passed. `git fetch origin` and `git rev-list --left-right --count HEAD...origin/main` confirmed the delivery baseline.

### Move working Treaty docs into the current folder layout (Codex, GPT-6)

- Adopted the current `treaty_docs/` layout through released Treaty CLI 0.10.2, keeping `AGENTS.md`, `project_overview.md`, `CLAUDE.md`, and `.copier-answers.yml` at the root. Used the documented `--force` path because the preceding template update is intentionally uncommitted; no baseline commit or stash was needed.
- Preview and apply agreed on the four moves and the rewrite counts: 13 references in `AGENTS.md`, nine in the overview, seven root links in the conventions, and one in next steps. Recorded `docs_dir: treaty_docs` through the CLI. The new, still-untracked archive chunk moved with its directory and retained its exact bytes; earlier update edits survived.
- Repaired project-owned prose that the link rewriter cannot infer: the old flat-layout sentence, the overview's directory diagram, and the root-relative meaning of planning paths in backticks. Lesson files and root reader navigation remain unchanged. Historical command paths remain as recorded at the time.
- The CLI staged its relocation and clearly named the pre-existing work-log changes it left unstaged. Restored the initially empty index after review so the combined template update and relocation remain unstaged and uncommitted for user review.
- Found and reported [upstream #35](https://github.com/yzhaoinuw/agent_collab_treaty/issues/35): the v0.10.2 conventions template omits `root_prefix` on the Work Log Discipline reference to `project_overview.md`. Relocation correctly adds `../`, so its output differs from a pristine nested render by exactly that prefix. Retained the correct relocated reference; no upstream source was edited. The connector denied issue creation; authenticated host `gh issue create --body-file` succeeded, and the posted body was read back.
- Verification:
  - `treaty relocate . --to treaty_docs --dry-run --force`; `treaty relocate . --to treaty_docs --force` both exited 0.
  - `treaty validate .`; `git diff --check`; `verify_treaty_relocation.py` passed after the documentation repairs: 167 relative links/anchors, exact archived history, preserved prior session text, unchanged lesson files, and an empty Git index. `treaty diff .` identifies the single conventions-section difference described in #35; `compare_relocated_conventions.py` renders the pinned template separately and confirms its exact one-prefix cause.

### Assess the incremental Treaty v0.10.2 update (Codex, GPT-6)

- Chose the released CLI and template v0.10.2 after reading release notes and existing upstream discussions. Clean `main` and `origin/main` matched `ad1549d`; retained the flat layout and all recorded answers. Installed the CLI in an isolated temporary environment, not a project runtime.
- Preview and apply both exited 0 with no conflicts and the same two content changes: `.copier-answers.yml` and `treaty_conventions.md`. Preview preserved every tracked file's SHA-256. Preview separately reported a newline-only `CLAUDE.md` rewrite; apply did not leave one. The real staging hint named only the two content paths. No update-time manual merging or scaffold cleanup was needed.
- Assessment: the maintained-conventions split made this incremental upgrade small and reviewable. The full preview diff and release notes explained the changes before apply. Existing project-specific guidance, overview, planning content, lesson files, and historical log were preserved by the update. This run does not re-test older conflicting migrations or relocation.
- In a disposable copy, confirmed v0.10.2 rejects a dotted-version anchor with the correct suggestion; corrected dotted-version, underscore, and repeated-hyphen anchors pass; collapsing `background--paused` produces the right suggestion. A Currently Hot link to a thread under Background / Paused warns without failing. These are controlled diagnostics, not defects observed in this repo's original planning file.
- Replaced the two informal Currently Hot references with checked anchor links and removed the stale upstream follow-up: issue #23 is now closed after field confirmation of the warnings. Adding today's sixth date rotated the five previous dates together into `work_log_archive/work_log_2026-05-30_to_2026-10-22.md`, retaining the historical blocks exactly.
- Posted and read back follow-ups on [#32](https://github.com/yzhaoinuw/agent_collab_treaty/issues/32#issuecomment-6502604008) (real preview/apply evidence) and [#34](https://github.com/yzhaoinuw/agent_collab_treaty/issues/34#issuecomment-6502606593) (additional Windows anchor cases). The connector denied comment writes; authenticated host `gh issue comment --body-file` succeeded after verifying the account and repository. No distinct new defect found. Leave all local edits uncommitted for review; commit, push, and PR creation were not requested.
- Verification:
  - `git fetch origin`; `git rev-list --left-right --count HEAD...origin/main` returned `0 0`.
  - `treaty --version`; `treaty validate .`; `treaty diff .`; `treaty update . --dry-run --diff --data has_releases=false --data uses_precommit=false --data 'verification_command=git diff --check'`; same apply without `--dry-run --diff`.
  - `check_treaty_diagnostics.py` passed four released-CLI diagnostic cases in an external disposable copy; `diagnostic-results.json` records outputs. Python 3.11.11, Copier 9.19.1, Typer 0.27.3, PowerShell, `core.autocrlf=true`.
  - Final `treaty validate .`, `git diff --check`, and `verify_treaty_docs.py` passed. `treaty diff .` reports all seven maintained-conventions sections untouched; expected project-owned guidance/overview drift remains. Git index is empty; no conflict markers, rejects, or unmerged entries remain.
  - Initial sandbox restrictions on FETCH_HEAD, ensurepip/pip temporary files, and Copier cache/temp directories cleared on focused host retries; these were environment friction, not Treaty defects.

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
