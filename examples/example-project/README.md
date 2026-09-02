# Example project

A knowledge base small enough to read in five minutes, built from two documents that disagree.

> **What this shows, and what it doesn't.** Deliberately the default configuration: sources in
> `docs/core-sources/`, flat page layers, one index. That's what `/init-docs` produces for a fresh
> start, and it's the shape most bases keep. The options it doesn't exercise — a top-level
> `sources/`, pointing at material you already file elsewhere, subfolder grouping past ~30 pages
> per layer — are all one-line declarations away and documented in
> [`docs/DOCS.md`](../../docs/DOCS.md). Nothing here would change if you used them.

The disagreement is the point. Two sources, one specification and one runbook, give different
values for the same setting. A retrieval system that returns the more relevant chunk answers
this question confidently and wrongly. This kit records both and says nobody has ruled.

## What is here

```
docs/
  DOCS.md                        the schema, with three sections customized
  INDEX.md                       the index
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
| 5 | `/test-docs` | Runs the three scenarios in `docs/scenarios/questions.yaml`. |

**[`TRANSCRIPT.md`](TRANSCRIPT.md) shows what steps 3, 4 and 5 actually output** against this
base — the contradicted answer, a clean lint with the contradiction held open as INFO, and three
passing scenarios. It ends with four edits that each make something fail on purpose.

## What to look at, and why

**The contradiction** — [`docs/topics/session-management.md`](docs/topics/session-management.md),
under *Expiry*. Both values are on the page with their anchors. The runbook even explains why
the TTL was raised, which makes it tempting to close. The page does not close it, because
nothing in `DOCS.md` says a runbook outranks a spec. That call is the owner's, and
`docs/DOCS.md` shows the shape the resolution would take — the ruling on top, both sources
preserved underneath.

**Claim typing** — [`docs/summaries/260415-auth-spec.md`](docs/summaries/260415-auth-spec.md).
The spec records that JWTs were rejected. That is a decision, but the page's `claim_type` is
`fact`, and the decision sits in a `[!check]` callout citing §5. A source describing a decision
is evidence one was made, not authority that it still holds. An agent may record it; only a
the owner may promote it.

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

Two edits worth making before you read anything else, because a system that only ever succeeds
teaches you nothing about what it does under stress:

1. **Change the runbook's `SESSION_TTL` to `3600`.** Re-scan. The contradiction should change
   its numbers, not disappear.
2. **Ask something no source covers** — `/ask-docs "How many concurrent sessions can Redis
   hold?"` The answer should be that the knowledge base does not cover it, naming the gap. Any
   number in that answer is a bug.

[`TRANSCRIPT.md`](TRANSCRIPT.md) has four more, each aimed at a specific check: a broken
citation, malformed front matter, a stale page, and a scenario whose *expectation* is what's
wrong rather than the knowledge base.
