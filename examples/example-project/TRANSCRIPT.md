# Transcript

What `ask-docs`, `lint-docs`, and `test-docs` actually produce against this knowledge base.

The pages in `docs/` show what `scan-docs` produces. This file covers the other three, because
describing an output format is not the same as showing one — and for a kit whose argument is
*evidence over assertion*, that gap was the wrong one to leave open.

Run from `examples/example-project/`. Output is lightly trimmed for width, not for content.

---

## `/ask-docs "What is the absolute session TTL?"`

The question the whole example exists for. A retriever returns the more relevant of two
conflicting documents and answers confidently. This does not.

```
Selected from the index: docs/topics/session-management.md (entry point),
then one hop: docs/entities/redis.md.
Budget: 4 pages, ~3.2k tokens. Within budget.

Contradicted — the sources disagree on the absolute TTL, and no human has ruled.

  30 minutes from issue, no refresh
    ([[docs/core-sources/260415-auth-spec|260415-auth-spec]], §4.2)

  24 hours — SESSION_TTL=86400, raised from 1800 on 2026-06-02
    ([[docs/core-sources/260710-ops-runbook|260710-ops-runbook]], "Configuration in production")

The runbook explains its change: the 30-minute expiry logged users out mid-checkout during slow
payment callbacks in June. That is a reason, not a ruling — the spec has not been revised, and
no rule in DOCS.md makes a runbook authoritative over a spec. Both values stand.

Known — the idle timeout is 15 minutes, and both sources agree
  (260415-auth-spec §4.2 ; 260710-ops-runbook SESSION_IDLE).

Consulted: docs/topics/session-management.md, docs/entities/redis.md
Not consulted: the summaries — the topic page carried the answer and its citations.

To resolve this, see Human review in docs/DOCS.md. It needs a person.
```

**What to notice.** It refuses to pick, it says *why* the tempting answer is tempting, and it
separates the part that is contradicted from the part that is known. The `Consulted:` line is
auditable — you can check it read what it says it read.

---

## `/lint-docs`

```
ERROR    0
WARNING  0
INFO     1

INFO  contradictions (1)
      docs/topics/session-management.md — absolute session TTL
      260415-auth-spec §4.2 says 30 minutes; 260710-ops-runbook says 24 hours.
      Open since 2026-08-30. Needs a human ruling, not a scan.

Checked: 4 pages, 2 core sources, 3 scenarios.
  front matter parses and carries all six required fields on every page
  all field values valid
  4 citations resolve to files that exist
  2 wikilink targets resolve ([[redis]], [[session-management]])
  every page appears in docs/README.md exactly once, and has an inbound link
  no source is newer than the page that summarizes it
  no security-excluded paths in docs/core-sources/

Nothing to fix.
```

**What to notice.** The open contradiction is **INFO, not ERROR.** It is the system working, and
a merge gate set on ERROR would pass this base. Lint will never close that finding — only a
person can.

---

## `/test-docs`

```
docs/scenarios/questions.yaml — 3 scenarios

PASS  3
FAIL  0

PASS  session-revocation                                    [structural]
      expect_state: unknown ✓
      Answer said no source describes what happens to live sessions on
      password change, and named the gap.
      forbid_claims: neither "immediately invalidated" nor "all sessions are
      revoked" appeared. ✓

PASS  session-ttl                                           [structural]
      expect_state: contradicted ✓
      require_facts: "30 minutes" ✓  "24 hours" ✓
      expect_sources: both cited ✓

PASS  session-identifier                                    [structural]
      expect_state: known ✓
      require_facts: "opaque" ✓   forbid_claims: "yes" absent ✓
      expect_sources: docs/core-sources/260415-auth-spec.md cited ✓

Semantic pass not run. Add --semantic for unsupported claims, overstated
certainty, and evidence sufficiency.
```

**What to notice.** `session-revocation` is the scenario that matters. Both sources are silent on
revocation timing and ownership, and an agent that invents a plausible 90-day policy here would
still pass every schema check in `lint-docs`. Asserting that the base keeps saying *"I don't
know"* is the only way to catch that, and it is why `expect_state: unknown` is the first case in
the file.

---

## Make one fail

The transcript above is the healthy state. To see the machinery work, break something:

**Break a citation.** In `docs/topics/session-management.md`, change a source path to
`docs/core-sources/260415-auth-spec-v2.md`. `/lint-docs` should report **ERROR — broken
citation**, naming the file that does not exist.

**Break the front matter.** Delete the closing `---` from any page. `/lint-docs` should report
**one ERROR** with the corrected block, keep checking the other three pages, and refuse to
ingest the half of the metadata that still looked like YAML.

**Make a page stale.** Edit `docs/core-sources/260710-ops-runbook.md`. `/lint-docs` should set
`status: stale` on its summary and on both pages citing it, because their `updated` now predates
the source. Stale means unverified, not wrong.

**Force a scenario failure.** In `docs/scenarios/questions.yaml`, change `session-ttl` to
`expect_state: known`. `/test-docs` should FAIL it — the base is contradicted and correctly says
so, and the expectation is what is wrong. It should say that, and propose the edit rather than
making it, because `docs/scenarios/` is human-owned.
