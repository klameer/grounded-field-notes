# Ablation: cross-encoder reranker on versus off

Date: 2026-08-02
Instrument: retrieval-only eval (query embedding, hybrid retrieval, score
against expected sources; no chat, no judge).
Decision record: [ADR 0010](../../docs/adr/0010-turn-on-the-reranker.md).

## Setup

Hybrid retrieval (Postgres full-text + pgvector, reciprocal rank fusion)
returns a candidate set; the reranker reorders it. Same questions, same
corpora, same embeddings, reranker off then on.

## Results

| Corpus | n | hit@1 off → on | hit@3 off → on | hit@k off → on | MRR off → on |
|---|---|---|---|---|---|
| Rail | 12 | 0.42 → **0.75** | 0.50 → **0.92** | 0.75 → **0.92** | 0.48 → **0.81** |
| Biotech | 18 | 0.44 → **0.72** | 0.56 → **0.83** | 0.89 → **0.94** | 0.53 → **0.78** |

Per question: 14 improved, 3 regressed (rank 1 → 2, 1 → 3, 5 → 7; all still
top ten; none diagnosed), 13 unchanged. Three documents that were outside
the top ten without the reranker landed at rank one to three with it.

## Cost of the stage

| Measure | Value |
|---|---|
| Added retrieval latency | +0.39 s per search |
| Median reranker call | 0.17 s over a 39-document candidate set |
| Billing unit | 1 search unit per call regardless of candidate count (observed on the meter, contradicts the vendor's pricing page) |
| Searches per answered question | median 4, mean 3.9, p90 7, max 12 (86 real eval threads) |
| Cost per question | about $0.008 median, $0.014 at p90 |

Sub-agent searches inside the deep-research harness are not counted in the
per-question figure; those answers cost a multiple.

## What it exposed

The reranker had never run before this test. Its base URL setting was
empty, and empty meant disabled. Nothing in the test suite would have
caught that, because the client's error path returned the unranked list
with a log line. The retrieval eval is now the first check after any
change to the retrieval stack, and the client has retries with backoff and
sixteen unit tests.
