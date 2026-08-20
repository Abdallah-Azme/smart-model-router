# Skill routing — giving a worker the right skills, and nothing else

Read this when you are about to delegate and want to decide what a worker should know.

## Contents

- [What the environment supports](#what-the-environment-supports)
- [How to grant a subset](#how-to-grant-a-subset)
- [The selection rule](#the-selection-rule)
- [Coordinator-only skills](#coordinator-only-skills)
- [Worked bundles](#worked-bundles)
- [Conflicts and duplication](#conflicts-and-duplication)
- [Project-local versus global](#project-local-versus-global)

## What the environment supports

Claude Code 2.1.226. The items marked **tested** were exercised with live probe workers and probe
skills in this environment; the rest come from the shipped binary and the installed definitions.

**Tested — a worker with bare `Skill` reaches everything.** A probe worker whose `tools:` included
`Skill` invoked a global skill, a **project-local** skill, and a plugin skill
(`superpowers:using-superpowers`) successfully, choosing them at run time. There is no need to
pre-bind a skill to a worker in order for it to use one.

**Tested — that access is not free, and the cost is measured.** The same worker was shown
**~217 skills** in its available list. A skill grant carries the whole catalogue of names and
descriptions into the worker's context, whether or not it uses any of them. Measured on an
identical trivial task at identical model and effort: **26.2k vs 14.4k first-turn input tokens —
~11.8k tokens (+82%) per spawn** is the price of the catalogue.

**Tested — omitting `Skill` removes both ability and listing.** A probe worker with
`tools: Read, Grep, Glob` had no `Skill` tool and reported its visible skills as "none listed".
Omission is the cheapest and most reliable way to keep skill context out of a worker.

**Tested — skills are discovered live but unreliably; agents are not discovered at all.** Some
newly written `SKILL.md` files (global and project-local) appeared in the available-skills list
within seconds; another took most of an hour and was invisible to three consecutive workers that
were explicitly told to invoke it. A newly written agent definition was rejected with `Agent type
not found`. Consequences: a new worker profile needs a new session, and a newly added skill must
be handed over **by file path**, never by name alone. Also verified: fresh sessions discover
project skills by walking up parent directories from the working directory.

**Tested — naming formats differ by source.** Plugin skills use `plugin:skill`
(`superpowers:brainstorming`, `claude-security:scan`); others are bare (`gsd-add-tests`). Pass the
exact name.

**From the binary — `Skill(<name>)` is a real permission form.** The CLI maps a `skills` option into
allowed-tool entries of the form `Skill(<name>)`, with `"all"` becoming an unrestricted `Skill`.
Sibling options in that table are `model`, `fallbackModel`, `tools`, `disallowedTools`,
`allowedTools`.

**From the definitions — bind through `tools:`, not a `skills:` key.** No `skills:` frontmatter key
appears in any of the 337 installed agent definitions, and the binary exposes no `allowed-skills`,
`preloadSkills` or `inheritSkills`. Scoped tool entries are in active use elsewhere
(`Agent(claude-security:explore)`, `Workflow(claude-security:scan)`), and the official
`claude-security` plugin ships `model:` and `effort:` in agent frontmatter.

**Now verified — `Skill(<name>)` scoping does NOT restrict.** A probe agent defined with
`tools: Read, Glob, Skill(smr-test-alpha)` (exercised in a fresh session) invoked a different,
non-granted skill without error, and reported the full catalogue (~203 entries by its own count)
in its context. In agent frontmatter the scoped form degrades to unrestricted `Skill`: no
invocation restriction, no catalogue trim, no context saving. **All skills** and **no skills**
are the only two real settings; narrow by instruction in between.

**Verified — the path handoff is the best of both.** A worker told to `Read` a specific
`SKILL.md` by path followed 4/4 planted, non-guessable conventions — at *lower* total cost than
both a skill-enabled worker and a higher-effort worker without the document — and it works
identically for workers with no `Skill` tool. Include the path whenever you name a skill.

## How to control what a worker knows

Given the above, there are three reliable settings and you choose between them at *profile* level,
then narrow by instruction at *call* level.

**No skill access** — the cheapest. Omit `Skill` from `tools:`. Nothing is listed, nothing can be
invoked. This is what `smart-haiku-recon` and `smart-haiku-mechanical` do, and it is right whenever
the work is retrieval or an already-decided edit.

**Full access, narrowed by instruction** — the normal case for engineering workers. Include `Skill`,
then name in the assignment the one or two skills that matter:

```
Use the project's own conventions skill (project-conventions) and nothing else. If it conflicts
with any general guidance you have, the project skill governs.
```

This works because the worker can reach anything but follows what it is told. It is the practical
substitute for per-skill profiles, and it costs the 226-entry catalogue.

**One document, no catalogue** — the middle ground. Use a no-skill profile and point it at a path:

```
Read C:\Users\abdallah\.claude\skills\project-conventions\SKILL.md and follow it for this task.
```

Cheap and precise, at the cost of losing whatever the skill's own progressive disclosure would have
loaded.

Inside a `Workflow` script a profile can be selected per step with
`agent(prompt, {agentType: "smart-sonnet-worker"})`, and model or effort overridden there when a
step genuinely needs something the profile does not offer.

## The selection rule

For each worker, in order:

1. **What exactly is this worker responsible for?** Write the one-sentence assignment first.
   Skill choice follows the assignment; if you cannot state the assignment, you are not ready to
   pick skills.
2. **Which skills materially help with *that* responsibility?** Not the parent task — the
   worker's slice of it.
3. **Would the skill save work, or only add context?** A skill that encodes conventions the
   worker would otherwise have to discover saves work. A skill that restates what you already
   put in the handoff does not.
4. **Does the worker need it, or do only you need it?** See below.
5. **Cut anything left over.** Default to none.

## Coordinator-only skills

Some skills exist to help *you* decide and sequence, and are dead weight in a worker:

- routing and orchestration policy — including this skill; a worker does not re-route its own
  assignment
- planning and project-management methodology, roadmap and phase workflows
- review-process and reporting frameworks the worker is not producing

A Haiku scout asked "which files reference `PaymentStatus`?" needs a precise question and
read-only tools. Handing it a planning framework buys nothing and gives it instructions that
compete with its actual job.

The inverse also holds: do not assume that because you are running a broad workflow, every child
needs all of it. Split it by role — the planning part goes to the planning worker, the execution
part to the implementer, and the scout gets neither.

## Worked bundles

For "implement inventory reservation during checkout":

| Worker | Model / effort | Skills granted | Why |
|---|---|---|---|
| recon scout | haiku / low | none | needs a question and read-only tools, nothing more |
| architecture | opus / high | domain or architecture skill; the project's own architecture doc | the decision is a contract, and the project's rules outrank generic advice |
| implementer | sonnet / medium | framework skill + testing skill | encodes conventions worth not rediscovering |
| gate | opus / high | security or data-integrity review skill | one lens on the risky surface only |

Note each worker gets something different, and two of the four get one skill or none.

Counter-example — a worker that has been given a planning framework, a frontend skill, a
Playwright skill, a Cloudflare skill and a GitHub skill in order to add a database index. Every
one of those is context it must read past to find the one thing it needs.

## Conflicts and duplication

Two granted skills can disagree, and the worker has no principled way to arbitrate. Prevent it
rather than hoping:

- **Never grant two skills that solve the same problem.** Two testing workflows, two planning
  methodologies, two review checklists — choose the one that fits the task. If you genuinely
  cannot choose, that is a signal you have not scoped the assignment tightly enough.
- **Do not run two prescriptive workflows at once.** If two skills each prescribe an end-to-end
  process, running both produces a hybrid that follows neither. Pick the workflow aligned with
  the task and say in the assignment which one governs.
- **State precedence when overlap is unavoidable.** One line in the handoff — "follow the
  project's module rules where they conflict with the generic guidance" — costs nothing and
  removes the ambiguity.
- **Watch for a skill that contradicts the packet.** Your handoff carries the decisions already
  made. A skill that would re-open them is either the wrong skill or a sign the decision was not
  actually settled.

## Project-local versus global

Global (personal) skills live under `~/.claude/skills/`; project skills live under
`.claude/skills/` in the repository. Both are discovered, and the project ones are more specific
to the work in front of you.

Order of precedence when they overlap:

```
project-local convention  >  technology-specific skill  >  generic engineering skill
```

A project skill describing this codebase's module boundaries beats a general architecture skill
on any question about *this* codebase. Generic advice must never override an explicit project
constraint — if a global skill says "prefer a service layer" and the project says "logic lives in
Actions", the project wins, and the worker should be told so plainly.
