---
name: smart-model-router
description: Route engineering work to the cheapest worker that can do it reliably. Picks a reusable worker profile — each pinning a model tier (Haiku / Sonnet / Opus), a reasoning effort, and whether it can reach other installed skills at all — then names the few skills that worker should use and hands it a compact context packet instead of a repository. Route on decision complexity, contract impact, blast radius and reversibility, never on framework, language, repository size or file count. Use at the start of any software engineering task and before delegating to any subagent: implementing a feature, fixing a bug, refactoring, reviewing code, auditing architecture or security, exploring an unfamiliar repository, planning a migration or infrastructure change, or working autonomously through a multi-step project. Also use when asked about model choice, reasoning effort, subagent strategy, which skills a worker should get, or token and cost efficiency — including "use the model router", "route this properly", "stop burning Opus". Not for casual conversation or non-engineering questions.
---

# Smart Model Router

Pick the cheapest worker that can do the work reliably. Never trade away correctness, security,
architecture, or production safety to save tokens.

Delegation here means choosing a **worker profile** — a reusable agent definition that pins a
model, an effort level, and a tool and skill boundary — then giving it a focused assignment. You
do not assemble a worker per task; you pick one of a small set and aim it.

## Start here, and usually stop here

**Is this routine work inside the existing architecture, with a cause you already understand?**
Feature work, tests, a rename, type errors the compiler already located, config, a normal
migration, an ordinary review, a bug whose cause you have identified. If yes:

> **Do it yourself. No delegation, no worker profile, no routing commentary.**

Then stop. Do not name heuristics, do not write a rationale, and do not investigate anything *in
order to justify a route* — investigate only what the work needs. A router that costs more than
the task is the waste it exists to prevent, and reciting this policy while doing a three-file
rename is the most common way it fails.

Read on only when one of these is true:

- the change **defines or alters a contract** (see below),
- a mistake would be **hard or expensive to reverse**,
- you are **diagnosing a symptom you cannot yet reproduce or verify**,
- the relevant surface is **larger than you can read**, so recon must be delegated,
- or a serious attempt already **failed**.

## The workers

Global profiles in `~/.claude/agents/`, usable from any project. Delegate with
`Agent(subagent_type: "<name>", ...)`.

| Profile | Model / effort | Skill access | For |
|---|---|---|---|
| `smart-haiku-recon` | haiku / low | **none** | where is it, what references it, how many, which check failed |
| `smart-haiku-mechanical` | haiku / low | **none** | already-decided repetitive edits, obvious renames, applying one agreed pattern |
| `smart-sonnet-worker` | sonnet / medium | yes | **the default.** Features inside existing architecture, normal debugging, tests, refactoring, integration |
| `smart-sonnet-deep` | sonnet / high | yes | difficult debugging, subtle behavior, complex refactors, important correctness review |
| `smart-opus-architect` | opus / high | none (read-only) | architecture, boundaries, invariants, contracts, concurrency and migration strategy |
| `smart-opus-gate` | opus / high | none (read-only) | focused review of finished work on a risky surface |

There is deliberately no xhigh profile: measured head-to-head on a hard contract decision,
`opus/high` caught everything that changed the outcome and xhigh added nuance at roughly twice
the output cost. When a high-effort answer seems shallow, the measured fix is a sharper packet,
not more effort.

**When no profile clearly wins, use `smart-sonnet-worker`.** It is the default, not a compromise.

The profiles are deliberately generic — the same Sonnet worker serves Laravel, React, Go, Python,
mobile and infrastructure. What specialises it is the assignment and the skills you name, not the
profile. Resist creating stack-specific variants; that path ends in dozens of unmaintainable
profiles.

## Choosing one

Three questions, in order of how much they tell you:

**1. Contract or implementation?** The strongest signal. If the change defines or alters a
contract — public API, module boundary, service interface, event schema, database invariant,
authorization rule, auth flow, tenancy boundary, persistence guarantee, state-machine transition,
backward-compatibility promise — that decision goes to `smart-opus-architect`. Work *inside* an
established contract is `smart-sonnet-worker`, however many files it touches.

**2. How expensive is being wrong?** Easily reversible (renames, styling, tests, docs) → Haiku or
Sonnet. Moderately reversible (behavior, a new endpoint, a pre-production migration) → Sonnet.
Hard to reverse (destructive migrations, live data transformation, auth or authz semantics, an
externally consumed API, data ownership, money, distributed synchronization) → Opus plan and Opus
gate. Judge blast radius, not diff size: `DROP INDEX` is short, so is `kubectl apply`.

**3. What is it asking for?** Retrieval, counting, classification and mechanical application →
Haiku. Building, fixing and testing inside known shapes → Sonnet. Deciding what must always be
true → Opus.

Framework, language, repository size and file count carry **no** routing information. Thirty files
inside one convention is routine; one method that redefines an invariant is not.

## What to send it

Three things, and nothing else.

**The assignment.** One sentence on what this worker is responsible for. If you cannot write it,
you are not ready to delegate.

**The skills it should use — named explicitly, with the file path.** A worker with skill access
can reach every installed skill, so it is *your* job to narrow that. Name the one or two that
materially help — and include the path to each `SKILL.md`, because catalogue discovery of
recently added skills is unreliable (verified: a project skill took most of an hour to appear),
and a path also works for workers with no Skill tool at all. Say which skill governs if two could
conflict. Default to naming none. Skill context is not free:
an unrelated skill costs tokens, competes for attention, and can contradict what matters. Prefer
the most specific thing that applies:

```
project-local convention  >  technology-specific skill  >  generic engineering skill
```

Orchestration, planning-methodology and routing skills — including this one — are usually
coordinator-only. A scout asked which files reference a symbol needs a precise question, not your
workflow. Never name two skills that solve the same problem; pick the one that fits.

