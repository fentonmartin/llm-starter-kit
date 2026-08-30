---
type: summary
title: Operations Runbook — Sessions
status: active
claim_type: fact
created: 2026-08-30
updated: 2026-08-30
sources: [docs/sources/260710-ops-runbook.md]
confidence: high
---

# Operations Runbook — Sessions

## What it is

The operational runbook for session infrastructure, revised 2026-07-10 after an incident in
June. Unlike [[docs/sources/260415-auth-spec|260415-auth-spec]], it describes what production actually runs rather than what
was designed.

## Main claims

- [[redis]] runs as a single primary with one replica, database 0, key prefix `sess:`.
- Production configuration: `SESSION_TTL=86400` (24 hours), `SESSION_IDLE=900` (15 minutes),
  `REDIS_MAXMEMORY_POLICY=noeviction`.
- **`SESSION_TTL` was raised from 1800 to 86400 on 2026-06-02.** The spec still specifies 1800.
  See the contradiction on [[session-management]].
- Draining the Redis primary destroys all live sessions and logs out every user.

> [!question] Open question
> Password change does not invalidate existing sessions ("Known gaps"). A user who changes
> their password after suspecting compromise remains logged in on the attacker's device until
> the TTL expires — now up to 24 hours. The runbook records this as tracked but not scheduled;
> no source says who owns it or when it will be fixed.

## Evidence

The TTL change is the only claim with stated support: the 30-minute expiry logged users out
mid-checkout during slow third-party payment callbacks, and support volume dropped after the
change. The drop is reported, not quantified.

## Limits

- **Records the change without recording the decision.** It says the TTL was raised and why it
  was raised, but not who approved it or whether the spec was meant to follow. That is exactly
  why the contradiction with [[docs/sources/260415-auth-spec|260415-auth-spec]] stays open.
- No date on the "known gaps" entry, so how long revocation has been outstanding is unknown.
- Covers only session infrastructure, not the wider authentication flow.
