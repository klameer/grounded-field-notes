# Architecture decision records

One file per decision, numbered in the order they were taken. Format follows
Michael Nygard's ADR template: context, decision, consequences, alternatives.
A record is never edited after it is accepted. If a decision is reversed, a
new record supersedes it and both stay.

| # | Decision | Date | Status |
|---|---|---|---|
| [0001](0001-files-are-the-source-of-truth.md) | Files are the source of truth, no fact table | 2026-07-17 | Accepted |
| [0002](0002-refusal-is-enforced-in-code.md) | Refusal is enforced in code, not requested in a prompt | 2026-07-17 | Accepted |
| [0003](0003-per-customer-instances.md) | One instance per customer | 2026-07-17 | Superseded by 0009 |
| [0004](0004-retire-pay-per-token-routing-for-dev.md) | Retire pay-per-token routing for development | 2026-07-22 | Accepted |
| [0005](0005-notes-as-a-parallel-table-with-trust-tiers.md) | Analyst notes as a parallel table with trust tiers | 2026-07-27 | Accepted |
| [0006](0006-evals-grade-what-the-user-sees.md) | Evals grade the user-visible answer | 2026-07-27 | Accepted |
| [0007](0007-roll-forward-migrations.md) | Roll forward, never roll back a live migration | 2026-07-28 | Accepted, scan job not built |
| [0008](0008-the-documents-table-is-the-queue.md) | The documents table is the ingestion queue | 2026-07-28 | Designed, not built |
| [0009](0009-pooled-tenancy-with-dedicated-as-premium.md) | Pooled tenancy, dedicated instance as a premium tier | 2026-07-28 | Accepted, supersedes 0003 |
| [0010](0010-turn-on-the-reranker.md) | Turn on cross-encoder reranking | 2026-08-02 | Accepted |
| [0011](0011-per-org-llm-keys.md) | Per-organisation LLM keys, no per-org base URL | 2026-08-02 | Accepted |
| [0012](0012-trust-chain.md) | The trust chain: every link checked by code | 2026-08-06 | Accepted |
| [0013](0013-benchmark-is-a-neutral-instrument.md) | The public benchmark is a neutral instrument | 2026-08-08 | Accepted |
| [0014](0014-evals-never-run-automatically-in-ci.md) | Evals never run automatically in CI | 2026-08-14 | Accepted, CI not yet built |
