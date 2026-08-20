# Routing policy — full catalogs and tie-breakers

Read this when a task sits on a tier boundary and the heuristics in `SKILL.md` did not settle
it. The catalogs are illustrative, not exhaustive; match on the *kind of question being
asked*, never on the technology.

## Contents

- [Haiku](#haiku)
- [Sonnet](#sonnet)
- [Opus](#opus)
- [Review routing](#review-routing)
- [Boundary cases and tie-breakers](#boundary-cases-and-tie-breakers)
- [Greenfield projects](#greenfield-projects)
- [Existing projects](#existing-projects)

## Haiku

Prefer Haiku aggressively wherever the answer is retrieved, counted, classified or
mechanically applied rather than decided.

**Recon and inventory** — repository exploration and mapping, file discovery, grep and
search, locating classes / functions / routes / migrations / components, finding usages and
references, inspecting configuration, dependency inventories, package metadata, documentation
lookup.

**Classification and triage** — categorizing test failures, lint and static-analysis output,
classifying command output (what passed, what failed, which files), finding duplicated code,
naming inconsistencies, obvious omissions.

**Mechanical change** — repetitive boilerplate, obvious renames, simple documentation
updates, applying one already-decided pattern across many known sites.

**Compression before a handoff** — turning a pile of findings into the compact packet the
next model receives. This is one of the highest-leverage uses of Haiku, and the easiest to
forget.

**Not Haiku:** any final call on architecture, contracts, security posture or invariants, and
anything where "plausible-looking but wrong" is expensive. Haiku gathers the evidence for
such a decision; it does not make it.

## Sonnet

The default worker. Most engineering lands here, and reaching for another tier needs a reason
you could state out loud.

- Implementing an approved plan or design
- Feature work inside an existing architecture, across any number of files
- Endpoints, controllers, validation, serializers, DTOs, actions, services, jobs
- UI components, hooks, forms, state management, client-side routing
- Migrations that follow an already-decided schema strategy
- Tests of every kind, and normal debugging and refactoring
- Library and package integration
- Static-analysis fixes, type errors, dependency and configuration work
- Ordinary code review and ordinary security remediation
- Making separate parts of a system work together

## Opus

A scarce specialist and reviewer. The test is not "is this hard?" but "would deeper reasoning
here materially reduce a real risk?"

- System architecture; module, package and service boundaries; where responsibility lives
- Domain modeling, aggregate boundaries, the invariants that must always hold
- Contracts other components will depend on, including event schemas and public APIs
- API versioning and backward-compatibility strategy
- Database architecture, destructive or live-data migrations, schema evolution at scale
- Transaction boundaries, concurrency, races, locking, synchronization semantics
- Money, accounting and inventory invariants
- Authentication and authorization architecture, tenant isolation
- Threat modeling and difficult security review
- Difficult production bugs that have resisted a serious, evidence-backed attempt
- Major refactoring *strategy* — the strategy, not the edits
- Destructive data operations and critical release gates

Also use Opus as a **milestone gate**: Sonnet implements, Opus reviews the risky surface,
Sonnet fixes. Gate reviews are where Opus pays for itself most reliably.

Not Opus: routine implementation, even a great deal of it.

## Review routing

| Review kind | Model |
|---|---|
| Consistency, missing files, obvious omissions, mechanical checklist | Haiku |
| Correctness, maintainability, ordinary security, test adequacy | Sonnet |
| Architecture, contracts, hard security, concurrency, financial and data integrity, final production gate | Opus |

## Boundary cases and tie-breakers

**Many files, simple decisions** → Sonnet. File count is not risk.

**One file, one hard decision** → Opus for the decision, Sonnet for the edit. A single method
can hold a contract.

**A pile of ordinary type errors** → Sonnet, straight in. The compiler already located them,
so there is nothing to explore. Only a specific error that turns out to expose a design
problem in a shared type escalates — that one question, not the batch.

**A bug that only happens in production** → Haiku collects traces, logs, timings and the diff
since the last good deploy; Sonnet forms and tests hypotheses; Opus only if the evidence
points at concurrency, ordering or corrupted state.

**"Audit this large repository"** → first check whether it is actually large *for this
question*. If the relevant surface fits in your context, read it directly; parallel recon on a
small surface costs more than it saves. If it genuinely does not fit, fan out on narrow
independent questions, compress, then Sonnet or Opus depending on the subject. Naming
consistency is Sonnet. Tenant isolation is Opus. Repository size decides how much compression
you need, not which tier reasons.

**A cheap task that failed once** → retry at the same tier with a better-scoped question
before escalating. Most cheap-tier failures are bad prompts, not insufficient reasoning.

**A short command with a large blast radius** → route on the blast radius. Destructive
migrations, production infrastructure changes and data transformations get an Opus plan or
Opus review even when the diff is one line.

**Something already decided and documented** → read the document. Re-deriving a recorded
decision at Opus prices is one of the most common and most avoidable wastes.

## Greenfield projects

A new project does not mean Opus designs everything.

```
Opus    → one focused architecture pass, only where the major choices genuinely matter
          (then write the decisions into CLAUDE.md / docs/ARCHITECTURE.md)
Sonnet  → implements
Haiku   → explores and verifies
Opus    → returns only for new high-impact decisions
```

The written record is what stops you paying Opus repeatedly to rediscover the same
decisions. If a decision keeps getting re-litigated, the fix is documentation, not a bigger
model.

## Existing projects

```
Haiku   → reconnaissance: conventions, structure, prior art in sibling code
Sonnet  → understand the conventions and implement within them
Opus    → only when the requested change challenges or changes those conventions at a
          risky level
```

Existing architecture is evidence. Do not redesign a working project because a different
architecture would look cleaner — that is an Opus-priced rewrite nobody asked for. The
question is always whether *this change* needs the contract to move.
