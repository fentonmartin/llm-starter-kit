---
type: summary
title: Authentication Specification v1.2
status: active
claim_type: fact
created: 2026-08-30
updated: 2026-08-30
sources: [docs/sources/260415-auth-spec.md]
confidence: high
---

# Authentication Specification v1.2

## What it is

The platform team's session-handling specification for the web application, drafted 2026-04-15.
It is a design document — it states what the system should do. Whether production matches it is
a separate question, and on one point it does not.

Scope is web sessions only. API tokens and service-to-service authentication are excluded (§1).

## Main claims

- Sessions are opaque 256-bit random identifiers carrying no user data. Not JWTs (§2).
- Session state lives in [[redis]], keyed by identifier, storing user id, issue time, and
  originating IP (§3).
- Idle timeout is 15 minutes (§4.1).
- **Absolute TTL is 30 minutes from issue, with no refresh** (§4.2). This conflicts with
  production — see [[session-management]].

> [!check] Decision
> JWTs were considered and rejected (§5). Revocation would have required a denylist, which
> reintroduces the shared-state dependency stateless tokens are meant to remove.
>
> Recorded as evidence that the platform team decided this, not as an authority on what the
> project does now. Nobody has confirmed the decision still stands.

## Evidence

None. This is a specification, not a study — it asserts a design rather than measuring one. The
JWT rejection in §5 is the only claim carrying a stated reason.

## Limits

- **Prescriptive, not descriptive.** Reading it as a description of running behavior is the
  mistake it invites, and the reason the TTL contradiction exists.
- Last revised 2026-04-15, before the June incident that changed the production TTL. Nothing in
  the document reflects that change.
- Silent on session revocation. The gap surfaces in [[docs/sources/260710-ops-runbook|260710-ops-runbook]], not here.
