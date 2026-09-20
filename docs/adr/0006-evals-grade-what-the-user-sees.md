# 0006. Evals grade the user-visible answer

Date: 2026-07-27
Status: Accepted

## Context

My eval runner originally graded the model's raw answer, before the numeral
guard ran. During a demo-prep run the model produced a revenue figure of
$1,975.1M that was wrong. The guard caught it and the user saw a refusal.
I was watching the run and saw a refusal on screen next to a green
row. The eval had graded the raw text
and the raw text matched the expected value within tolerance by accident of
formatting.

## Decision

The eval runner grades exactly what the user would see: the post-guard
answer text, refusals included. If the guard rewrote the answer, the eval
sees the rewrite.

## Consequences

- Pass rates dropped on the first run after the change, then recovered as
  I fixed the tolerance table and prompts. The earlier numbers had been
  flattering me.
- A guard regression now shows up as an eval regression, which is the point.
- The same principle applied later to the public benchmark: raw models were
  graded on their transcript, the product was graded on its rendered answer.
