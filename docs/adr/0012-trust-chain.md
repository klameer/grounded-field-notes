# 0012. The trust chain: every link checked by code

Date: 2026-08-06
Status: Accepted

## Context

A citation says where a number claims to come from. It does not prove the
number is right. An answer can cite the correct cell and misread it, or cite
the correct cells and add them up wrongly. Finance readers know this, which
is why "AI with citations" gets a polite nod and no budget.

The distinction the product needed: a citation is an after-the-fact audit; a
verification proves the number. Who wrote the citation, and who did the
arithmetic?

## Decision

Every figure in an answer sits on a chain, and every link is checked by
code, not by the model:

```
document  →  extraction  →  computation  →  answer
   │             │              │             │
 hash of      cell/page      deterministic  numeral guard
 the file     provenance     calculator     (ADR 0002)
```

I built it in one day on a branch and merged it five days later:

1. **Deterministic calculator.** Arithmetic in answers runs in code. The
   model proposes the expression and the input bindings; the calculator
   evaluates it.
2. **Verified input bindings.** Each input to a computation is itself a
   lookup with a resolved cell or page provenance. An input that does not
   resolve fails the computation.
3. **Provenance-fed numeral guard.** The post-answer check (0002) now takes
   its allowed set from the provenance registry, not from a text match over
   retrieved chunks. Tighter, and it covers computed values.
4. **Derived-value chips.** A computed figure renders with a function chip
   showing the expression and its inputs, each input clickable to its cell.
5. **Trust-chain strip.** Under every answer, a strip shows the chain state
   for that answer: sources cited, inputs verified, computations checked,
   guard result.

## Consequences

- The public benchmark's citation-verifiability column is the external
  measure of this: 60 of 60 non-refusal citations machine-confirmable on
  the product's best cells, against 53 of 60 for the best raw model.
- Known gap: a figure computed and verified in turn N is not trusted in
  turn N+1. A follow-up question that converts a verified figure to another
  unit was refused because the input had no provenance in the new turn.
  Candidate fix is to persist verified computed values into the thread's
  provenance registry.
- "Trust chain" is the product name for this and the marketing name, used
  verbatim. The word is verification, never validation; validation is
  reserved for golden-question runs.
