# Changelog

Append-only. Newest first. One entry per `scan-docs`, `ask-docs`, `lint-docs`, or `test-docs` run. Never rewrite history.

## 2026-08-30 — scan

- Scanned: docs/core-sources/260710-ops-runbook.md
- Created: docs/summaries/260710-ops-runbook.md
- Updated: docs/topics/session-management.md, docs/entities/redis.md, docs/INDEX.md
- Flagged: contradiction on absolute session TTL — 260415-auth-spec §4.2 gives 30 minutes,
  260710-ops-runbook gives 24 hours. Left open; the runbook explains the change but no rule in
  DOCS.md makes it authoritative over the spec.
- Flagged: open question on session revocation after password change. No source assigns an owner.

## 2026-08-30 — scan

- Scanned: docs/core-sources/260415-auth-spec.md
- Created: docs/summaries/260415-auth-spec.md, docs/topics/session-management.md,
  docs/entities/redis.md
- Updated: docs/INDEX.md
- Noted: the JWT rejection in §5 is recorded as a `[!check] Decision` callout citing the spec,
  not as `claim_type: decision` in front matter. A source describing a decision is evidence
  that one was made, not authority that it still holds.

## 2026-08-30 — init

- Scaffolded: docs/ with DOCS.md, README.md, CHANGELOG.md, scenarios/questions.yaml, and the
  sources, summaries, topics, entities folders
- Wrote: docs/DOCS.md from the interview
- Wrote: docs/scenarios/questions.yaml with 3 cases
