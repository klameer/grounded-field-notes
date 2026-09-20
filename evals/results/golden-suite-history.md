# Golden-question suite history

Regression runs on the internal corpora. Each row is a full suite, three
runs unless stated. These are not comparable to the benchmark table; they
answer "did the last change break anything".

| Date | Event | Rail corpus | Biotech corpus | Notes |
|---|---|---|---|---|
| 2026-07-21 | Computation phase gate | 9/9 stability, 13/13 full | | All perfect. Small suite. |
| 2026-07-27 | Demo build | 47/51 (92%) | | Four flappers root-caused and fixed the same hour. Runner switched to grade the post-guard answer (ADR 0006). |
| 2026-08-02 | After upstream merge (prompt caching, PII surrogates) | 49/51 (96%) | 65/71 (92%) | |
| 2026-08-02 | After org tenancy (ADR 0009) | 50/51 (98%) | 66/72 (92%) | Organisation-scoped retrieval, citation parity. |
| 2026-08-07 | Board-pack corpus, in-app calibrated series | | | 93 to 96% across runs. Internal regression metric only, kept out of the benchmark league table. |

Remaining known failures on the biotech corpus are two questions whose
expected source is a chart image, which the product does not read by design
(ADR 0005).
