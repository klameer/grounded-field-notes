# Board Pack Test v1.0.1, baseline runs

Run 2026-08-08, before the benchmark was published, so contamination-free.
25 questions, 3 independent single-pass runs per cell, medians. Adjudicated
under pre-registered rules the same day. Machine-readable rows in
[2026-08-08-board-pack-test-v1.0.1.json](2026-08-08-board-pack-test-v1.0.1.json).
Full verification bundles are in the benchmark repository's `results/`.

## League table

| Agent | Auto r1 / r2 / r3 | Auto median (of 81) | Final median (of 100) |
|---|---|---|---|
| Grounded v2 × gpt-5.5 | 81 / 79 / 81 | **81** | 99 |
| Grounded v2 × claude-opus-5 | 81 / 81 / 81 | **81** | 99 |
| GPT-5.5, raw with code interpreter | 74 / 69 / 71 | 71 | 98 |
| Grounded v1 × gpt-5.5 | 66 / 68 / 68 | 68 | 95 |
| Grounded v1 × claude-opus-5 | 69 / 66 / 66 | 66 | 99 |
| Claude Opus 5, raw with code sandbox | 68 / 66 / 66 | 66 | **100** |

Read the auto column. The final column saturates: after a fair human pass
every frontier cell lands between 95 and 100, and the best raw model beat
the product by one point. The product's advantage is in what survives
verification with no human in the loop.

## Per-tier auto medians

| Agent | T1 lookup (24) | T2 cross-doc (20) | T3 compute (20) | T4 version (16) | T5 refusal (20) |
|---|---|---|---|---|---|
| Claude Opus 5, raw | 24 | 15 | 15 | 12 | **0** |
| GPT-5.5, raw | 21 | 14 | 13 | 10 | 15 |
| Grounded v1 × gpt-5.5 | 23 | 15 | 15 | 12 | 3 |
| Grounded v1 × claude | 24 | 15 | 15 | 12 | 0 |
| Grounded v2 × gpt-5.5 | 24 | 15 | 15 | 12 | **15** |
| Grounded v2 × claude | 24 | 15 | 15 | 12 | **15** |

Tier 5 is where the table is decided. Raw Claude answered every
unanswerable question with a figure. The v1 product did nearly as badly: its
refusals contained digits (a "closest available figure"), and any figure in
a refusal scores zero.

## Citation verifiability

Share of 60 non-refusal citation rows per cell that a machine can confirm
against the answer key's sheet, cell or page traces.

| Agent | Machine-confirmable |
|---|---|
| Grounded v2 × claude-opus-5 | **60 / 60** |
| Grounded v1 × claude-opus-5 | **60 / 60** |
| Grounded v2 × gpt-5.5 | 59 / 60 (one wrong-sheet citation) |
| Claude Opus 5, raw, including prose rescue | 59 / 60 |
| Grounded v1 × gpt-5.5 | 57 / 60 |
| GPT-5.5, raw, including prose rescue | 53 / 60 |

## Conditions

- Raw rows received all 34 documents first-party: PDFs native in context,
  spreadsheets and decks in the vendor's own code sandbox, one small CSV
  inlined as text. A 16-file sandbox limit applied identically to both.
- Product rows ran the product's own ingestion, which could not ingest CSV
  at the time: 32 of 34 documents. Adjudicated as a product gap with no
  compensating adjustment. Cost the product one working point on question
  P10 in every run.
- Same exam preamble for every cell, plus one identical formatting
  instruction requesting a `SOURCES:` block so the deterministic source
  gate can read each model's own citations. Provider defaults otherwise.

## v1 to v2: what moved the score

Both product cells gained 13 to 15 auto points in one day, and run
variance on the Claude cell went to zero. Two changes:

1. **Figure-free refusals.** A prompt rule plus a hold-back template so a
   refusal never carries a digit. Worth 12 to 15 points per run on Tier 5.
2. **Authority-aware ranking.** Documents the corpus marks as superseded
   (a v1 where a v2 FINAL exists, a legacy tracker) are demoted before
   rerank. Fixed the FTE question (P04) in four of six runs.

A third change, exact claim cells (citations name `Opex!H12` rather than a
row band), was auto-neutral but made human confirmation instant.

Knock-ons: one export citation had to be narrowed because a £51k figure sat
under the claim-figure size floor (fixed afterwards), and the hold-back
template swallowed one refusal's readable reason (cosmetic, queued).

## Rulings that cut both ways

- P13's answer-key trace pointed at gross profit; the question asked for
  gross margin percent, one cell down. Raw GPT had cited the correct cell
  in all three runs, so its three denials were reversed.
- A product v2 run on the same question was denied under the corrected
  trace: it had cited a different sheet entirely.
- P04: a document the pack marks as superseded is not a valid source even
  when its figure happens to be right. Ruled against the product's v1
  cells.

## Traps

All three embedded traps (a £285k accrual gap between an export and the
issued pack, a reissued v2 FINAL, a stale plan-basis footnote) were beaten
on content by every cell in every run. They separate citation and
reasoning quality, not verdicts.
