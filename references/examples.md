# Worked routings

Calibration examples across stacks. The point of each one is the *reasoning* — which contract
it touches and how reversible it is — not the arrow diagram. Note how many involve no Opus at
all, and how the ones that do use it for a single narrow question.

## Trivial — no delegation at all

> "Rename this DTO and update its imports."

```
Route: Sonnet, directly. No subagents.
```

Inside an existing contract, trivially reversible, and the one decision was already made by
the user. A recon step would cost more than the grep it replaces, and the handoff would be
longer than the diff. Do it, run the type checker and tests, then grep the old name to prove
zero hits remain. Orchestration here is pure overhead.

## Small — cheap recon, then build

> "Add a new allowed status filter to `GET /api/v1/products`."

```
Route: Haiku (locate the controller, request and existing filter conventions)
     → Sonnet (implement + test). No Opus — inside an existing API contract.
```

If the conventions are already known from earlier in the session, skip the recon too.
Re-deriving known context is exactly the waste this policy exists to prevent.

## Medium — conventions first, then build

> "Add a Products module API endpoint with query filtering, permissions and pagination."

```
Route: Haiku (read a sibling module for conventions: query building, permission naming,
       pagination shape, test layout)
     → Sonnet (implement endpoint, policy, resource, tests)
     → verification: test suite + static analysis.
```

Several files, no new contract: every decision is "match what this codebase already does".
Opus enters only if the endpoint must expose something across a module boundary that does not
exist yet — that would be a new contract.

## Cheap fix — resist the escalation reflex

> "Fix these five TypeScript type errors."

```
Route: Sonnet, directly. Then re-run tsc.
```

No Opus, and no recon agent: the compiler already reported file and line, so there is nothing
to explore. The compiler is the deterministic tool that replaces model reasoning here. If one
error turns out to expose a genuine design problem in a shared type, that single question can
escalate — not all five.

## Frontend — volume is not complexity

> "This dashboard has 40 components and re-renders constantly. Make it fast."

```
Route: Haiku (inventory the components, locate the state providers and the API call sites,
       find which values live in context)
     → Sonnet (memoization, query keys, splitting providers, fixing the actual hot paths)
     → verification: profiler numbers before/after, plus the test suite.
```

Forty components is not an architecture problem. It becomes one only if the fix requires
changing *where application state lives* — that is a contract, and that question goes to Opus
while the implementation stays with Sonnet.

## Infrastructure — route on blast radius

> "Add a `zero-downtime` deploy step to the CI pipeline and cut over the production database
> to the new instance this weekend."

```
Route: Sonnet (the CI workflow file — ordinary configuration work)
     + Haiku (inventory current deploy definitions, env vars, health checks)
     → Opus (the cutover plan: ordering, replication lag, rollback point, what happens to
       in-flight writes) — irreversible, production-critical
     → Sonnet (implement the runbook steps)
     → Opus (review the runbook before it is executed).
```

Two different tiers in one request, because it is two different tasks. The YAML is cheap; the
cutover cannot be undone.

## Database — the diff is one line, the risk is not

> "Drop the `legacy_price` column, we backfilled `price_cents` months ago."

```
Route: Haiku (grep every reference to legacy_price across code, queries, views, exports,
       analytics, and check whether any environment still writes it)
     → Opus (is this safe, in what order, and with what rollback?) — destructive and
       irreversible against production data
     → Sonnet (write the migration and the guard).
```

A one-line migration earns Opus attention here purely because the data does not come back.

## Bug hunt — escalate one rung, only if earned

> "Checkout intermittently fails in production, works locally."

```
Route: Haiku (collect failing traces, recent deploys, relevant log lines, which requests
       fail and when) → compress
     → Sonnet (form hypotheses, reproduce, test them)
     → Opus ONLY IF it survives that and the evidence points at ordering, locking or a race.
```

The escalation is conditional on evidence, not on frustration. Most intermittent failures are
a timeout, a missing index or an environment difference — all Sonnet work.

## Complex — Opus for the decision, Sonnet for the code

> "Design inventory reservation behavior during checkout with concurrent orders."

```
Route: smart-haiku-recon (inventory the current reservation and order paths, transactions,
       locks, existing tests) → compress into a handoff packet
     → smart-opus-architect (define invariants, transaction boundaries, concurrency strategy;
       tell it to use the project conventions skill)
     → smart-sonnet-worker (implement, including a contention test; name the framework and
       testing skills)
     → deterministic verification (tests, static analysis)
     → smart-sonnet-worker (fix ordinary issues)
     → smart-opus-gate (review the risky surface only — blocking, wait for it)
```

This is the full shape: six steps across four profiles, and the recon worker has no skill access
at all. The two Opus steps decide and review; neither writes implementation, and neither ever sees
the repository — only the packet. Most tasks should not look remotely like this.


## Security audit — Opus, but only after compression

> "Audit this multi-tenant authorization architecture for possible tenant data leakage."

```
Route: Haiku ×3 in parallel (policies and gates; global scopes and query paths; jobs,
       events, exports and other background paths that may bypass scoping)
     → consolidate into one packet: where tenancy is enforced, where it is not
     → Opus (reason about the gaps, rank by exploitability)
     → Sonnet (fix), then verify with tests that assert cross-tenant denial.
```

