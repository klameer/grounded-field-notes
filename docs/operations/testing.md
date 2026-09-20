# Testing

Four tiers, one command, each tier priced in time and money so the right
one gets run.

| Tier | What runs | Needs | Cost | When |
|---|---|---|---|---|
| fast | Unit tests, not api and not slow | Nothing | Free, about 20 s | Every change; enforced by the pre-commit hook |
| api | Router and service tests | Live backend, fixture accounts | Free, minutes | Any router or service change |
| smoke | Six stable golden questions (three per corpus), pass rate 1.0 | Backend and corpora | Pennies | Before "done"; after any ingest, model change or deploy |
| gates | Both full golden suites, three runs each | Same | About an hour wall-clock | Before merging anything that touches answering, retrieval or citations; before anything reaches a customer |

Baselines at 2026-09-10: fast tier 925 passed, 0 failed. API tier about
243 passing. Golden gates last full run at 96 to 98 percent on the rail
corpus and 92 percent on the biotech corpus.

## Inventory

| Kind | Count |
|---|---|
| Backend test files | 81 (54 unit, 23 api, 7 top level) |
| Security and isolation test functions | 95 |
| Front-end Playwright specs | 52 |
| Golden questions in repository YAML | 41 (17 rail, 24 biotech), plus 32 on the board-pack corpus in the app |
| Stored eval result runs | 59, from 2026-07-16 to 2026-09-10 |

## Rules

- Every feature lands with its tests in the same session: unit tests for
  logic, API tests for endpoints.
- Anything that can change an answer adds or updates golden questions,
  including at least one refusal case.
- A new external surface (the MCP server, the Slack bot) gets a live smoke
  script as well as unit tests.
- Smoke question sets exclude known flappers. Two questions flap at one or
  two in three runs and are listed by name so nobody wastes an afternoon on
  them.
- The pre-commit hook runs the fast tier whenever anything under the
  backend is staged and blocks on red. Docs-only and front-end-only commits
  skip it. Skipping the hook is for emergencies and says so.

## What is deliberately not automated

The golden suites (ADR 0014). They cost money against production models
and they are stochastic. They run by hand, on a subset when a subset will
do, and their results are committed as dated files.

## Known state

Two long-standing unit failures are documented rather than hidden: a CSV
content-type test that fails only on Windows because of a registry quirk,
and one stale render-fallback test. The smoke and gate tiers were blocked
for a period in August when the fixture account lost its corpora during an
account re-provisioning; the preflight fails loudly on a missing expected
source rather than reporting a false pass.
