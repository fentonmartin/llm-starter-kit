---
type: entity
title: Redis
status: active
claim_type: fact
created: 2026-08-30
updated: 2026-08-30
sources: [docs/sources/260415-auth-spec.md, docs/sources/260710-ops-runbook.md]
confidence: high
---

# Redis

The in-memory key-value store that holds every live web session for this application.

## Deployment

Single primary with one replica ([[docs/sources/260710-ops-runbook|260710-ops-runbook]], "Where sessions live"). Database 0.

## What it stores

Session records under the key prefix `sess:`, keyed by the opaque session identifier. Each value
is a serialized map of user id, issue time, and originating IP ([[docs/sources/260415-auth-spec|260415-auth-spec]], §3).

## Configuration

| Setting | Value | Source |
|---|---|---|
| `SESSION_TTL` | `86400` seconds — 24 hours (since 2026-06-02) | [[docs/sources/260710-ops-runbook\|260710-ops-runbook]] |
| `SESSION_IDLE` | `900` seconds — 15 minutes (2026-07-10) | [[docs/sources/260710-ops-runbook\|260710-ops-runbook]] |
| `REDIS_MAXMEMORY_POLICY` | `noeviction` (2026-07-10) | [[docs/sources/260710-ops-runbook\|260710-ops-runbook]] |

`SESSION_TTL` is disputed. The runbook value above is what production runs; the spec specifies
30 minutes. See the contradiction on [[session-management]].

## Operational consequences

Redis is a hard dependency for being logged in, not a cache. Two things follow:

- Draining the primary logs out every user, so restarts are scheduled outside business hours
  ([[docs/sources/260710-ops-runbook|260710-ops-runbook]], "Restart procedure").
- Eviction is disabled deliberately. Under memory pressure Redis refuses writes rather than
  dropping sessions ([[docs/sources/260710-ops-runbook|260710-ops-runbook]], "Configuration in production").

## Where it appears

- [[session-management]] — storage and expiry
