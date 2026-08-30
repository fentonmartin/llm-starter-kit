---
type: topic
title: Session management
status: active
claim_type: contradiction
created: 2026-08-30
updated: 2026-08-30
sources: [docs/sources/260415-auth-spec.md, docs/sources/260710-ops-runbook.md]
confidence: medium
---

# Session management

How the web application keeps a user logged in, and for how long.

## Session identity

On successful password verification the server issues an opaque 256-bit random session
identifier ([[docs/sources/260415-auth-spec|260415-auth-spec]], §2). It carries no user data. JWTs were considered and
rejected: revocation would have required a denylist, which reintroduces the shared state that
stateless tokens exist to avoid ([[docs/sources/260415-auth-spec|260415-auth-spec]], §5).

## Storage

Sessions live in [[redis]], keyed by identifier, holding user id, issue time, and originating IP
([[docs/sources/260415-auth-spec|260415-auth-spec]], §3). Production runs a single primary with one replica on database 0,
key prefix `sess:` ([[docs/sources/260710-ops-runbook|260710-ops-runbook]], "Where sessions live").

Eviction is disabled — `REDIS_MAXMEMORY_POLICY=noeviction`, so sessions are never dropped under
memory pressure ([[docs/sources/260710-ops-runbook|260710-ops-runbook]], "Configuration in production").

## Expiry

Idle timeout is 15 minutes, and both sources agree ([[docs/sources/260415-auth-spec|260415-auth-spec]], §4.1;
[[docs/sources/260710-ops-runbook|260710-ops-runbook]], `SESSION_IDLE`).

The absolute TTL does not agree:

> [!warning] Contradiction
> [[docs/sources/260415-auth-spec|260415-auth-spec]] (§4.2) specifies an absolute TTL of **30 minutes** after issue.
> [[docs/sources/260710-ops-runbook|260710-ops-runbook]] (`SESSION_TTL`) records **24 hours** in production, raised from 1800
> seconds on 2026-06-02.
> Unresolved as of 2026-08-30.

The runbook gives a reason for the change — the 30-minute expiry logged users out mid-checkout
during slow payment callbacks in June. That is suggestive, but it is not a ruling. The spec has
not been revised, and nothing in `DOCS.md` says the runbook supersedes it. Both values stand
until a human decides.

*A human ruling would rewrite this section into the Decision / Rationale / Evidence shape in
the Human review section of `docs/DOCS.md`, keeping both values on the page.*

## Restarting Redis

Draining the primary destroys every live session and logs out every user, so it is scheduled
outside business hours ([[docs/sources/260710-ops-runbook|260710-ops-runbook]], "Restart procedure").

## Open questions

> [!question] Open question
> Password change does not invalidate existing sessions ([[docs/sources/260710-ops-runbook|260710-ops-runbook]], "Known gaps").
> A user who changes their password after suspecting compromise stays logged in on the
> attacker's device until the TTL expires. Tracked, not scheduled — and with the TTL at 24
> hours rather than 30 minutes, the exposure window is 48 times longer than the spec assumed.
