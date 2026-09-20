# Observability

One rule above the others: the trace store being down must never fail a
request. Prompt fetches fall back to in-code constants, score writes are
fire-and-forget, event ingestion is asynchronous and batched, and the
runbook has a step that stops the trace store and confirms the app still
answers.

## What I wanted to be able to see

| Question | Signal |
|---|---|
| What did this answer cost? | Tokens and cost per generation, priced from a model registry. Models on a flat-rate subscription are registered at $0 on purpose so tokens still count |
| What did retrieval return before and after rerank? | Retriever spans carry the query, the candidate set and the reranked order |
| Why did the guard refuse? | Guardrail spans for the numeral check, with the reason |
| Is quality drifting? | A sampled live LLM-as-judge for hallucination; thumbs down routed to a human annotation queue |
| What is slow? | Per-span latency; the dashboard shows p95 because the average hides what users complain about |
| Did a document leak into a trace? | Masking runs before any event leaves the process: emails, phones, card shapes, key shapes |

Observations are typed (agent, tool, retriever, guardrail), which is what
makes an agent run render as a graph instead of a flat list of spans.
Session id is the thread id. Every event carries the release and an
environment tag.

## Alerts

A scheduled script checks daily: traced cost above $10, hallucination
judge 24-hour average above 0.3, any single score above 0.7. First live
reading: $1.19 of $10, average 0.13, two singles flagged.

## What I learned

- The provider facade (one variable picks LangSmith, Langfuse or off, and
  off is an identity decorator) is the reason I could port the trace store
  in a day and the reason a trace outage cannot take chat down.
- The trace-store secret must never reach the browser. User feedback goes
  through an authenticated backend endpoint, and an end-to-end test
  asserts no secret appears in the front-end bundle.
- The local Docker stack is right for "did my change improve this trace"
  and wrong for anything I would page on: nothing replicated, nothing
  backed up, no retention. The application difference between local and
  hosted is one variable, and I verified that by running the same build
  against both.
- Two vendor-doc corrections I verified: the coding-assistant MCP transport
  key is `"type": "http"`, and "HTTPS required" does not apply to
  localhost. The trace store's MCP server exposes prompt-management tools
  only; traces are not queryable over MCP at the version in use.
