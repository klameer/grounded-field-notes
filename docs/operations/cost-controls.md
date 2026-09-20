# Cost controls

Every number here is one I measured, with the date. Undated cost numbers
are marketing.

## Where the money goes

Per answered question, production pricing at list, 122 gate answers on the
rail corpus, measured in Langfuse on 2026-08-02:

| Statistic | Cost per question |
|---|---|
| Median | $0.18 |
| p90 | $0.31 |
| Max | $0.82 |

A full double gate (two suites, three runs) is about $23 at list. Run
locally on a flat-rate subscription the cash cost of the same gate was about
$1.50, nearly all of it reranker calls.

For scale: a pilot at 20 questions a day is about $80 a month of inference
on a premium model, and 10 to 20 times less on a mini-class model.

## What it was before

Before 2026-07-22 the same question cost £1.50 to £4 through a pay-per-token
router on a frontier model with an uncached agentic loop. A day of evals
drained $50. That is the incident behind [ADR 0004](../adr/0004-retire-pay-per-token-routing-for-dev.md).

## The levers, in order of effect

1. **Prompt caching.** Merged from upstream rather than built. Measured on
   one rail-corpus turn: 60,928 cached input tokens against 17,889 fresh
   across seven generations. Steady state about 77 percent of input tokens
   from cache. Retail cost would be three to four times higher without it.
   The benchmark harness uses the same trick: the document prefix is cached
   so question N pays roughly a tenth for document tokens.
2. **Route by call class.** Chat on the strongest model; metadata
   extraction, the eval judge and citation checks on a mini-class model
   (judge verdict about $0.0002). Embeddings on a dedicated endpoint.
3. **Never re-ingest casually.** Ingestion runs embeddings plus a per-file
   LLM metadata pass. A corpus sync deletes and replaces wholesale. Rule:
   no delete-and-reupload without a file count, a cost estimate and an
   explicit yes.
4. **Evals are manual and subset-able** ([ADR 0014](../adr/0014-evals-never-run-automatically-in-ci.md)).
   Against the hosted instance, spot-check single questions, never full
   suites.
5. **Reranker is cheap, keep it.** Under a cent per question at a median of
   four searches. See the [ablation](../../evals/ablations/rerank-on-off.md).

## Provider switches: the sweep rule

Switching chat providers in production broke three things that were not
obviously provider-specific: a max-tokens variable the new model rejected, a
citation-check model id in the old provider's naming scheme (every citation
grade came back as verification failed), and a sub-agent model id in the
same scheme. My rule now: after any provider switch, sweep every model and
token variable, not just the chat ones, and confirm a new deployment went
active.
