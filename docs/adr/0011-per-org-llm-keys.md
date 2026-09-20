# 0011. Per-organisation LLM keys, no per-organisation base URL

Date: 2026-08-02
Status: Accepted

## Context

With pooled tenancy (0009), some organisations want chat to run on their own
model account: their enterprise agreement, their data-retention terms, their
bill. That means storing a third party's API key in the shared database.

## Decision

- Organisation keys are envelope-encrypted at rest: a per-row data key
  wrapped by a master key held in the environment, never in the database.
  Same pattern the PII vault uses. Readable by the service role only.
- The model is pinned from an allow-list, not free text.
- There is no organisation-settable base URL. A tenant who can point the
  server at an arbitrary URL with a shared key has a server-side request
  forgery primitive. The provider base URL is an instance setting.
- Chat path only. Embeddings and utility calls stay on the instance's keys,
  because there is one vector space per database and mixing embedding
  models inside it would corrupt retrieval.

## Consequences

- A tenant on their own key gets their own bill and their own retention
  terms for the chat model, which is the part that sees their documents.
- Rotating the master key means re-wrapping every row. Scripted, not yet
  needed.
- A key-generation script existed in the scaffold before this work and
  was wired to nothing. I treated it as scaffolding, not a foundation.
  This decision is the first real use of encryption at rest in the app.
