# Routing by domain

Read the one section that matches the work in front of you. These catalogs are illustrative,
not exhaustive, and they never override the core rule: route on decision complexity, contract
impact and reversibility. A domain name is a hint about *where the contracts usually live*,
not a tier assignment.

## Contents

- [Backend](#backend)
- [Frontend](#frontend)
- [Mobile](#mobile)
- [Infrastructure and DevOps](#infrastructure-and-devops)
- [Database](#database)
- [Security](#security)
- [Debugging](#debugging)

## Backend

*Laravel, Django, Rails, Spring, NestJS, Express, FastAPI, Go services — same routing.*

| Tier | Work |
|---|---|
| **Haiku** | Inspect routes, map models and entities, locate migrations, find middleware, inventory endpoints, inspect dependency usage, collect logs and failures |
| **Sonnet** | Controllers, services and actions, validation, serializers and resources, migrations, tests, jobs, ordinary business logic, API implementation, package integration |
| **Opus** | Domain boundaries, difficult transactions, consistency guarantees, concurrency, tenancy, authorization architecture, major schema evolution, complex state machines, critical production review |

## Frontend

*React, Next.js, Vue, Nuxt, Angular, Svelte — same routing.*

| Tier | Work |
|---|---|
| **Haiku** | Find components, map routes, inspect state usage, locate API calls, find duplicated components, inspect dependency versions, inventory accessibility issues |
| **Sonnet** | Components, hooks and composables, forms, state management, API integration, routing, tests, performance fixes, normal refactoring, TypeScript fixes |
| **Opus** | Application state architecture, the frontend/backend boundary, a large framework migration, rendering architecture, complex offline-first strategy, security-sensitive flows, major performance architecture |

A large frontend is not an Opus frontend. Volume is not complexity.

## Mobile

*React Native, Flutter, native iOS, Android.*

| Tier | Work |
|---|---|
| **Haiku** | Map screens, inspect permissions, find native integrations, inventory platform-specific files, locate API usage |
| **Sonnet** | Screens, navigation, API integration, local state, native bridges with established requirements, tests, normal platform bugs |
| **Opus** | Offline synchronization architecture, conflict resolution, sensitive credential storage, payment architecture, complex native or background execution, large cross-platform architecture decisions |

## Infrastructure and DevOps

*Docker, Kubernetes, CI/CD, cloud, servers, deployment.*

| Tier | Work |
|---|---|
| **Haiku** | Inspect configs, collect logs, identify which step failed, inventory services, diff environment variables, locate deployment definitions |
| **Sonnet** | Ordinary Dockerfiles, CI workflows, deployment configuration, ordinary server fixes, health checks, monitoring integration |
| **Opus** | Production migration strategy, zero-downtime architecture, disaster recovery, network and security boundaries, high-availability design, irreversible infrastructure changes, difficult distributed failures |

Route on **blast radius, not command length**. A one-line command can be the most
consequential act of the session. If a mistake takes out production or cannot be undone, the
plan deserves Opus even when the diff is trivial.

## Database

| Tier | Work |
|---|---|
| **Haiku** | Inspect schema, inventory tables and indexes, find query usages, identify slow-query candidates, compare migrations |
| **Sonnet** | Ordinary migrations, query optimization, indexes, ORM changes, normal data transformations |
| **Opus** | Destructive migrations, live-data transformation, partitioning strategy, consistency model, large-scale schema redesign, irreversible data migration, transaction and concurrency strategy |

## Security

Security is **not** automatically Opus. Most security work is evidence-gathering and ordinary
remediation.

| Tier | Work |
|---|---|
| **Haiku** | Gather dependency findings, exposed endpoints, configuration, existing authorization checks, secret-scanning results, known-pattern matches |
| **Sonnet** | Routine remediation, validation issues, missing authorization checks, ordinary secure-coding fixes |
| **Opus** | Threat modeling, privilege boundaries, tenant isolation, authentication architecture, cryptographic design, complex exploit chains, security-sensitive architecture, production-critical security review |

The split is reliable: finding and fixing a missing `authorize()` call is Sonnet work.
Deciding whether the authorization model itself can leak across tenants is Opus work.

## Debugging

```
Haiku    → collect evidence: logs, failing traces, recent changes, which tests fail, timings
Sonnet   → form hypotheses, reproduce, fix, add a regression test
Opus     → only if the problem stays genuinely hard or high-risk
```

Do not escalate merely because a bug survived one attempt. Escalation is earned when the
evidence points at concurrency, ordering, distributed behavior, deep architectural coupling,
corrupted state, complex security behavior, or production-only behavior with unclear
causality.

The most common debugging routing error is skipping the evidence step: an expensive model
reasoning about a bug it has no logs for, when a cheap worker could have handed it the
failing trace and the diff since the last good deploy.
