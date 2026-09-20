# Provenance

What in Grounded is mine, what is not, and what is deliberately absent from
this repository.

## The starting point

Grounded is a fork of a retrieval-augmented-generation application built
along with a paid course. The course codebase is itself actively developed
by its authors, and my fork tracks it as an upstream. Reading the app's git
history you will see roughly 420 commits by the course authors from January
to June 2026 and roughly 100 by me from 17 July 2026 onward.

Being precise about the line matters more to me than where it falls.

**The scaffold, as it stood when I forked it**, already had: a FastAPI
backend with an agentic tool loop, a React front end, Supabase with pgvector
and full-text search fused by reciprocal rank fusion, Docling-based
ingestion with LLM metadata extraction, a reranker client, sub-agents, a
folder system, a skills system, a domain-harness engine, thread compaction,
two citation modes with inline chips, Presidio-based PII redaction with
reversible surrogates, a sandbox, a Tavily web-search tool, tracing hooks,
and a provider that runs chat on a coding-assistant subscription. That is a
serious piece of software and I am not going to pretend otherwise.

**What I built on it** is everything that makes it a finance product a
controller could sign off on:

| Area | Mine |
|---|---|
| Verification | Post-answer numeral guard with refuse mode; deterministic calculator with verified input bindings; provenance-fed guard; trust-chain strip and derived-value chips; figure-free refusals |
| Spreadsheets | Native openpyxl extractor keeping the grid, sheet and A1 reference per cell; spreadsheet viewer with cited-cell highlighting; per-row citation fallback; exact-cell pinning |
| Retrieval quality | Turning the reranker on, with retries and tests; the retrieval-only A/B eval; authority-aware ranking that demotes superseded documents |
| Trust tiers | Notebooks as a parallel notes table; source-of-record, commentary and analyst-note tiers on every citation |
| Evals | The golden-question runner with code-asserted sources and a stochastic-safe gate; the in-app evals page; the organisation golden pack; the Board Pack Test benchmark and its grader |
| Tenancy and secrets | Organisation multi-tenancy with org-scoped retrieval and citation parity; envelope-encrypted per-org model keys; the 95 security and isolation tests |
| Audit | Audit packages that survive thread deletion; the audit page |
| Integration | Grounded as an MCP server; the Slack bot; the confirm-before-ingest local-file tool; the six-skill month-end close suite |
| Operations | The per-instance deployment runbook and its Railway move; porting the Langfuse instrumentation and writing the observability notes; cost alerts; the tiered test runner and pre-commit hook |
| Harnesses | The advisory harness (five-phase deep research to a three-tier cited report) |

Everything in `docs/adr/` is a decision I took. Where a decision was to
adopt something from upstream (prompt caching, for instance) the record says
so.

## What is not here

- **No application code.** The course licence permits use, modification and
  deployment, and forbids publishing the software or its documentation in
  any public repository. Every document here was written fresh, about my
  system, from my own design notes and measurements. No file in this
  repository is derived from the course's documentation.
- **No client data.** Every corpus mentioned is fictional: a rail operator
  built from public annual reports, a biotech, and Caldergate Distribution
  Group, the fictitious UK distributor behind the Board Pack Test. Screenshots
  are taken on those corpora only.
- **No pricing, no pipeline, no go-to-market.** This is an engineering
  record. The product lives at [codelessops.com](https://codelessops.com).

## Related public repositories

- [test-your-finance-llm](https://github.com/klameer/test-your-finance-llm):
  the Board Pack Test. Pack, questions, key, grader, adjudicated results.
  CC BY 4.0.
- [audited-ai-close](https://github.com/klameer/audited-ai-close): a
  month-end close run by Claude with reviewer gates and a sealed audit
  binder. MIT.

## Licence

The text and diagrams in this repository are licensed
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Eval result data
files are CC BY 4.0 as well. Nothing here grants any right to the Grounded
software.
