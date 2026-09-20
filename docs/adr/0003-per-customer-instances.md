# 0003. One instance per customer

Date: 2026-07-17
Status: Superseded by [0009](0009-pooled-tenancy-with-dedicated-as-premium.md)

## Context

First public deployment. The security story for finance teams is "your data
does not mingle with anyone else's", and the simplest way to make that true
is hard isolation: one database, one backend service, one frontend project
per customer, with environment variables defining the instance.

## Decision

Multi-instance, multi-user. One Supabase project, one Railway service and one
Vercel project per customer. I rejected multi-tenancy inside a database
because database-level isolation was part of the trust pitch.

## Consequences

- Zero shared infrastructure between paying customers. Easy to explain.
- Every migration runs N times. There is no control plane, so "run the
  migration everywhere" is a manual loop, and drift between instances is
  invisible until something breaks. This is what happened: the demo instance
  sat ten migrations behind the code for two weeks before anyone noticed.
- Cost scales linearly with customers before revenue does.

Eleven days later I reversed it. See 0009.
