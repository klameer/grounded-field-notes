# 0005. Analyst notes as a parallel table with trust tiers

Date: 2026-07-27
Status: Accepted

## Context

I wanted analysts to be able to add their own commentary to the corpus: "the Q3 dip is the
warehouse move", "ignore the legacy tracker after May". Useful context, but
a note is an opinion and a spreadsheet is a record. If both enter the same
retrieval pool with equal weight, an unvalidated note gets cited with the
same authority as an audited workbook. The design problem was trust, not
relevance. The failure mode has a name here: trust laundering.

## Decision

Notes live in their own table, embedded with one call (title plus content,
truncated for the embedding only), with no metadata extraction and no
document pipeline. They are merged into hybrid retrieval before the
reranker, so they compete on relevance like any chunk.

Every source carries a trust tier that travels with every retrieved chunk
and every citation:

| Tier | Examples | May source a number? |
|---|---|---|
| source-of-record | Workbooks, issued packs, statements | Yes |
| commentary | Board narrative, management commentary | No |
| analyst-note | Notes written in the app | No |

Tiers below source-of-record get the treatment web search already had:
context, visibly labelled, never the source of a figure. The numeral guard
(ADR 0002) enforces this with zero extra code because note text is split
into spans the same way document text is.

The UI shows a tinted, labelled, clickable sources strip so the reader can
see at a glance which kind of source each claim rests on.

## Consequences

- No vector index on the notes table. At tens of rows per user an exact
  scan is correct and simpler. This is the project convention for small
  tables.
- Editing a note is a single row update. Routing notes through the document
  pipeline would have re-ingested on every edit.
- I assessed images and OCR and parked them at low feasibility for the same
  reason the fact table was rejected: an OCR'd figure reproduces the
  semantic mis-mapping risk.

## Alternatives considered

- Notes through the document pipeline. Rejected: re-ingest on every edit,
  and the metadata pass costs money.
- Relax the not-null constraint on chunk-to-document so notes become
  document-less chunks. Rejected: highest blast radius of the three, touches
  every retrieval query.