**A compact context packet.** Objective, affected modules, the specific code that matters, known
behavior, constraints, findings, open questions, risks. **Excerpt, do not attach** — quote the
twenty lines carrying the logic, not the file, and never the directory "for completeness".
Scoping to the right files is not compression if you then paste them whole. Never hand a
repository to an expensive worker: a tight packet reasons *better*, because irrelevant context is
a distraction, not insurance. Format: `references/handoff-protocol.md`.

## Delegate only when delegating is cheaper than doing

Every worker costs a prompt, a handoff, and a summary coming back.

**Recon defaults to reading it yourself.** Delegate it only once you have confirmed the surface is
genuinely larger than you can read — check before accepting a claim that a repository is huge.
Fanning workers across a small surface costs more *and finds less*: a worker's summary is lossier
than the file, so findings die in the compression. When the surface truly does not fit, split by
**independent question** — three or four narrow ones, never ten overlapping.

Announce the route only when you actually split or escalate, in one line:

```
Route: smart-haiku-recon ×3 → smart-opus-architect (invariants) → smart-sonnet-worker (implement) → smart-opus-gate. Contract change, production-critical.
```

If you did the work yourself, that line adds nothing. Skip it.

## The gate: valuable, narrow, and easy to overuse

```
smart-sonnet-worker implements → smart-opus-gate reviews the risky surface → worker fixes
```

Deep reasoning earns its cost most reliably reviewing a finished diff for what the implementer
could not see while writing it — a bulk `UPDATE` silently depending on request-scoped state, a
non-idempotent backfill, a mass assignment dropping a field. Prefer this over routing the
implementation itself to Opus.

Calibrate it against what was measured: on planted core defects (a stale-read race, a
check-then-insert idempotency hole, an unscoped tenant query), a fresh `smart-sonnet-worker`
review caught them all — the Opus premium showed up as *additional* real findings on the risky
surface (a lost-update path, an input-validation hole, a missing auth boundary). So gate only
where breadth on a dangerous surface is worth paying for: tenancy, money, concurrency,
destructive or irreversible operations, production release. For ordinary correctness, a fresh
Sonnet review is the right gate. **Never gate a report-only or read-only task**, and never gate
work whose failure is cheap to observe and reverse.

**A gate must block.** If you spawn a review you intend to act on, wait for it and incorporate it
before you finish. A review still outstanding when you report done is not a review.

## Diagnose the failure before escalating

When an attempt falls short, find out why before spending more. Most failures are not model
failures:

| The problem was | The fix |
|---|---|
| the worker lacked facts | better, focused context — not a bigger worker |
| the assignment was vague | scope it properly, re-run the same profile |
| it lacked domain knowledge | name the relevant skill |
| it genuinely needed to think harder | step up effort: `smart-sonnet-worker` → `smart-sonnet-deep` |
| the risk is beyond this tier | step up model: → `smart-opus-architect` |

Where the model step is worth it was measured directly: on a hard concurrency contract decision,
`sonnet/high` produced a plausible answer containing a genuinely wrong load-bearing detail, while
`opus/high` on the identical packet caught it plus the landmines around it. Contract decisions
earn the model step; ordinary implementation does not — on a two-defect billing fix,
`sonnet/medium` and `sonnet/high` produced equivalent correct fixes.

Prefer effort before capability: `smart-sonnet-deep` resolves a great deal that looks like an Opus
problem, at much lower cost. `Sonnet/medium + precise context` beats `Sonnet/high + noisy context`,
and `Sonnet/medium + the right skill` often beats `Sonnet/high` with none — ask whether the worker
needs to *think* more or *know* more.

Escalate capability when evidence points at concurrency, ordering, distributed behavior, deep
architectural coupling, corrupted state, complex security behavior, or production-only causality —
or when the work has become a contract decision. "It failed once" and "it is slow" are not on that
list; a confused cheap answer usually means the question was too broad.

**A diagnosis you cannot verify deserves a second opinion.** If you cannot reproduce a symptom or
run the check that would confirm your explanation, a focused gate is cheap next to shipping a
confident wrong root cause.

## Verification, and proof instead of claims

Savings must never come out of verification. Run the tests, type checker, linter, static analysis,
build, architecture tests, security checks — whichever apply. A deterministic tool is cheaper and
more reliable than any model simulating it. If a tool cannot run, say so rather than implying it
passed.

Where completeness matters, finish with a **deterministic proof**: after a rename sweep, grep the
old name and show zero hits. Models miss sites — duplicated enums in frontend types, bare strings
in config, column defaults in migrations.

## Route autonomously

Make the call yourself and move. Do not ask which worker, effort or skills to use — that is
inferable. Ask only about genuine product or business ambiguity that reading the code cannot
settle. Explicit user instructions beat this policy, always.

## References

Each costs context; open one only when the situation calls for it.

- `references/worker-profiles.md` — what each profile is for, its boundaries, and how to add or
  change one without causing profile explosion.
- `references/skill-routing.md` — how skill access actually works here (verified), choosing skills
  per worker, coordinator-only skills, conflicts and duplication.
- `references/effort-routing.md` — effort levels, what is settable where, effort-before-model
  escalation.
- `references/routing-policy.md` — tie-breakers when a task sits on a boundary; per-tier catalogs;
  greenfield vs existing projects.
- `references/domains.md` — tier splits for backend, frontend, mobile, infrastructure, database,
  security, debugging.
- `references/handoff-protocol.md` — the packet format.
- `references/examples.md` — worked routings and an anti-pattern gallery.
- `references/frameworks/laravel.md` — optional per-framework notes. A framework name is never a
  routing signal.
