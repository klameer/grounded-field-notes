# 0001. Files are the source of truth, no fact table

Date: 2026-07-17
Status: Accepted

## Context

My original roadmap built toward a dimension-keyed facts table: ingest every
report, extract every figure into rows keyed by entity, period, measure and
version, then answer questions from the table. It is the obvious design for a
finance question-answering system and it is what most "AI for finance" tools
do under the hood.

The input is unbounded. Every finance team keeps its reports differently, and
the same team changes layouts between months. A fact table forces that
unbounded variety into one bounded schema, with no way to know that ingestion
understood a given layout correctly.

## Decision

Files stay the permanent source of truth. There is no fact table.

A number can only be stated in two ways:

1. A lookup, transcribed from a cited cell or page, with a citation that
   resolves to that exact location on one click.
2. A computation, run by deterministic code in a sandbox, with the inputs
   (each one a lookup) and the code shown.

The model picks which cells matter. It never authors a value.

Any derived structure built for speed is a cache and never a source. It must
be re-verified against the file at answer time. If the file disagrees, the
answer refuses.

## Why I rejected the fact table

Two failure modes disqualify it for a product whose only promise is trust.

**Semantic mis-mapping launders errors.** A fact row can hold the right value
and the right cell reference and the wrong meaning: a forecast column read as
an actual, a restated figure read as original. The citation click-through
passes. The answer is wrong. The store has converted an interpretation error
into audit-grade confidence, which is worse than no answer.

**A review queue relocates the problem.** The usual fix is a human review
queue for extracted facts. That turns unbounded parsing complexity into
unbounded review labour. It is the cost structure of every planning-system
implementation I have run, and it never ends because the inputs never stop changing.

## Consequences

- Aggregation questions ("total opex across all departments for FY25") are
  computed at question time from cited cells, not read from a table. Slower,
  visibly more work, correct by construction.
- Spreadsheet ingestion has to keep the grid intact (sheet, row, column, A1
  reference per cell) because the citation must resolve to a cell. See the
  [ingestion notes](../architecture/ingestion.md).
- Web search results are context only, labelled external, and can never be
  the source of a company number.
- Live-system integrations inherit the rule: a figure from a planning system
  must cite system, query and timestamp.
- A bounded per-customer store ("we learn your pack") stays parked. It is
  revisited only if three things are all true: a real customer, stable
  recurring formats, and observed demand for aggregation that question-time
  computation cannot serve.

## Alternatives considered

- Universal fact extraction with a review queue. Rejected above.
- Text-to-SQL over an extracted store. Same failure modes, plus a second
  place for meaning to drift.
- Fine-tuned extraction models per layout. Does not remove the mis-mapping
  risk and adds a retraining loop per customer.
