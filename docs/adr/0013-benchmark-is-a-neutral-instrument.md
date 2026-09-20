# 0013. The public benchmark is a neutral instrument

Date: 2026-08-08
Status: Accepted

## Context

I built a benchmark to find out what my product got wrong, and it ended up
being the most useful thing I had built. Publishing it raised an obvious
conflict: a vendor publishing a benchmark on which the vendor's product
scores well.

## Decision

The benchmark repository
([test-your-finance-llm](https://github.com/klameer/test-your-finance-llm))
is a clinical instrument. Rules:

- The repository contains the pack, the questions, the answer key, a
  deterministic grader, checksums, and adjudicated results with full
  verification bundles. No marketing copy. Every claim about it lives
  outside the repository and points at it.
- Scoring is split into a machine-verifiable column (81 of 100 points,
  banked by the grader, same transcript same score for anyone) and a
  judgment column (19 points, awarded under rules pre-registered before any
  row was judged). The auto column is the headline metric.
- Rulings are recorded and cut both ways. Two went against the product, one
  for a competitor. They are in the results file.
- Conditions that disadvantage the product are disclosed, not compensated.
  The product could not ingest CSV at the time, so it saw 32 of 34
  documents and lost a point on one question in every run. Disclosed, zero
  adjustment.
- A private holdout of 25 parallel questions, hash-committed and never
  published, exists for verified runs and for contamination detection.
- Framing rule for anything written about it: "I benchmarked raw models to
  learn how to improve the product, and the improvements moved the auto
  score 13 to 15 points in a day." Never a league-table blowout, because
  after a fair human pass the public set saturates at 95 to 100 for every
  frontier tool, and the best raw model scored 100 to the product's 99.

## Consequences

- The benchmark drove the two product changes that mattered most in August
  (figure-free refusals and authority-aware ranking). See
  [evals/results](../../evals/results/).
- The line-ending attribute on the repository is load-bearing: end-of-line
  conversion changes checksums. Every published file has a SHA-256.
- A canary string sits in the pack to detect training-set contamination.
