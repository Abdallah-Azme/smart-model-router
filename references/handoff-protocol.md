# Handoff protocol and context compression

The model you route to only ever sees what you hand it. A handoff is therefore both the
cost control and the quality control: an expensive model given a repository dump reasons
worse *and* costs more than the same model given a tight packet.

## The packet

Use these headings. Drop any that are genuinely empty rather than padding them.

```
OBJECTIVE
  One or two sentences. What decision or change is needed, and why now.

CURRENT STATE
  How the relevant part of the system behaves today. Observed, not assumed.

RELEVANT FILES
  Paths, with a clause each on why they matter. Include the small excerpts that
  carry the actual logic — not whole files.

IMPORTANT FINDINGS
  What recon established. Facts, with the file or command they came from.

ARCHITECTURAL CONSTRAINTS
  Conventions, boundaries, framework and package constraints, existing contracts
  that must not break, deployment or data constraints.

RISKS
  What breaks if this is wrong. Be concrete: overselling, double-charging,
  cross-tenant reads, lost writes.

ALREADY TRIED
  Approaches attempted and why they were rejected or failed. This prevents the
  next model re-deriving a dead end.

WHAT THE NEXT MODEL MUST DECIDE / DO
  The specific question or deliverable. One thing, ideally.
```

## Rules that matter

**Never paste an entire conversation.** Extract the conclusions. A transcript is a record
of how you got somewhere; the next model needs where you are.

**Excerpt, do not attach.** Twenty relevant lines of a `reserve()` method beat the whole
`InventoryService`. If a file matters only as a name, name it.

**Facts carry their source.** "Stock is decremented in
`app/Modules/Inventory/Actions/ReserveStock.php:42`, inside no transaction" is usable.
"Inventory handling looks inconsistent" is not.

**State the open question explicitly.** A packet that ends without a question invites the
expensive model to re-scope the problem, which is the most expensive thing it can do.

**Compress on the way back too.** When a cheap worker reports to you, or you report to the
user, return conclusions and the evidence for them — not everything the worker read. This
is a large part of what makes parallel Haiku recon cheap: four workers, four short
findings, one consolidated packet.

## Sizing

A good packet for an architecture decision is on the order of a page or two. If it is
running much longer, either the objective is really several objectives (split it), or
unfiltered context is leaking in (cut it).

## Good example

```
OBJECTIVE
  Decide the concurrency strategy for stock reservation at checkout so two
  simultaneous checkouts cannot oversell the last unit.

CURRENT STATE
  ReserveStock reads stock_on_hand, compares, then decrements. Read and write are
  separate statements with no transaction and no lock.

RELEVANT FILES
  app/Modules/Inventory/Actions/ReserveStock.php:28-61 — the read-compare-write.
  app/Modules/Order/Actions/PlaceOrder.php:74 — calls ReserveStock, then creates
    the order in a separate DB::transaction.
  database/migrations/2025_03_11_create_stock_levels_table.php — no unique
    constraint, no version column.

IMPORTANT FINDINGS
  No pessimistic locking anywhere in the module (grep for lockForUpdate: 0 hits).
  Order creation and stock decrement are in different transactions, so a failed
  order leaves stock decremented.
  Existing tests never exercise concurrency.

ARCHITECTURAL CONSTRAINTS
  MySQL 8 / InnoDB. Modular monolith: Order must not write inventory tables
  directly; it goes through the Inventory module contract.

RISKS
  Overselling physical stock; stock silently lost when order creation fails.

ALREADY TRIED
  Wrapping only PlaceOrder in a transaction — does not help, the race is inside
  the read-compare-write.

WHAT THE NEXT MODEL MUST DECIDE
  Locking strategy (row lock vs conditional update vs reservation table), where
  the transaction boundary belongs given the module boundary, and the invariant
  the tests should assert.
```

## Bad example, and why

```
Here is our inventory code, please help with concurrency.
[8 files pasted in full, including the Product model and two unrelated migrations]
```

No objective, no question, no constraints, no findings. The expensive model now has to do
the recon that Haiku could have done for a fraction of the cost, and it has to guess what
"help" means. Both the bill and the answer get worse.
