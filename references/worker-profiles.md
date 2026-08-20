# Worker profiles

The router delegates by picking one of a small set of reusable workers, not by assembling a
bespoke one per task. Definitions live in `~/.claude/agents/` (global, usable from any project);
a project may add its own under `.claude/agents/`.

## Contents

- [Why profiles instead of per-task assembly](#why-profiles-instead-of-per-task-assembly)
- [The six](#the-six)
- [Skill access is the real boundary](#skill-access-is-the-real-boundary)
- [Adding or changing a profile](#adding-or-changing-a-profile)
- [Avoiding profile explosion](#avoiding-profile-explosion)

## Why profiles instead of per-task assembly

Because the runtime does not allow per-task assembly. Verified in Claude Code 2.1.226: the `Agent`
tool accepts `model` but exposes no `effort`, no `skills` and no per-call `tools`. Reasoning effort
and tool/skill boundaries come from the agent definition file, which is static.

So a worker's shape is fixed at definition time and only its *assignment* varies at call time.
That is the constraint the whole design follows: keep the number of shapes small, and put all the
task-specific variation — including which skills to use — into the assignment text.

## The six

| Profile | Model | Effort | Tools | For |
|---|---|---|---|---|
| `smart-haiku-recon` | haiku | low | Read, Grep, Glob | narrow factual questions about a codebase |
| `smart-haiku-mechanical` | haiku | low | Read, Grep, Glob, Edit, Write, Bash | already-decided repetitive changes |
| `smart-sonnet-worker` | sonnet | medium | + Skill | the default engineering worker |
| `smart-sonnet-deep` | sonnet | high | + Skill | difficult engineering short of a contract decision |
| `smart-opus-architect` | opus | high | Read, Grep, Glob | contract-level decisions |
| `smart-opus-gate` | opus | high | Read, Grep, Glob | focused review of finished risky work |

Two deliberate asymmetries are worth noticing.

**The Haiku pair has no `Skill` tool.** Mechanical work needs a precise instruction, not a
workflow — and the omission also keeps a large amount of skill context out of the worker entirely
(see below).

**The Opus profiles are genuinely read-only: `Read, Grep, Glob` and nothing else.** An earlier
revision gave them `Bash` and `Skill`; a live probe showed `Bash` let a "no write tools" gate
create files freely (the restriction was advisory, not real), and no Opus run ever invoked a
skill while each spawn paid the full catalogue in expensive input tokens. They decide and review;
the coordinator runs deterministic verification and hands them its output, and names the path of
any document they must read. This also forces the cheap implementation step to actually happen on
a cheap worker.

**There is no xhigh profile.** `smart-opus-max` (opus/xhigh) was benchmarked head-to-head against
`smart-opus-architect` (opus/high) on a genuinely hard concurrency contract decision: both caught
every outcome-relevant issue, including landmines the recon packet omitted; xhigh added extra
nuance at roughly double the output tokens. That is not a profile's worth of value. If a
high-effort answer seems shallow, sharpen the packet — measured, packet quality moved outcomes
more than effort did. (The definition is preserved under `~/.claude/skill-backups/` if a genuinely
exceptional case ever wants it back.)

## Skill access is the real boundary

Verified by direct test in this environment:

- A worker whose `tools:` includes a bare `Skill` can invoke **any** installed skill, chosen at run
  time — global, project-local, and plugin-provided alike. A probe worker invoked a global skill, a
  project-local skill and `superpowers:using-superpowers` successfully.
- That worker was shown **~217 skills** in its available list. Skill access is therefore not
  free: the grant carries the full catalogue of skill names and descriptions into the worker's
  context. Measured on identical trivial tasks at the same model and effort, the catalogue cost
  the child **~11.8k extra input tokens (+82%)** per spawn.
- A worker whose `tools:` omits `Skill` has **no skill tool and no skill list at all** — a probe
  with `Read, Grep, Glob` reported "none listed". Omitting `Skill` is the cheapest and most
  reliable way to keep skill context out.
- **Catalogue discovery of newly added skills is unreliable mid-session** — three probe skills
  appeared immediately, a fourth took most of an hour. The robust handoff is the skill's file
  path: a worker told to `Read <path to SKILL.md>` followed 4/4 planted conventions, at lower
  cost than a skill-enabled worker, with no catalogue in context. Name the path alongside the
  skill name in every assignment.

Two consequences for how you use these profiles:

1. **Naming skills is your job, not the worker's.** Because a skill-enabled worker can reach
   everything, "give it the right skills" means *telling it which ones to use* in the assignment.
   An unnamed worker will either ignore the catalogue or pick from it unpredictably.
2. **Prefer a no-skill profile when no skill is needed.** Routing recon to `smart-haiku-recon`
   rather than a Sonnet worker saves the model cost *and* the skill catalogue.

Naming formats differ by source: plugin skills use `plugin:skill` (`superpowers:brainstorming`,
`claude-security:scan`), while others are bare (`gsd-add-tests`). Use the exact name.

**Now verified — `tools: Skill(some-skill-name)` does NOT scope.** A probe agent defined with
`Skill(smr-test-alpha)` successfully invoked a *different*, non-granted skill, and its visible
catalogue was the full list (~203 entries by the worker's own count), not one entry. In agent
frontmatter the scoped form degrades to an unrestricted `Skill` grant: no invocation restriction,
no catalogue trim, no context saving. The two reliable settings really are "all skills" and "no
skills"; narrow by instruction (and by `Read <path>`) in between.

## Adding or changing a profile

Frontmatter that is known to work:

```yaml
---
name: my-worker
description: When to use this worker, in enough detail that the router can pick it.
model: sonnet          # opus | sonnet | haiku | inherit
effort: medium         # low | medium | high | xhigh  (support is model-dependent)
tools: Read, Grep, Glob, Edit, Write, Bash, Skill
---
```

`model: inherit` keeps the session's tier while still pinning the worker's own effort — useful when
you want the caller's capability with a different depth.

The body is the worker's system prompt. Two things belong there: what it is responsible for, and
what it should refuse to do. The second matters more than it looks — a worker that says "this
assignment needs a design decision I was not given" is far more useful than one that quietly
invents the decision.

**A new or edited definition is not picked up mid-session.** Start a new session before expecting
to select it. Skills, by contrast, *are* discovered live.

## Avoiding profile explosion

The temptation is a profile per stack: `sonnet-laravel-pest`, `sonnet-react-playwright`,
`sonnet-node-security`. Resist it. That set grows combinatorially, every entry duplicates the same
engineering guidance, and each one drifts separately.

The generic profiles already handle every stack, because what makes a worker effective on Laravel
or React is the assignment and the named skills — both of which vary per call for free. Adding a
profile is justified only by a genuinely different **shape**, not a different technology:

- a different model or effort combination you actually need,
- a different tool boundary (no writes; no skill access; network access),
- a different *responsibility* (deciding versus implementing versus reviewing).

`smart-opus-gate` earns its place because reviewing is a different responsibility from deciding,
with a different tool boundary. A hypothetical `smart-opus-gate-laravel` would not: same shape,
same tools, same job — the difference belongs in the assignment.

If you find yourself wanting a stack-specific profile, what you probably want is a **project skill**
describing that project's conventions, which any of these workers can then be told to use.
