# Index

This is the index of generated pages. Raw material lives in [core-sources/](core-sources/) and is not
listed here.

Every page in `summaries/`, `topics/`, and `entities/` is listed here exactly once.
Maintained by the agent; `lint-docs` reports anything missing or dangling.

## Summaries

- [260415-auth-spec](summaries/260415-auth-spec.md) — the platform team's session design. Prescriptive; predates the June TTL change.
- [260710-ops-runbook](summaries/260710-ops-runbook.md) — what production actually runs, revised after the June incident.

## Topics

- [session-management](topics/session-management.md) — how sessions are issued, stored, and expired. **Holds an open contradiction on the absolute TTL.**

## Entities

- [redis](entities/redis.md) — the key-value store holding every live session. A hard dependency, not a cache.
