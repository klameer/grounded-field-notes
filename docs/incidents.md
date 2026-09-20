# What broke, how I found it, what changed

Eight incidents from July to September 2026. Each one produced a rule that
is still in force. Symptom, detection, fix, rule.

## 1. The reranker had never run

**Symptom.** None. Retrieval looked fine. That is the point.

**Detection.** I wrote a retrieval-only eval (hit@1, hit@3, MRR against
expected sources, no chat, no judge) to measure a chunking change, and ran
it with the reranker toggled. The "off" and "on" numbers were identical.
The reranker's base URL setting was empty, and empty meant disabled. The
client's error path returned the unranked list with a log line, so nothing
had ever failed loudly.

**Fix.** Production key, base URL set, retries on 429 and 5xx honouring
Retry-After, sixteen unit tests on a client that had none. hit@1 went from
0.42 to 0.75 on one corpus and 0.44 to 0.72 on the other.

**Rule.** A stage that can silently degrade to a no-op gets a test that
proves it ran. The retrieval eval is the first thing I run after any
retrieval change. [ADR 0010](adr/0010-turn-on-the-reranker.md).

## 2. $50 of evals in a day

**Symptom.** The pay-per-token account was empty by evening. One eval
question against the hosted demo cost £1.50 to £4.

**Detection.** The 402 from the provider, mid-demo.

**Fix.** Chat on a flat-rate subscription for development, a mini-class
model for the judge and metadata calls, and a standing rule that no corpus
is deleted and re-ingested without a file count and a cost estimate.

**Rule.** Cost is measured per question in the trace store, never
discovered per invoice. Evals must be cheap enough to run casually or they
stop being run. [ADR 0004](adr/0004-retire-pay-per-token-routing-for-dev.md),
[cost controls](operations/cost-controls.md).

## 3. The eval passed an answer the user never saw

**Symptom.** A revenue figure of $1,975.1M, wrong, was refused by the
numeral guard. The eval marked the question as passed.

**Detection.** I was watching the demo-prep run and saw a refusal on screen
next to a green row in the report. The runner graded the raw model output,
before the guard.

**Fix.** The runner grades the post-guard answer text, refusals included.
Pass rates dropped on the next run and recovered as I fixed real issues.

**Rule.** Evals see exactly what users see. Applied again in the public
benchmark: the product is graded on its rendered answer.
[ADR 0006](adr/0006-evals-grade-what-the-user-sees.md).

## 4. PII surrogates rewrote the question

**Symptom.** Every date-bearing question on the hosted demo came back with
"those dates match no document".

**Detection.** A prospect demo. The date-time surrogate had replaced
"December FY26" in the question with a random fake date before the model
saw it.

**Fix.** A test that pins numerals through the anonymise and de-anonymise
round trip, and a per-instance switch so a fully synthetic corpus can run
with redaction off.

**Rule.** Surrogates never touch anything that could be a figure. Verify
the redaction setting before every demo. [PII pipeline](security/pii-pipeline.md).

## 5. The demo database was ten migrations behind

**Symptom.** Features that worked locally errored on the hosted demo.

**Detection.** Merging the trust-chain work. A dry run of migrations
against the cloud project listed ten pending, going back two weeks. Code
had auto-deployed on every push; migrations were manual and nothing checked
drift.

**Fix.** Applied the ten, additive first.

**Rule.** Dry-run migrations before every promotion and apply pending ones
before the code that needs them. Migrations go first, code second.
[ADR 0007](adr/0007-roll-forward-migrations.md).

## 6. Twenty-six days of failed deployments that looked successful

**Symptom.** Production was running month-old code. Merges to the
production branch reported success.

**Detection.** A demo session where a feature merged three weeks earlier
was missing. The hosting dashboard's "deployment successful" banner
belonged to the old active deployment; every new one had failed its health
check and was visible only in history. Cause: a boolean setting set to an
empty string, which the settings parser rejected on boot.

**Fix.** Empty string to `false`. Purged four stale variables from the old
provider while I was there, one of which had been making every citation
grade come back as "verification failed".

**Rule.** After every promotion, confirm a new deployment went active.
Merge success is not deploy success. After any provider switch, sweep every
model and token variable, not just the chat ones.
[deployment](operations/deployment.md).

## 7. The parser took the API down

**Symptom.** One PDF upload, backend dead, every in-flight ingestion stuck
in "processing" forever.

**Detection.** Health check failed on the host; the stuck rows then blocked
the retry endpoint, which tempted exactly the delete-and-reupload cycle
that costs money.

**Fix.** Parsing in an isolated process, converter recycled per document,
preflight limits on page and image pixels, 50-page windows. A sweeper for
stuck rows is designed and not yet built; I still clear them by hand.

**Rule.** Size the heaviest dependency import before choosing a host. The
same parser had already exhausted two memory tiers on the previous host.
[ingestion](architecture/ingestion.md), [ADR 0008](adr/0008-the-documents-table-is-the-queue.md).

## 8. The benchmark ruled against my own product

**Symptom.** Not a failure. A finding.

**Detection.** Adjudicating the Board Pack Test baselines under rules I
had pre-registered. One product run cited a superseded tracker whose
figure happened to be right. Under the rules, a superseded document is not
a valid source. Another product run cited the wrong sheet on a question
where the answer key itself had a wrong trace; correcting the key reversed
three denials for the raw competitor and upheld the denial for my product.

**Fix.** Authority-aware ranking that demotes superseded documents before
rerank, and figure-free refusals. Together they moved the auto score 13 to
15 points in a day.

**Rule.** Rulings are pre-registered and recorded, and they cut both ways.
A benchmark I would not publish a loss on is not a benchmark.
[ADR 0013](adr/0013-benchmark-is-a-neutral-instrument.md),
[results](../evals/results/2026-08-08-board-pack-test-v1.0.1.md).
