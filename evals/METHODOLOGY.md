# Evaluation methodology

Three instruments, each answering a different question. Results are
committed as dated files under [results/](results/) so the history lives in
git.

## 1. Golden questions (does the product still work?)

A YAML suite per corpus. Each question carries:

| Field | Purpose |
|---|---|
| `type`, `tier` | lookup, cross-document, computation, version, refusal |
| `expected_answer`, `must_mention`, `tolerance_note` | what a correct answer contains |
| `expected_sources` (with `any_of`) | which document, sheet or page a correct answer must cite |
| `abstention_expected` | true for questions whose answer is not in the corpus |
| `rubric: {value, source}` | points split between the figure and its citation |

The runner drives the live chat endpoint over server-sent events, one fresh
thread per question, and grades the post-guard answer the user would see
(ADR 0006). An LLM judge on a mini-class model scores against the rubric
(about $0.0002 per verdict); citation assertions are deterministic. A
preflight step checks every `expected_sources` entry exists in the corpus
before spending a token.

Flags: `--runs N` for stability (three-run clean is the demo gate),
`--min-pass-rate` (default 0.8), `--check-citations`, `--deep`.

Every suite contains refusal questions. A digit-free refusal is the passing
answer for those; any figure is a fail.

The same questions live in the database as the organisation's golden pack,
runnable from the app's evals page. The repository YAML stays the
engineering gate; the in-app pack is the installation's acceptance contract.

Suite sizes at last count: rail corpus 17 questions, biotech corpus 24,
board-pack corpus 32.

## 2. Retrieval-only eval (did retrieval change?)

Embeds each question, runs hybrid retrieval, and scores hit@1, hit@3, hit@k
and MRR against `expected_sources`. No chat, no judge. Costs embedding calls
only, runs in seconds, and is the first check after any change to chunking,
embeddings, fusion or reranking. Known limit: document-level truth, not
chunk-level.

## 3. The Board Pack Test (how does it compare?)

A public benchmark, published separately as a neutral instrument
(ADR 0013): 34 documents from one deterministic financial model, 25
questions across five tiers, a deterministic grader, a 25-question private
holdout. The product sits it the same way any other agent does: documents
in, one fresh conversation per question, three independent passes, median
reported.

Scoring is 25 × 4 = 100, split into 81 machine-verifiable points (values,
citations, digit-free refusals) and 19 judgment points awarded under
pre-registered rules. The auto column is the headline because the judgment
column saturates for every frontier tool.

Repository: [klameer/test-your-finance-llm](https://github.com/klameer/test-your-finance-llm).
Essay: [The Board Pack Test](https://codelessops.com/posts/the-board-pack-test).

## What each instrument is not for

- Golden questions are regression, not comparison. The in-app calibrated
  series on the board-pack corpus (93 to 96 percent) is deliberately kept
  out of the benchmark league table; it runs under different conditions.
- The retrieval eval cannot see answer quality. A perfect MRR with a wrong
  answer is a generation problem, and it happens.
- The benchmark's public set is contaminable from the day it was published.
  Baselines were run before publication; later runs are labelled by date.
