# Grounded: field notes

Engineering notes on a production retrieval-augmented-generation system for
finance teams. Decision records, architecture, evaluation results, cost and
latency measurements, and the incidents that shaped them. No application
code; see [PROVENANCE](PROVENANCE.md) for why.

**Grounded** answers questions over a company's own finance documents
(board packs, management accounts, forecasts, statements) and never states
a number without a citation that resolves to a cell or a page, or a shown
computation whose inputs do. The product site is
[codelessops.com](https://codelessops.com). This repository is the
engineering record behind it.

## Headline numbers

On the [Board Pack Test](https://github.com/klameer/test-your-finance-llm),
a public benchmark of 25 questions over 34 finance documents with a
deterministic grader, run 2026-08-08 before publication:

| Agent | Machine-verifiable points (of 81) | Citations a machine can confirm |
|---|---|---|
| Grounded v2 on Claude Opus 5 | **81**, zero variance across 3 runs | 60 / 60 |
| Grounded v2 on GPT-5.5 | **81** | 59 / 60 |
| GPT-5.5, raw with code interpreter | 71 | 53 / 60 |
| Claude Opus 5, raw with code sandbox | 66 | 59 / 60 |

After a human judgment pass every frontier tool lands between 95 and 100,
and the best raw model beat the product by one point. The product's edge
is in what survives verification with no human in the loop, which is the
point. Full table, conditions and the rulings that went against the
product: [evals/results](evals/results/2026-08-08-board-pack-test-v1.0.1.md).

| Measure | Value | Source |
|---|---|---|
| Cost per answered question, production pricing | $0.18 median, $0.31 p90 | [cost controls](docs/operations/cost-controls.md) |
| Input tokens served from cache | about 77 percent | same |
| Reranker lift, hit@1 | 0.42 → 0.75 (rail), 0.44 → 0.72 (biotech) | [ablation](evals/ablations/rerank-on-off.md) |
| Latency, lookup question | 12 to 20 s | [retrieval and answering](docs/architecture/retrieval-and-answering.md) |
| Unit tests, fast tier | 925 passing in about 20 s | [testing](docs/operations/testing.md) |
| Security and isolation tests | 95 | [isolation](docs/security/isolation.md) |

## Start here

1. [ADR 0001: files are the source of truth](docs/adr/0001-files-are-the-source-of-truth.md).
   The decision everything else follows from, and why the obvious design
   (extract every figure into a table) was rejected.
2. [What broke, how I found it, what changed](docs/incidents.md). Eight
   incidents and the rule each one left behind.
3. [ADR 0012: the trust chain](docs/adr/0012-trust-chain.md). A citation
   is an audit; a verification is a proof. How every link is checked by
   code.
4. [The path of one question](docs/architecture/c4-components.md). Sequence
   diagram from question to verified answer.
5. [Board Pack Test results](evals/results/2026-08-08-board-pack-test-v1.0.1.md).
   Including what moved the score 13 to 15 points in one day.
6. [ADR 0009: pooled tenancy](docs/adr/0009-pooled-tenancy-with-dedicated-as-premium.md).
   A decision I reversed eleven days after taking it, and what it cost.

## What I would do differently

- **Build the eval before the feature, not after.** The retrieval eval
  took an afternoon and found a reranker that had never run. Every stage
  that can silently degrade to a no-op should have had a test on day one.
- **Make evals cheap from the start.** I lost a month of eval discipline
  because each run cost pounds. The moment they became free I ran them
  constantly, and quality moved.
- **Skip the fact-table detour.** I spent the first roadmap building toward
  a design I already knew from planning-systems work would launder errors.
  The files-first rule (ADR 0001) should have been the opening decision.
- **Decide tenancy once.** One instance per customer felt like the honest
  security answer and lasted eleven days. Pooled with a dedicated premium
  tier is where it landed and where it should have started.
- **Treat every empty environment variable as a bug.** A blank boolean
  cost me 26 days of failed deployments that reported success.
- **Publish the benchmark on day one.** The Board Pack Test was the most
  useful thing I built and the last thing I started. Every product change
  in August came from it.

## Map

```
PROVENANCE.md            what is mine, what is the scaffold, what is absent
docs/
  incidents.md           eight things that broke, and the rule each left behind
  adr/                   14 decision records, Nygard format, never edited after acceptance
  architecture/          C4 context, containers, components; ingestion; retrieval and answering
  security/              isolation layers and tests; the PII pipeline
  operations/            observability, deployment, cost controls, testing
evals/
  METHODOLOGY.md         three instruments and what each is not for
  results/               dated result files, benchmark and golden-suite history
  ablations/             reranker on/off
assets/                  screenshots on fictional corpora
CHANGELOG.md
```

## Stack, in one line

FastAPI and Python 3.13; React 18, TypeScript and Vite; Supabase (Postgres,
pgvector, full-text search, auth, storage); Docling and openpyxl for
ingestion; a cross-encoder reranker; frontier chat models behind a provider
facade with a flat-rate subscription path for development; Presidio for
PII; Langfuse for traces; Railway and Vercel for hosting; Playwright and
pytest.

## Related

- [test-your-finance-llm](https://github.com/klameer/test-your-finance-llm):
  the benchmark, as a neutral instrument.
- [audited-ai-close](https://github.com/klameer/audited-ai-close): a
  month-end close run by Claude with reviewer gates and a sealed binder.
- Essays: [The Board Pack Test](https://codelessops.com/posts/the-board-pack-test),
  [How AI finds your numbers](https://codelessops.com/posts/how-ai-finds-your-numbers/),
  [Use AI without leaking data](https://codelessops.com/posts/use-ai-without-leaking-data).

## Author

Karim Lameer. CIMA-qualified accountant, Master Anaplanner, fifteen years
in FP&A and planning systems before building this.
[GitHub](https://github.com/klameer) ·
[LinkedIn](https://www.linkedin.com/in/karimlameer) ·
[codelessops.com](https://codelessops.com)

Text and data in this repository: [CC BY 4.0](LICENSE).
