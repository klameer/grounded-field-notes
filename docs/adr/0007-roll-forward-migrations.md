# 0007. Roll forward, never roll back a live migration

Date: 2026-07-28
Status: Accepted as policy. The destructive-migration scan is not yet a CI job; the dry-run check is done by hand before every promotion.

## Context

Designing the deployment pipeline. Application code can be rolled back by
re-pointing a deployment at a previous immutable image. A database cannot:
rolling back a migration on a live database that has taken writes since is
where data gets lost.

## Decision

- Deployments pin image tags, not branches. A rollback is a re-pin, not a
  rebuild.
- Migrations are never rolled back on a live database. A bad migration is
  fixed by a new migration that rolls forward.
- This is only safe if migrations are non-destructive by default, so a CI
  job scans every migration for DROP TABLE, DROP COLUMN, TRUNCATE and column
  type changes, and fails unless the file carries an explicit
  `-- allow-destructive: <reason>` marker. Expand and contract is the
  pattern: add the new column, backfill, switch the code, drop the old
  column in a later release.
- Cloud migrations are applied by hand, before the code that needs them is
  promoted, and a dry run is checked before every promotion.

## Consequences

- Schema changes take two releases instead of one. Accepted.
- The drift incident (demo database ten migrations behind for two weeks)
  produced the standing dry-run rule. Nothing checked drift automatically
  before; the dry run now does.
- A later incident showed that a successful merge is not a successful
  deploy: a platform variable set to an empty string crashed the settings
  parser on boot and every promotion for 26 days failed its health check
  while the old deployment kept serving. The rule that followed: after every
  promotion, confirm a new deployment went active, not just that the merge
  succeeded.
