# Effort routing — how hard a worker should think

Read this when deciding how much reasoning a step deserves, or when an attempt fell short and
you are choosing between more context, a skill, more effort, or a bigger model.

## Contents

- [What the environment supports](#what-the-environment-supports)
- [Defaults by class of work](#defaults-by-class-of-work)
- [Effort before model](#effort-before-model)
- [What effort cannot fix](#what-effort-cannot-fix)
- [Where to spend the deepest thinking](#where-to-spend-the-deepest-thinking)
- [Avoiding both failure modes](#avoiding-both-failure-modes)

## What the environment supports

Claude Code 2.1.226, from the shipped binary and the installed agent definitions:

- **Effort is not a parameter on an `Agent` call.** The `Agent` tool exposes `model` and not effort,
  so a one-off delegation cannot dial reasoning depth. This is the single fact that shapes the whole
  design: effort must be pinned in a reusable worker profile.
- **Agent definitions accept `effort:`.** `~/.claude/agents/<name>.md` (or a project's
  `.claude/agents/`) carries `model:` and `effort:`. The official `claude-security` plugin ships
  `effort: medium` and `effort: xhigh`, so the loader honours the field.
- **The value set is `low`, `medium`, `high`, `xhigh`** — these appear as a contiguous enum in the
  binary, described as "Persisted effort level for supported models". `Workflow`'s `agent()` also
  accepts `max`.
- **Support is model-dependent.** The binary refers to an "xhigh-capable model", so do not assume
  every tier accepts every level. If a pairing misbehaves, suspect this before suspecting the file.
- **`model: inherit` is valid**, letting a worker keep the session's tier while pinning its own
  effort.
- **`Workflow` scripts can set effort per step** via `agent(prompt, {model, effort})`. Workflows
  need explicit user opt-in, so treat this as available but not the default path.

**Verified by test:** a new agent definition is **not** picked up mid-session — a freshly
written profile was rejected with `Agent type not found`. Start a new session before expecting to
select one.

**Verified by test — effort is honoured and observable.** Each subagent's transcript
(`~/.claude/projects/<project>/<session>/subagents/agent-*.jsonl`) records `message.model` and an
`effort` field per assistant turn. Observed live: `sonnet/medium`, `sonnet/high`, `opus/high`,
`opus/xhigh`, each matching its profile's frontmatter, with visibly more thinking at higher
levels (e.g. 19.3k vs 13.5k thinking characters and ~2× output tokens for xhigh vs high on the
same task). Haiku transcripts record **no effort field**, so `effort: low` on Haiku is configured
but unconfirmed — treat Haiku effort as a no-op until shown otherwise.

**Measured effort-versus-effort results (small n — one task per comparison; directional):**
- Ordinary two-defect billing fix: `sonnet/medium` and `sonnet/high` produced equivalently
  correct fixes. High bought nothing on routine implementation.
- Hard concurrency contract decision, identical packet: `sonnet/high` was plausible but included
  a genuinely wrong load-bearing detail; `opus/high` caught it plus unplanted landmines. The
  *model* step earns its cost on contract decisions.
- Same decision, `opus/high` vs `opus/xhigh`: same outcome-relevant content; xhigh added nuance
  at ~2× output cost. xhigh is not part of the recommended policy.
- `sonnet/medium` with a focused packet beat `sonnet/high` without one on cost and time at equal
  quality, and `sonnet/medium` with the right conventions document followed 4/4 project rules
  that `sonnet/high` without it could not know (0/4). Context and knowledge beat effort.

## Defaults by class of work

| Effort | Use for | Typical pairing |
|---|---|---|
| **low** | locating files, grep, listing references, repository inventory, reading straightforward config, classifying command output, obvious mechanical edits | Haiku |
| **medium** | ordinary implementation, straightforward debugging, endpoints, components, normal tests, ordinary type fixes, refactoring inside a contract, known-pattern integration | Sonnet — the most common route by a wide margin |
| **high** | difficult debugging, implementation with subtle behavior, complex refactors, tricky migrations, important correctness review, moderately hard design | Sonnet for difficult engineering; Opus for architecture and high-risk review |
| **xhigh / max** | not in the recommended policy: measured against `opus/high` on a genuinely hard contract decision it added ~2× output cost for nuance, not outcomes. If tempted, sharpen the packet instead. | — |

Two rules keep this honest. The maximum available effort existing is not a reason to use it. And
model tier and effort are independent — `Haiku/medium` is a real and useful worker for harder
classification and compression, and `Sonnet/high` handles a great deal of work that looks at
first glance like it needs Opus.

## Effort before model

When a step needs more than it got, the cheaper move is usually more thinking at the same tier:

```
Haiku/low   → Haiku/medium    before  → Sonnet
Sonnet/medium → Sonnet/high   before  → Opus
Opus/high   → a sharper packet, not more effort (xhigh measured ~2× cost for marginal nuance)
```

`Sonnet/high` is the most under-used route in practice. A difficult caching bug across several
services, a gnarly refactor, an important correctness review — these are frequently solved
there, and jumping straight to Opus pays a capability premium for a depth problem.

## What effort cannot fix

Before raising effort at all, work out why the attempt fell short. Effort is the right answer to
exactly one of these:

| The problem was | The fix |
|---|---|
| missing facts — it had not read the relevant code | focused context, not more thinking |
| a vague assignment | scope the task, then re-run at the same setting |
| missing domain knowledge or conventions | grant the relevant skill |
| genuinely needed to think harder | raise effort |
| beyond this tier's capability, or the risk is unacceptable | escalate the model |

Two comparisons worth internalising, because both cut against the reflex to escalate:

- `Sonnet/medium` with a precise, compressed packet beats `Sonnet/high` with a large noisy one.
  Effort does not compensate for context pollution; it just reasons harder over the noise.
- `Sonnet/medium` plus the right specialised skill often beats `Sonnet/high` with none. Ask
  whether the worker needs to *think* more or to *know* more — they are different problems with
  different prices.

## Where to spend the deepest thinking

Concentrate high effort on the decision that is hard to reverse, and keep the mechanical work
that follows it cheap:

```
Haiku/low recon → compact packet → Opus/high decision → Sonnet/medium implementation
                → deterministic verification → Opus/high focused gate
```

Note what is *not* in that shape: Opus at maximum effort writing the implementation. Once the
invariants and boundaries are settled, the code that follows them is ordinary work. The same
logic applies to a large production migration — the strategy and the rollback plan earn deep
reasoning; the migration file does not.

A focused high-effort **review** of a finished diff is usually better value than a high-effort
**implementation**, because it targets exactly the failure modes the implementer could not see
while writing it.

## Avoiding both failure modes

**Inflation** — `Sonnet/high` and `Opus/xhigh` for everything. Every step above the default
needs a stated reason: a contract, an irreversible consequence, or evidence that a cheaper
setting already fell short.

**Micromanagement** — spending real deliberation choosing an effort level for renaming a
variable. For obvious work, take the default immediately and move; the routing decision must stay
cheaper than the task. If you find yourself weighing effort levels on a task the fast path in
`SKILL.md` already covers, stop and just do it.
