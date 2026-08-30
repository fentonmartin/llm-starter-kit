# Example project

A knowledge base small enough to read in five minutes, built from two documents that disagree.

The disagreement is the point. Two sources, one specification and one runbook, give different
values for the same setting. A retrieval system that returns the more relevant chunk answers
this question confidently and wrongly. This kit records both and says a human has not ruled.

## What is here

```
docs/
  DOCS.md                        the schema, with three sections customized
  README.md                      the index
  CHANGELOG.md                   what each run did
  scenarios/questions.yaml           three questions this base must answer correctly
  core-sources/
    260415-auth-spec.md          a specification. Says the TTL is 30 minutes.
    260710-ops-runbook.md        a runbook. Says production runs 24 hours.
  summaries/
    260415-auth-spec.md          one page per source
    260710-ops-runbook.md
  topics/
    session-management.md        the synthesis. Holds the open contradiction.
  entities/
    redis.md                     the store both sources describe
```

Two sources in, four pages out. That ratio is normal — the pages exist to be read by someone
who has not read the sources.

## The loop

```
sources/  ──scan──▶  summaries/ topics/ entities/  ──ask──▶  answer + citations
                              ▲                                      │
                              └──────  lint  ◀──  test  ◀────────────┘
```

Run it yourself from this directory:

| Step | Command | What it does here |
|---|---|---|
| 1 | `/init-docs` | Already run. It produced `DOCS.md` and the folder structure. |
| 2 | `/scan-docs` | Already run twice, once per source. See `docs/CHANGELOG.md`. |
| 3 | `/ask-docs "What is the session TTL?"` | Should return **both** values and say the sources disagree. |
| 4 | `/lint-docs` | Should report the open contradiction as INFO, not as an error. |
| 5 | `/test-docs` | Runs the three cases in `docs/scenarios/questions.yaml`. |

## What to look at, and why

**The contradiction** — [`docs/topics/session-management.md`](docs/topics/session-management.md),
under *Expiry*. Both values are on the page with their anchors. The runbook even explains why
the TTL was raised, which makes it tempting to close. The page does not close it, because
nothing in `DOCS.md` says a runbook outranks a spec. That call belongs to a human, and
`docs/DOCS.md` shows the shape the resolution would take — the ruling on top, both sources
preserved underneath.

**Claim typing** — [`docs/summaries/260415-auth-spec.md`](docs/summaries/260415-auth-spec.md).
The spec records that JWTs were rejected. That is a decision, but the page's `claim_type` is
`fact`, and the decision sits in a `[!check]` callout citing §5. A source describing a decision
is evidence one was made, not authority that it still holds. An agent may record it; only a
human may promote it.

**The unknown** — the *Open questions* section of the same topic page, and the
`session-revocation` case in `docs/scenarios/questions.yaml`. Neither source says who owns session
revocation or when it will be fixed. The scenario asserts that the base keeps saying so. It is the
most valuable case in the file: a system that quietly invents a plausible policy here passes
every other check.

**Provenance** — every claim carries the most precise anchor its source supports: `§4.2` for the
spec's numbered sections, `SESSION_TTL` and section names for the runbook's tables. Nothing
carries a page number, because Markdown sources do not have pages.

**Lifecycle** — every page is `status: active` today. Edit `docs/core-sources/260710-ops-runbook.md`
and run `/lint-docs`: its summary and both pages citing it become `stale`, because their
`updated` now predates the source. Staleness means unverified, not wrong.

## Try breaking it

Three edits, each of which should produce a specific finding rather than a wrong answer:

1. **Change the runbook's `SESSION_TTL` to `3600`.** Re-scan. The contradiction should change
   its numbers, not disappear.
2. **Delete the closing `---` from any page's front matter.** Run `/lint-docs`. It should report
   one ERROR with the corrected block, keep checking the other files, and not ingest the half of
   the metadata that still looked like YAML.
3. **Ask something no source covers** — `/ask-docs "How many concurrent sessions can Redis
   hold?"` The answer should be that the knowledge base does not cover it, naming the gap. Any
   number in that answer is a bug.
