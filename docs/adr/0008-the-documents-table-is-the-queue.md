# 0008. The documents table is the ingestion queue

Date: 2026-07-28
Status: Accepted as a design, not yet built. Stuck rows are still cleared by hand.

## Context

Ingestion runs as in-process background tasks on the API server, bounded by
a per-process semaphore. When the backend restarts mid-ingest (every deploy
does this), the in-flight document is left with status "processing" forever.
A stuck row blocks the retry endpoint, and a stuck row tempts the operator to
delete the document and upload it again, which re-runs embeddings and the
metadata pass and costs money. The question was whether to add a proper
queue (Redis, a worker service) to fix it.

## Decision

No Redis. The documents table is the queue.

- Add a heartbeat timestamp and an attempt counter to the documents row.
- Partial index on in-flight rows only.
- A sweeper reclaims rows whose heartbeat is older than a stale threshold
  and re-queues them, up to a hard attempt cap, after which the row is
  marked failed with the reason.
- The stale threshold (default 300 seconds) is chosen to exceed the longest
  healthy gap between heartbeats (one embedding batch of about 50 chunks)
  plus the platform's deploy-overlap and termination grace window.

## Consequences

- One fewer service to run, monitor and pay for.
- The sweeper is idempotent, so a startup sweep covers the restart case with
  no extra mechanism.
- Known limit, recorded in the tenancy decision (0009): semaphores do not
  coordinate across replicas, so N replicas give N times the intended
  concurrency. Before pooling tenants, ingestion moves to its own worker.
  The queue stays in the table either way.

## Alternatives considered

- Redis plus a worker. Correct, and premature at this scale. Revisit when
  ingestion moves out of process.
- Manual SQL to unstick rows. What was happening before. Not a design.