Opus is right for the reasoning — tenancy leaks are the definitive "expensive and hard to
reverse" case. Opus is wrong for the file-reading, which is most of the work by volume.
Background paths get their own worker because they are the classic place scoping is missed.

## Large repository audit — compression is the whole trick

> "Audit this very large repository's order architecture."

```
Route: Haiku ×3–4 in parallel, each with one narrow question (order state machine and
       transitions; who writes order rows and from where; cross-module calls in and out of
       Order; order tests and what they actually assert)
     → consolidate: a map, plus specific smells with file:line
     → Opus (architecture judgment on the compressed map)
     → Sonnet (whatever remediation is agreed).
```

The failure mode to avoid is handing Opus the repository and asking it to audit. It would
spend most of its budget reading files Haiku reads for a fraction of the cost, and reason
worse for the noise.

**But check the premise before fanning out.** "Large repository" is a claim to verify, not
accept. If the order-related surface turns out to be a dozen files you could read directly,
read them — the fan-out costs four prompts and four summaries to learn what two greps would
have told you. The parallel shape above earns its keep when the surface genuinely exceeds one
context, not whenever the word "audit" appears.

## Mechanical sweep — almost all Haiku

> "We renamed `ProductStatus::ARCHIVED` to `RETIRED`. Update everything."

```
Route: Haiku (find every usage, including bare strings in config, migration defaults,
       seeders, tests, and any duplicated copy of the enum in frontend types)
     → Sonnet (apply, or review the mechanical apply)
     → verification: tests + type check + grep for the old name returning zero hits.
```

The final grep matters: it is a deterministic proof of completeness that costs nothing and is
more trustworthy than any model claiming it caught every site. The duplicated frontend copy is
the site that gets missed — worth asking about explicitly.

## Greenfield — one architecture pass, then get cheap

> "New project: multi-tenant SaaS, needs offline-capable mobile clients."

```
Route: Opus (one focused pass: tenancy model, sync and conflict-resolution strategy, module
       boundaries) → write the decisions into docs/ARCHITECTURE.md
     → Sonnet (build everything)
     → Haiku (explore, verify, inventory)
     → Opus (returns only for the next genuinely new high-impact decision).
```

The document is the point. Without it, every session re-litigates the same choices at Opus
prices.

## Anti-pattern gallery

**Opus recon.** An Opus subagent asked to "explore the codebase". It will do it well and the
bill will be absurd. Haiku, every time.

**The orchestration ladder on a small task.** `Haiku → Sonnet → Haiku → Opus` to add a
nullable column. The handoffs cost more than the change.

**Escalating on one failure.** A cheap worker returns something confused, and the next move is
Opus. Usually the question was too broad; re-ask it narrowly at the same tier first.

**Ten overlapping workers.** Four workers reading the same three files to answer variations of
one question. Split by independent question, or use one worker.

**Fan-out on a small surface.** Spawning parallel recon workers across a repository you could
have read yourself. Four prompts plus four summaries to learn what two greps would have shown,
and the summaries are lossier than the files. Delegate recon when the surface exceeds your
context, not when the task sounds big.

**Paying the router more than the work.** Reading the whole policy and its references to decide
that a three-file rename is a three-file rename. Routine work inside the existing architecture
takes the first exit: do it, Sonnet, no announcement.

**Routing ceremony.** Writing a multi-section rationale that cites heuristics by number, for
work the fast path already covered — and worse, doing extra investigation purely to justify the
route. Observed in testing: a run produced a policy-quoting justification for a three-file
rename while quoting the very line telling it not to, and spent 19 tool calls where doing the
job took 12. Investigate what the work needs, not what the explanation needs.

**Skill explosion.** Handing a worker a planning framework, a frontend skill, a browser-testing
skill, a CDN skill and a GitHub skill so it can add a database index. Every one is context it
must read past, and any of them may contradict the assignment. Default to none.

**Effort inflation.** Maximum effort everywhere because it is available. Every step above the
default needs a reason: a contract, an irreversible consequence, or evidence a cheaper setting
already fell short.

**The dangling gate.** Spawning an Opus review and finishing before it returns. Observed in
testing: a gate came back with two genuine blockers — a mass-assignment that silently dropped
`tenant_id`, and a backfill race that erased its own evidence — after the run had already
reported done. A review you do not wait for is not a review.

**Lossy fan-out.** Delegating recon across a surface you could have read, then reasoning over
the summaries. Observed in testing: a fanned-out audit reported three file-referenced findings
where reading directly produced thirteen, including a whole missed category. Worker summaries
are lossier than files; fan out only when the surface genuinely does not fit.

**Escalating the model when the context was the problem.** A worker that lacked the relevant
file does not need a bigger model; it needs the file. Diagnose before you spend.

**Routing by technology.** "It is Kubernetes, so Opus." "It is CSS, so Haiku." Both wrong: a
Helm values tweak is cheap and a design-system token migration across 200 components can hold
a real contract decision.

**Routing by file count.** Thirty files inside one convention is Sonnet work. One method that
redefines an invariant is not.

**Redesigning instead of implementing.** Being handed a feature request and returning an
architecture proposal. Existing architecture is evidence; only move the contract if this
change actually requires it.

**Skipping verification because the route was cheap.** The savings evaporate the first time
something broken ships. Run the tools — and if they cannot run, say so rather than implying
they passed.

**Asking the user which model to use.** The decision is inferable from the task; making it is
this skill's job.
