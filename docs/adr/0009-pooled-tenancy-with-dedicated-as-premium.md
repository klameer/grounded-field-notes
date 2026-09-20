# 0009. Pooled tenancy, dedicated instance as a premium tier

Date: 2026-07-28
Status: Accepted. Supersedes [0003](0003-per-customer-instances.md).

## Context

Eleven days after deciding on one instance per customer (0003), the cost of
that decision was visible: every migration is a manual fan-out, there is no
instance registry, and the demo instance had already drifted from the code.
Rough sums: twenty small tenants on their own instances cost upwards of $600
a month before any of them pays; pooled on one larger database they cost
well under $150.

## Decision

Pooled multi-tenancy is the default. A dedicated single-tenant instance
(customer's own database, optionally their own model keys or a zero-retention
cloud endpoint) is a premium tier, not the base.

Inside the pool, isolation is enforced in three layers:

1. Every table carries an organisation id and a visibility column
   (private or organisation). A null-owner sentinel that used to mean
   "shared" was retired, which also fixed a data-loss bug on descendant
   ownership and an asymmetry where shared folders were invisible to
   retrieval.
2. All five retrieval functions are organisation-scoped in the database,
   with citation parity: the grounding step admits exactly the set of
   chunks retrieval surfaced, so a citation can never point outside the
   caller's scope.
3. Row-level security is the backstop, and a centralised visibility module
   is the only place scope rules are written.

## Consequences

- Blast radius is now shared. A corrupting migration corrupts every tenant,
  so rollback speed replaces ring rollout as the primary mitigation, and the
  rollback runbook became the most load-bearing document in the pipeline
  plan. Point-in-time recovery must be verified before the first real
  customer.
- Pooled vector search needs the tenant filter leading its indexes or every
  query scans every tenant's chunks.
- Ingestion must move to its own worker before real pooling: in-process
  background tasks are a noisy neighbour, deploys kill in-flight jobs, and
  per-process semaphores multiply across replicas.
- Verified by a twelve-test isolation suite (same-org retrieve and cite,
  cross-org wall, unshare revokes) plus security drift pins on function
  grants. Writing the pins found two pre-existing gaps: one retrieval
  function was executable by the anonymous role, and one table had no pin.
- The security story keeps its "dedicated deployment available" claim
  because it is true. It drops "no shared infrastructure" as a default
  claim because that is no longer true.
