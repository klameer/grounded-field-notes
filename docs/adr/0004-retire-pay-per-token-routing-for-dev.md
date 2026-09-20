# 0004. Retire pay-per-token routing for development

Date: 2026-07-22
Status: Accepted

## Context

Chat, sub-agents, metadata extraction and the eval judge all ran through a
pay-per-token router on a frontier model with an agentic loop. One eval
question against the hosted demo cost between £1.50 and £4. A full suite
drained $50 in a day. Evals stopped being something I ran casually, which
means they stopped being run.

## Decision

Development and demos run locally, and each call class gets the cheapest
provider that meets its need:

| Call class | Provider | Why |
|---|---|---|
| Chat and sub-agents | Flat-rate coding-assistant subscription via a Responses-API bridge | Unmetered, so evals are free to run |
| Utility calls (metadata, judge, citation check) | A mini-class model on a pay-per-token API | Cheap and good enough. A judge verdict costs about $0.0002 |
| Embeddings | Pay-per-token embedding API | One vector space per database. Never changes casually |

The router stays configured as a fallback. Production later moved to a
single pay-per-token provider with prompt caching (see
[cost controls](../operations/cost-controls.md)).

## Consequences

- Evals became routine again: three-run stability gates before every demo,
  full suites after every merge.
- The bridge needed two fixes: the endpoint rejects a max-output-tokens
  parameter, and structured outputs must be strict-mode JSON schemas
  (additionalProperties false, every field required).
- A standing rule followed: never delete and re-ingest a corpus without an
  explicit yes and a cost estimate, because ingestion runs embeddings plus a
  per-file LLM metadata pass, and a corpus sync deletes and replaces
  wholesale.
- Cost is now measured per question, not discovered per invoice.

## Alternatives considered

- Top up and carry on. I called it a bottomless pit and stopped.
- Switch every call to the mini model. Rejected: chat quality on multi-step
  finance questions visibly dropped.
