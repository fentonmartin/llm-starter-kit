---
name: test-docs
description: Run the scenarios in docs/scenarios/questions.yaml against the knowledge base — whether each still retrieves the right sources, states the required facts, avoids the forbidden claims, and admits what it does not know. Use when the user says test the docs, run the scenarios, eval or evaluate the knowledge base, or asks whether the docs still answer correctly after a change.
---

# Test

`lint-docs` checks whether the knowledge base is well-formed. **`test-docs` checks whether it is
still right.** A base can pass every schema check and quietly stop answering the questions it
was built to answer — a source gets superseded, a page goes stale, a merge drops a citation.

Read `docs/DOCS.md` **Part 1** first — it is the complete rules. Part 2 is examples and
rationale, read on demand.

**The declarations.** Part 1 of `docs/DOCS.md` opens with three paths. Every path in this skill is
written using their defaults; read them as whatever Part 1 declares:

| Written here | Declared as |
|---|---|
| `docs/…` | **Root** — `docs/` unless the base was scaffolded beside a published docs site. |
| `docs/core-sources/` | **Source folder** — one folder: `docs/core-sources/`, a top-level `sources/`, or wherever the material already lives, including outside the root. Immutable to you wherever it is. Part 1 may also list *read-in-place sources*, which you cite but never file. |
| `docs/INDEX.md` | **Index** — `docs/INDEX.md` or `docs/README.md`. |

If the user passed a root as an argument, read `<root>/DOCS.md` and use that base only.

**Page layers may have subfolders.** `docs/topics/auth/session-management.md` is a topic page like
any other: the layer folder decides what a page is, however deep it sits. Slugs are unique across a
whole layer, so a wikilink resolves by slug and does not care about the path. The layers, their
names, and their rules never change with any declaration.

**Commits.** Part 1 also declares whether you commit your own work — `none` by default. This
command writes only a changelog entry; at `per-run` or `per-file`, stage that file alone, and
never push.

**Address.** Part 1 declares what to call the owner. Use it when you speak to them directly —
opening an answer, flagging something that needs them — and not in every sentence. If it reads
`(not set)`, say *"you"* and never guess a name from git history or an email address. It never
appears inside a page.

## Honest scope

Every check here runs by reading files and reasoning about them. There is no program, and
nothing in this kit executes.

That has a consequence worth stating plainly rather than papering over: **these results are
not bit-reproducible, and they cannot gate CI.** Two runs may word the same finding
differently. The structural checks below are far more stable than the semantic ones because
they compare paths and strings rather than meaning, but "more stable" is not "deterministic."

Treat test output as a considered review, not as a test suite exit code.

## Input

`docs/scenarios/questions.yaml`. The owner writes it; you never edit it. Each case:

```yaml
- id: session-ttl
  question: What is the session TTL?
  expect_sources:
    - docs/core-sources/260710-ops-runbook.pdf
  require_facts:
    - "24 hours"
  forbid_claims:
    - "30 minutes"
  expect_state: known        # known | contradicted | unknown
```

Every field except `id` and `question` is optional. A case with only a question still checks
that the question is answerable at all.

| Field | Checks |
|---|---|
| `expect_sources` | These paths appear in the answer's citations. |
| `require_facts` | Each string appears in the answer. |
| `forbid_claims` | None of these strings appears. Use it to pin down a known wrong answer. |
| `expect_state` | The answer's overall state matches — `known`, `contradicted`, or `unknown`. |

`expect_state: unknown` is the most valuable case in the file. It asserts that the base admits
a gap instead of inventing an answer, which is the failure this kit exists to prevent, and it
is the one no other check catches.

## Procedure

For each case, run `ask-docs` on the question **exactly as written**, under normal rules — same
retrieval, same budget, same citation contract. Do not read ahead to the expectations and do
not steer toward them. A scenario that helps itself pass measures nothing.

Then score in two passes.

### Pass 1 — structural

Comparisons of paths and strings. These are the ones to trust.

| Check | Fails when |
|---|---|
| **Citation validity** | The answer cites a path that does not exist, or one it did not open. Hard failure — a fabricated citation is worse than a wrong answer, because it survives review. |
| **Source coverage** | An `expect_sources` path is missing from the citations. |
| **Required facts** | A `require_facts` string is absent. |
| **Forbidden claims** | A `forbid_claims` string is present. |
| **State match** | The answer's state differs from `expect_state`. |
| **Retrieval health** | The question overflowed the context budget, or found nothing. |
| **Stale evidence** | The answer rests on a page with `status: stale`. A warning, not a failure — but it explains a drift before the content does. |

### Pass 2 — semantic

Judgment. Slower, less stable, and the only way to catch the interesting failures. Run it when
asked (`--semantic`), or when pass 1 is clean but the answers still look wrong.

| Check | Looks for |
|---|---|
| **Unsupported claims** | Statements in the answer that the cited sources do not actually support. |
| **Overstated certainty** | An inference presented as a fact, or a contradicted question answered with one confident side. |
| **Evidence sufficiency** | Whether the cited evidence would convince a reader, or is merely present. |
| **Contradiction awareness** | A known contradiction in the pages that the answer silently resolved. |
| **Consistency** | Two cases that should agree, and do not. |

Say which pass produced each finding. A structural failure is a fact about the base; a semantic
one is your reading of it, and the user should be able to tell them apart at a glance.

## Output

```
docs/scenarios/questions.yaml — 12 cases

PASS  10
FAIL   2

FAIL  session-ttl                                          [structural]
      expect_sources: docs/core-sources/260710-ops-runbook.pdf not cited
      The answer cited only 260415-auth-spec.pdf (§4.2, "30 minutes"),
      which forbid_claims rules out.
      Likely cause: docs/topics/session-management.md is status: stale;
      its source changed 2026-07-10, the page was updated 2026-04-20.
      Fix: /scan-docs docs/core-sources/260710-ops-runbook.pdf

FAIL  revocation-policy                                     [structural]
      expect_state: unknown, got: known
      The answer asserted a 90-day rotation policy citing
      docs/topics/session-management.md, which carries no citation for it.
      This is an unsupported claim reaching a user as a fact.
      Fix: cite it or cut it — see lint check 6.

WARN  auth-flow                                             [structural]
      Answer rests on docs/topics/authentication.md (status: stale).
```

Then append to `docs/CHANGELOG.md`:

```markdown
## 2026-08-30 — test
- Ran: 12 cases from docs/scenarios/questions.yaml (structural)
- Passed: 10
- Failed: session-ttl (source not cited, page stale), revocation-policy (unsupported claim)
- Warned: auth-flow rests on a stale page
```

Report failures, then stop. **Do not fix the knowledge base to make a scenario pass.** Editing a
page so its question scores green is how an scenario suite becomes decoration. Say what failed and
what would fix it — `/scan-docs` on a stale source, a ruling on a contradiction, a
citation added — and let the user decide.

If a case fails because the *expectation* is wrong rather than the base, say so. Sources change
and yesterday's required fact becomes today's superseded one. But `docs/scenarios/` is the owner's:
propose the edit, never make it.

## Rules

- Never edit `docs/scenarios/questions.yaml`.
- Never edit pages during a test run. Findings go in the report; fixes belong to `scan-docs`,
  `lint-docs`, or the owner.
- Never look at a case's expectations before answering its question.
- Never report a semantic finding as a structural one.
- Never claim a case passed on a technicality — an answer that contains `"24 hours"` inside a
  sentence saying the opposite has not passed.
