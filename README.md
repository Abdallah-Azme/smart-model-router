# routing-policy — `smart-model-router`

A Claude Code skill that routes engineering work to the cheapest worker that can do it reliably.

Delegation here is not "pick a model per task". It is picking one of a small set of reusable
**worker profiles** — each pinning a model tier, a reasoning effort, and a tool/skill boundary —
then handing it a focused assignment and a compact context packet.

```
TASK → classify → is delegation worthwhile? → choose worker profile
     → send assignment + named skills + packet → execute → verify → escalate only if justified
```

The routing signals are decision complexity, contract impact, blast radius and reversibility.
Never framework, language, repository size, or file count.

## Install

Clone as the skill itself:

```bash
git clone https://github.com/Abdallah-Azme/routing-policy.git ~/.claude/skills/smart-model-router
```

Then install the worker profiles, which the router delegates to:

```bash
cp ~/.claude/skills/smart-model-router/agents/*.md ~/.claude/agents/
```

Agent definitions are **not** discovered mid-session — start a new Claude Code session before
expecting the profiles to be selectable. Skills, by contrast, are picked up live.

## Layout

```
SKILL.md                      the concise policy — the only file always in context
references/
  worker-profiles.md          what each profile is for, and how to avoid profile explosion
  skill-routing.md            how skill access actually works here (measured)
  effort-routing.md           effort levels, what is settable where, effort-before-model
  routing-policy.md           tie-breakers, per-tier catalogs, greenfield vs existing
  domains.md                  backend / frontend / mobile / infra / database / security / debugging
  handoff-protocol.md         the context packet format
  examples.md                 worked routings and an anti-pattern gallery
  frameworks/laravel.md       optional per-framework notes
evals/                        eval definitions used to benchmark the router
agents/                       the worker profiles (copy to ~/.claude/agents/)
```

## The workers

| Profile | Model / effort | Skill access | For |
|---|---|---|---|
| `smart-haiku-recon` | haiku / low | none | where is it, what references it, how many |
| `smart-haiku-mechanical` | haiku / low | none | already-decided repetitive edits |
| `smart-sonnet-worker` | sonnet / medium | yes | **the default** engineering worker |
| `smart-sonnet-deep` | sonnet / high | yes | difficult engineering short of a contract decision |
| `smart-opus-architect` | opus / high | read-only | contracts, invariants, boundaries |
| `smart-opus-gate` | opus / high | read-only | focused review of finished risky work |

Most work should never leave the fast path: routine changes inside existing architecture are done
directly, with no delegation and no routing commentary at all.

## What was measured, not assumed

Verified against Claude Code 2.1.226 by live probe workers and probe skills in a real session:

- The `Agent` tool exposes `model` only — **no** `effort`, `skills`, or per-call `tools`. Effort
  and tool boundaries must come from a static agent definition. This constraint is why the design
  is profile-based.
- A worker with a bare `Skill` tool can invoke **any** installed skill — global, project-local, or
  plugin-provided — chosen at run time.
- That access is not free: the grant carries the whole skill catalogue into the worker's context,
  measured at roughly **+11.8k input tokens (+82%)** per spawn on identical trivial tasks.
- Omitting `Skill` removes both the tool and the catalogue listing entirely.
- `tools: Skill(some-name)` does **not** scope in agent frontmatter — a probe so defined invoked a
  different, non-granted skill and still saw the full catalogue. The two reliable settings are
  "all skills" and "no skills"; narrow by instruction and by naming a `SKILL.md` path.
- Passing a skill's **file path** was the most reliable and cheapest handoff, and beat catalogue
  discovery, which proved unreliable for recently added skills.
- No conflict-precedence mechanism exists: two contradicting skills both load and the worker
  arbitrates by reasoning. Prefer not loading conflicting skills over hoping for a good outcome.

## Status and honest caveats

Developed with the `skill-creator` workflow across two full benchmark iterations (10 evals,
with-skill vs baseline) plus targeted follow-up probes.

- Iteration 1 showed the skill bought routing discipline at roughly a 30% token premium, with
  cost-avoidance assertions already passing in the baseline.
- Iteration 2 fixed most of that overhead and, in doing so, regressed quality below baseline in
  three evals. Those regressions were diagnosed and addressed; the fixes for them are not
  themselves fully re-benchmarked.
- No effort-versus-effort benchmark backs the effort defaults beyond the head-to-head that removed
  the xhigh profile.

Treat this as a well-grounded, partly validated policy rather than a finished, fully proven one.
