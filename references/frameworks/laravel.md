# Laravel — framework notes

These notes refine the examples for Laravel codebases, especially modular monoliths. They do
**not** override the routing principle: route on decision complexity, contract impact and
reversibility. "It is Laravel" carries no tier information, and neither does the size of the
`app/` directory.

## Where the contracts usually live

In a Laravel modular monolith the risk concentrates in a predictable set of places. These are
the candidates for Opus:

- Module boundaries, and what is allowed into the Shared Kernel
- Aggregate boundaries and the domain invariants each aggregate owns
- Cross-module contracts: the interfaces in `Contracts/`, and events versus direct calls
- API versioning strategy under `Http/Controllers/Api/V*`
- Inventory lifecycle: reservation, allocation, fulfilment, returns
- Order lifecycle and its state machine transitions
- Pricing, tax and accounting rules
- Authorization architecture (policies, gates, permission model) and tenancy boundaries
- Schema evolution against live data

## Where the routine work lives

Sonnet implements the resulting design, and this is the bulk of the work:

Controllers, form requests, API resources, policies, DTOs, Actions, Services where justified,
migrations that follow the decided strategy, jobs, events, listeners, factories, feature and
unit tests, Inertia pages and React/TypeScript components, TanStack integration, Spatie
package wiring (permission, query-builder, media-library), Larastan and Pint fixes.

Haiku inspects the current implementation, gathers conventions from sibling modules, inventories
routes and migrations, and collects verification output.

## The useful heuristic for these codebases

If the change alters a **contract** — a module boundary, an invariant, an event payload other
modules consume, the permission model, the shape of a published API response — consider Opus.
If it works *within* an existing contract, it is Sonnet work no matter how many files it
touches.

Concretely: adding a filter to an existing index endpoint is Sonnet even across five files.
Changing what `reserved` means on a stock row is Opus even in one method.

## Laravel-specific traps worth a cheap recon step

These are cheap for Haiku to check and expensive to miss, so they are worth asking about
explicitly before implementing:

- **Global scopes that fail open.** A `BelongsToTenant` scope conditional on `Auth::user()`
  silently becomes a no-op in queue jobs, console commands and scheduled tasks. Any tenancy
  review must cover the unauthenticated paths, not just HTTP.
- **`withoutGlobalScopes()`** anywhere near a tenant-scoped model.
- **Enums duplicated as bare strings** in `config/`, migration column defaults, seeders, and
  hand-maintained TypeScript unions. A rename sweep that only greps the PHP enum will miss them.
- **Work outside the transaction that guards it.** Reserving stock or charging a card before
  the `DB::transaction` that creates the record leaves orphans when the transaction rolls back.
- **`firstOrFail` → compute → `save()`** on any counter. That is a read-compare-write race
  with no lock; `lockForUpdate` or a conditional `UPDATE` is the fix, and which one is a
  contract decision.

## Verification commands

Run whichever the project defines, and do not report a pass you did not observe:

```
composer test        # pest / phpunit
composer analyse     # phpstan / larastan
npm run types        # tsc --noEmit for the Inertia frontend
vendor/bin/pint      # formatting
```

Architecture tests (Pest's `arch()` presets, or Deptrac) are the deterministic way to enforce
module boundaries — cheaper and more reliable than any model reviewing imports by eye.
