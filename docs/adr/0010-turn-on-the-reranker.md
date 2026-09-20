# 0010. Turn on cross-encoder reranking

Date: 2026-08-02
Status: Accepted

## Context

Retrieval is hybrid: Postgres full-text search and pgvector similarity, fused
by reciprocal rank fusion, then a cross-encoder rerank. The rerank stage had
been in the code from the start. Its base URL setting was empty, and an empty
base URL is the disable switch. The reranker had never run.

Turning it on was cheap to test because the retrieval-only eval scores
hit@1, hit@3, hit@k and MRR against expected sources with no chat and no
judge.

## Decision

Reranking on, with a production key. Results on two corpora, same questions,
reranker off then on:

| Corpus | hit@1 | hit@3 | hit@k | MRR |
|---|---|---|---|---|
| Rail (12 questions) | 0.42 → 0.75 | 0.50 → 0.92 | 0.75 → 0.92 | 0.48 → 0.81 |
| Biotech (18 questions) | 0.44 → 0.72 | 0.56 → 0.83 | 0.89 → 0.94 | 0.53 → 0.78 |

Fourteen questions improved. Three regressed by one to two ranks, all still
in the top ten, none diagnosed. Three documents that had been outside the
top ten entirely landed at rank one to three.

Full table in [evals/ablations/rerank-on-off.md](../../evals/ablations/rerank-on-off.md).

## What turning it on found

- The first key was a trial key, rate-limited to about ten calls a minute.
  It returned 429s under a normal eval run.
- The client treated any error as "return the unranked list" with a log
  line. That is a silent quality drop in the middle of a demo. The client
  now retries 429 and 5xx with backoff honouring Retry-After, and it has
  sixteen unit tests. It had none.
- Latency cost: about 0.39 seconds per search. Median call 0.17 seconds
  over a 39-document candidate set.
- Billing is one search unit per search regardless of candidate count,
  measured by watching the meter, which contradicts the vendor's pricing
  page. At a median of four searches per answered question that is under a
  cent per question.

## Consequences

- The retrieval eval is now the first thing run after any change to
  chunking, embedding or ranking. It costs nothing but embedding calls.
- Authority-aware ranking came later on top of this: documents the corpus
  marks as superseded (a v1 where a v2 FINAL exists, a legacy tracker) are
  demoted before rerank. That single change fixed a benchmark question in
  four of six runs.
