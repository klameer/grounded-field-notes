# 0002. Refusal is enforced in code, not requested in a prompt

Date: 2026-07-17
Status: Accepted

## Context

Asking a model to "only give numbers you can cite" works most of the time.
Most of the time is not good enough for a board pack. A prompt instruction is
a request. A check is a guarantee.

## Decision

After every answer, before it reaches the user, a post-answer numeral check
runs outside the model. Every numeral in the answer must trace to either a
citation span in retrieved content or a shown computation. A numeral that
traces to neither is a violation.

In refuse mode, a violating answer is not shown. It is rewritten into a
graceful refusal that says what could not be supported. The check runs on
both streaming paths so a streamed answer cannot leak an unsupported figure
before the check finishes.

Tolerances are deliberate: 4,150.6 and 4150.6 and $4.15M match the same
source cell. Years, question numbers and list indices are excluded.

## Consequences

- Refusal is testable. The eval suite includes questions whose answer is not
  in the corpus, and a digit-free refusal is the passing result.
- The guard occasionally refuses a correct answer when the model rounds in a
  way the tolerance table does not cover. Each case is logged and the
  tolerance table grows. A false refusal costs a re-ask. A false pass costs
  trust.
- The same guard covers every source type added later (analyst notes, web
  context) with no extra work, because it operates on text spans, not on
  document types.

## Alternatives considered

- Prompt-only. Rejected: unmeasurable, and the benchmark later showed raw
  models fabricating figures on refusal questions (see
  [evals](../../evals/METHODOLOGY.md)).
- A second model as judge on the live path. Kept for evals, rejected for
  production: a judge is another opinion, not a proof.
