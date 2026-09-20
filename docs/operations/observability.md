# Observability

One rule above the others: the trace store being down must never fail a
request. Prompt fetches fall back to in-code constants, score writes are
fire-and-forget, event ingestion is asynchronous and batched, and there is a
runbook step that stops the trace store and confirms the app still answers.

## The facade

A single tracing module fronts three providers behind one environment
variable: LangSmith, Langfuse, or off. Off means the `traceable` decorator
becomes an identity and `trace()` returns nothing, with no conditional
imports at call sites. Grounded runs Langfuse.

Observations are typed (agent, tool, retriever, guardrail), which is what
makes the agent graph render as a graph instead of a flat list of spans.
Session id is the thread id, so one conversation is one session. Every
event carries the release (git short SHA by default) and an environment
tag (development, staging, production).

## What is captured

| Signal | Where it comes from |
|---|---|
| Tokens and cost per generation | Trace-store generations, priced from a model registry. Models on a flat-rate subscription are registered at $0 on purpose so tokens are still counted |
| Reranker spend | Billed search units counted in the rerank client, the only honest cost signal for that stage |
| Latency | Per span; the ops dashboard shows p95 with a note that the average hides what users complain about |
| Retrieval | Retriever spans carry the query, the candidate set and the reranked order |
| Guard outcomes | Guardrail spans for the numeral check, with refusal reason |
| Quality | A sampled live LLM-as-judge for hallucination (0 grounded, 1 hallucinated); user thumbs up/down through an authenticated backend endpoint |
| Prompts | Managed in the trace store with the in-code prompt as fallback; a seed script pushes changes idempotently by content compare |

Masking runs before any event leaves the process: emails, phones, card
shapes and key shapes are regex-redacted.

## Alerts

A scheduled script checks two things daily and exits non-zero on breach:

| Check | Threshold |
|---|---|
| Traced cost today | $10 |
| Hallucination judge, 24-hour average | 0.3 |
| Hallucination judge, any single score | 0.7 |

A live reading from the commit that wired it: cost $1.19 of $10, average
0.13, two single traces flagged for review.

Thumbs-down traces are routed to an annotation queue for a human.

## Local versus hosted

The local Docker stack (Postgres, ClickHouse, Redis, MinIO, one web and one
worker container) is right for "did my change improve this trace" and wrong
for anything you would page on: nothing is replicated, nothing is backed
up, nothing has a retention policy. The hosted service or a properly sized
self-host needs managed Postgres with point-in-time recovery, ClickHouse
sized as the real workload, persistent Redis as the ingestion queue,
S3-compatible blob storage with lifecycle rules, and at least two web and
two worker replicas. The application difference between the two is one
host variable, verified by running the same build against both.

## Two corrections to vendor documentation, verified

- The coding-assistant MCP transport key is `"type": "http"`, not the
  name the trace store's docs give.
- The "HTTPS required" note does not apply to localhost.

And a scope limit: the trace store's MCP server, at the version in use,
exposes six tools, all prompt management. Traces and metrics are not
queryable over MCP; use the REST API.
