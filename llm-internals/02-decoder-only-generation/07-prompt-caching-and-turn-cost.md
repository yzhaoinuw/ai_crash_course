# Prompt Caching: Why Re-Sending the Whole Context Is Cheap

Because the model is [stateless](01-stateless-next-token.md), every turn of a chat
re-sends the **entire** context — system prompt, tools, all prior messages and
tool results, plus the new message. The model remembers nothing between requests,
so nothing can be left out.

That should raise an alarm: if a long session re-sends 100k tokens on every single
turn, isn't each turn re-paying to process all of it? For a naive server, yes.
**Prompt caching** is the trick that makes it cheap.

## It is the same idea as the KV cache, one level up

The [KV cache](04-kv-cache.md) already solved this *within a single generation*:
process the prompt once, store the key/value states, and never recompute them,
because a decoder-only model's causal masking makes an immutable prefix's states
**provably unchangeable**. Its parting rule: *"caches are tied to an exact
prefix — if anything early changes, everything after it must be recomputed."*

Prompt caching applies that exact rule **across requests** instead of within one:

```text
KV cache        →  reuse the prefix's states across DECODE STEPS   (one answer)
Prompt caching  →  reuse the prefix's states across API REQUESTS   (turn to turn)
```

The serving system keeps the processed prefix around after your turn ends. On the
next turn, it matches the **start** of the new request against what it already
processed and skips re-running that part. The match is exact and there is no
per-message caching — it is a single contiguous prefix.

## The prefix rule: append is cheap, editing early is not

This is the whole game. Because a cache hit requires the prefix to match exactly:

- **Appending** to the end (a new user message, a tool call, a tool result) leaves
  everything before it identical → the prefix is reused, only the new tail is
  processed.
- **Changing anything early** (swap the model, edit the system prompt, trim or
  summarize an old message) makes every token after the change a cache miss → it
  is reprocessed from that point.

So deployed assistants deliberately order the request from least- to
most-volatile, and only ever add to the end:

```text
┌──────────────────────────────────────────────┐
│ system prompt + tool definitions   (rarely changes)  ─┐
│ project / user context             (per session)      │ CACHED PREFIX
│ conversation so far                (grows each turn)  ─┘  (reused)
├──────────────────────────────────────────────┤
│ >>> new message / tool result <<<  (this turn only)  ─── fresh work
└──────────────────────────────────────────────┘
```

As long as new content lands at the bottom, the expensive top is a cache hit.

## What a cached turn actually costs

Cache reads are not free, but they are cheap. Rough economics (exact numbers vary
by provider and change over time):

```text
cache READ    ≈  10%  of the normal input-token price   ← the reused prefix
new input     =  100% of the normal input-token price   ← only the new tail
output        =  100% (normal output price)             ← the model's reply
cache WRITE   ≈  125% of the normal input price          ← first time a prefix is stored
```

So on a warm turn you pay ~10% for the giant prefix plus full price for the small
new tail — not full price for the whole history. Writing to the cache costs a bit
*more* than plain input (a one-time premium), which is why the first turn on a new
prefix is the slow, expensive one and every turn after it is cheap.

## Warm vs. cold: the TTL

A cached prefix is kept only for a while after the last use — a **time-to-live
(TTL)** — and each hit resets the timer, so it stays warm as long as you keep
working. Walk away longer than the TTL and the prefix goes cold; the next turn
must re-process and re-write the whole thing (at the write rate) before it is warm
again.

```text
turn ──hit──▶ turn ──hit──▶ turn      · · · idle > TTL · · ·     turn
  warm         warm         warm                                COLD
(10% reads)  (10% reads)  (10% reads)                    (full re-write of prefix)
```

Providers offer different TTLs (commonly a short ~5-minute window and a longer
~1-hour one). Concretely, Claude Code requests the 1-hour TTL on a subscription
(where usage is plan-included, so the longer window is free) and falls back to
5 minutes on metered API-key usage.

## Applied: why a permission or authorization hiccup barely costs tokens

This is where the theory pays off for anyone using a tool-using agent — Claude
Code, a Codex-style CLI, or a custom agent loop. A common worry:

> When the agent hits a permission wall (a sandbox blocks a command, a tool needs
> authorization) and then retries after I approve, does it re-read my entire
> context and waste a pile of tokens?

**No — because the retry is just another message appended to the conversation.**
It does not touch the cached prefix, so the "re-read" is a ~10% cache read, not a
reprocess. Concretely, when a sandboxed command fails, Claude Code retries it
unsandboxed (its `dangerouslyDisableSandbox` escape hatch), and that retry goes
through the normal permission prompt. Mechanically it is one more tool call at the
bottom of the request:

```text
inference A:  agent runs a command (sandboxed) ──────────┐
   tool result: "blocked by sandbox"  ── appended        │  prefix above all
inference B:  agent retries the command (escalated) ─────┤  of this is READ
   ▸ permission prompt → you approve  (no inference)      │  FROM CACHE (~10%)
   tool result: success               ── appended        │
inference C:  agent continues ───────────────────────────┘
        └── only the NEW appended lines are full-price input
```

Nothing in that flow changes the model, the system prompt, or the tool
definitions, so the prefix stays warm the entire time. The escalation adds roughly
**one extra model inference** (inference B) versus if the command had worked the
first time — a cache-warm turn, the same cost profile as any normal step.

The real cost of a permission hiccup is therefore **not tokens — it is momentum**:

- your attention gets pulled back to click "approve," breaking flow, and
- the agent is blocked on wall-clock time until you respond.

The one exception is the TTL: if you leave the prompt sitting past the cache
window, the *resume* pays a full cold re-write — but that is the idle gap doing
it, not the permission itself. The lever that matters is reducing how *often* you
are interrupted (pre-allowing trusted paths/domains, sandbox auto-allow modes),
not optimizing tokens.

## Key takeaways

- Statelessness forces the full context to be re-sent every turn; prompt caching
  is what stops that from being ruinously expensive.
- It is the KV-cache principle across requests: an **immutable prefix** can be
  reused; a cache hit needs the prefix to match **exactly**.
- **Append is cheap** (~10% cache read); **editing anything early** is a full
  reprocess from that point. Model/effort/system-prompt changes bust the cache.
- An agent's permission or sandbox-escalation prompt appends to the conversation,
  so it is cache-safe. Its cost is paused momentum and latency, not tokens —
  unless you idle past the TTL.

## Where this connects

- The mechanism it builds on: [prefill vs decode](03-prefill-vs-decode.md) (the
  prefix's states come from prefill) and [the KV cache](04-kv-cache.md) (the same
  immutable-prefix logic within one answer).
- Why the prefix is expensive to (re)build in the first place:
  [why long context is expensive](06-long-context-cost.md).
- The layers doing the appending — prompt construction, the tool layer, and the
  safety/policy layer that gates a permission prompt — are mapped in
  [Module 05: the layers of a deployed assistant](../05-assistant-behavior-and-architecture/02-layers-of-an-assistant.md).
  The agent side is expanded in the "Agent architecture" future module in the
  [roadmap](../README.md#future-modules-not-yet-built-out).
