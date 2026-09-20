# 0014. Evals never run automatically in CI

Date: 2026-08-14
Status: Accepted. No CI workflow exists yet; the tiers run through a local runner and a pre-commit hook (see operations/testing.md).

## Context

Designing the CI workflow. The obvious move is to run the golden-question
suite on every pull request and fail the build on regression. Two things
argued against it: a full suite against a production-grade model costs real
money per run (see 0004), and LLM answers are stochastic, so a single-run
gate flaps.

## Decision

- Unit tests run on every commit through a pre-commit hook (925 tests,
  about twenty seconds, dummy credentials). API tests run against a live
  backend on any router or service change.
- The retrieval-only eval (no LLM calls beyond embeddings) can run on
  demand cheaply and is the first check after any retrieval change.
- The full golden-question suite is a manual workflow, dispatched by a
  person, on a named subset, with the run count chosen for the question.
  Three-run stability gates run before demos and after merges, by hand.
- Results are committed as dated files so the history is in git, not in a
  CI log that expires.

## Consequences

- No flaky red builds from model variance.
- The discipline has to be mine. The compensating rule is that no demo
  happens without a clean three-run gate, and the eval results directory
  shows whether that rule is being kept.
- Cost per full local suite is roughly a dollar and a half in reranker
  calls, because chat runs on a flat-rate subscription locally. Against
  production pricing the same suite is about $23 at list. That gap is why
  the gate is manual.
