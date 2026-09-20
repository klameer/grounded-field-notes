# Testing

Four tiers, one command, each priced in time and money so the right one
gets run. The design goal was that the cheap tier is unavoidable and the
expensive tier is deliberate.

| Tier | What runs | Cost | When |
|---|---|---|---|
| fast | 925 unit tests, dummy credentials | Free, about 20 s | Every commit; the pre-commit hook blocks on red |
| api | About 243 router and service tests against a live backend | Free, minutes | Any router or service change |
| smoke | Six stable golden questions, pass rate 1.0 | Pennies | Before "done"; after any ingest, model change or deploy |
| gates | Both full golden suites, three runs each | About an hour | Before merging anything that touches answering, retrieval or citations |

## The rules I hold myself to

- Every feature lands with its tests in the same session.
- Anything that can change an answer adds or updates golden questions,
  including at least one refusal case.
- A new external surface (the MCP server, the Slack bot) gets a live smoke
  script as well as unit tests.
- Known flappers are named in the runner so they are excluded from smoke
  and nobody spends an afternoon on them.
- The golden suites never run automatically ([ADR 0014](../adr/0014-evals-never-run-automatically-in-ci.md)).
  They cost money against production models and they are stochastic.
  Results are committed as dated files.

## What I learned the hard way

- A test suite with no test on the reranker client let it sit disabled for
  months ([incident 1](../incidents.md)).
- An eval that grades the raw model output instead of the rendered answer
  reported a pass on a refused answer ([incident 3](../incidents.md)).
- Re-provisioning fixture accounts wiped the golden corpora; the preflight
  now fails loudly on a missing expected source instead of reporting a
  false pass.

## Detail

| Kind | Count |
|---|---|
| Backend test files | 81 (54 unit, 23 api, 7 top level) |
| Security and isolation test functions | 95 |
| Front-end Playwright specs | 52 |
| Golden questions in repository YAML | 41, plus 32 on the board-pack corpus in the app |
| Stored eval result runs | 59, 2026-07-16 to 2026-09-10 |

Two long-standing unit failures are documented rather than hidden: a CSV
content-type test that fails only on Windows, and one stale render-fallback
test.
