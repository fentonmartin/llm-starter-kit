# DOCS.md — the knowledge governance contract

This file is the contract between you and the agent. The agent reads it before every
`scan-docs`, `ask-docs`, `lint-docs`, and `test-docs` run, and `init-docs` writes it. Edit it to
change how the knowledge base behaves; do not edit the agent's skills for project-specific rules.

**It is in two parts. [Part 1 — The contract](#part-1--the-contract) is the rules, and it is
complete on its own: read it in full, every run. [Part 2 — Reference](#part-2--reference) is
worked examples, workflows, and rationale — read it when you need it.**

Nothing in Part 2 adds a rule. If the two ever disagree, Part 1 wins and Part 2 is a bug.

---

# Part 1 — The contract

## Layers

Everything lives under `docs/`, but the layers are strictly separate.

| Layer | Path | Who writes it |
|---|---|---|
| Core sources | `docs/core-sources/` | **Humans only.** Immutable. The agent reads it, never edits, renames, or deletes. |
| Pages | `docs/summaries/`, `docs/topics/`, `docs/entities/` | **Agent only.** Every file derives from a source or an explicit instruction. |
| Scenarios | `docs/scenarios/` | **Humans.** Questions the knowledge base must answer correctly. |
| Bookkeeping | `docs/README.md`, `docs/CHANGELOG.md` | **Agent only.** Index and log. |
| Guidance | `docs/DOCS.md` | **Humans.** This file. |

`docs/core-sources/` is exempt from every page rule below — front matter, slugs, wikilinks, page
types. Only the `YYMMDD` file-name rule applies there. Checks that sweep `docs/` skip it.

**If a fact is not traceable to a source or to a human instruction, it does not belong in a page.**

### Why "core"

The prefix names a **role**, not a file type. `docs/core-sources/` is the root of the provenance
chain: every page in the knowledge base is reproducible from it, and nothing else is. Deleting a
page loses work; deleting from here loses the truth.

It also removes a genuine ambiguity. In a knowledge base that documents a codebase there are two
kinds of source — the curated, immutable material here, and `src/**`, which the codebase preset
reads in place and which changes every commit. Calling both "sources" blurred the immutability
rule precisely where it mattered most. And "source" is already overloaded across the `sources:`
field, the citation format, and source code; the folder now has a name that means one thing.

The practical payoff is that the hard rule reads as a boundary rather than a category:
*never write to `core-sources/`*.

## Authority

```
Human decision recorded in a page   ← highest
Source material in docs/core-sources/
Agent-generated page content
The agent's background knowledge    ← none; must be labelled if used at all
```

- **The agent discovers, summarizes, classifies, links, flags, and proposes. It does not decide.**
- **`confidence` is not authority.** It describes how well evidence supports a claim.
- **A summary is not a source.** Cite the source a summary came from, never the summary.
- An agent that believes a human decision is wrong records an open question. It does not edit
  the decision.

## Page types

| Type | Path | One page per | Purpose |
|---|---|---|---|
| Summary | `docs/summaries/<source-slug>.md` | source file | Faithful condensation of one source. The only page allowed to paraphrase at length. |
| Topic | `docs/topics/<slug>.md` | idea, question, theme | Synthesis across sources. Where contradictions surface. |
| Entity | `docs/entities/<slug>.md` | person, org, product, place, dataset | Stable facts about one thing, plus where it appears. |

## Front matter

**Front matter identifies and governs the document. The knowledge lives in the Markdown body,
never in YAML.**

Six required fields:

```yaml
---
type: summary | topic | entity
title: Human readable title
status: draft | active | stale | superseded | deprecated | archived
claim_type: fact | decision | assumption | hypothesis | open-question | contradiction | instruction
updated: YYYY-MM-DD
sources: [docs/core-sources/foo.pdf]     # summaries: exactly one. topics/entities: all that back it.
---
```

Four optional fields, added only when they carry information:

```yaml
created: YYYY-MM-DD
confidence: high | medium | low                    # evidential support. Not authority.
superseded_by: docs/topics/authentication-v2.md    # required when status is superseded
sensitivity: public | internal | confidential      # a label for humans, not enforcement
```

- **Flat scalars and flat lists only.** No nested maps, no lists of maps, no anchors. Structure
  goes in a Markdown section.
- `sources:` holds plain paths. Section and line detail belongs in the citation.
- Unknown extra fields are tolerated and ignored. Missing required fields are a lint finding.
- On a summary, `claim_type` defaults to `fact` and may be omitted. On topics and entities it is
  required.

**Backward compatibility:** pages predating `status` and `claim_type` remain valid. Missing
either is a **WARNING**, not an ERROR. Defaults: `status: active`; `claim_type: fact` on a
summary, `open-question` on a topic or entity that cannot be classified. **Never default to
`decision`.** No migration is required.

## Claim types

| `claim_type` | Means | Established by |
|---|---|---|
| `fact` | The sources support this as true of the world. | Evidence |
| `decision` | A human chose this for this project. | **Human only** |
| `assumption` | Taken as true to make progress; not evidenced. | Either, if labelled |
| `hypothesis` | Proposed, testable, untested. | Either, if labelled |
| `open-question` | Nobody knows yet. Evidence is missing. | Either |
| `contradiction` | Sources disagree and it is unresolved. | Either |
| `instruction` | A rule for how work is done here. | **Human only** |

*"Authentication uses Redis"*, *"the team chose Redis"*, and *"we believe Redis is required"* are
three different claims. Storing them identically is how a knowledge base starts lying.

**An agent may never set `claim_type: decision` or `instruction`.** One that thinks a decision
was made writes `open-question` and asks.

Mark individual claims in the body with callouts: `[!check]` decision, `[!note]` assumption,
`[!abstract]` hypothesis, `[!question]` open question, `[!warning]` contradiction,
`[!important]` instruction. Unmarked prose is a `fact` and must be cited.
→ [Examples](#marking-claims-inside-a-page)

## Lifecycle

| `status` | Means | Who sets it |
|---|---|---|
| `draft` | Being written; incomplete. | Agent or human |
| `active` | Current and believed accurate. | Agent or human |
| `stale` | A source it depends on changed after `updated`. Unverified, not wrong. | Agent |
| `superseded` | Replaced by another page. **Requires `superseded_by`.** | **Human only** |
| `deprecated` | Describes something no longer in use. Kept for history. | **Human only** |
| `archived` | Retired from active retrieval. Still readable. | **Human only** |

- **An agent may set only `draft`, `active`, and `stale`.**
- A `superseded` page keeps its content. It is not emptied and not deleted.
- `ask-docs` retrieves `active` and `stale` pages, labels the stale ones, and excludes
  `superseded`, `deprecated`, and `archived` unless asked for history.

**Freshness:** a page is `stale` when a source it cites changed after its `updated` date —
compared by modification time or last commit date, not by a dependency graph. A cited line range
that no longer exists is a *broken citation*, not staleness. Re-scanning clears it.

## Citations and provenance

Claims in topic and entity pages carry a source:

```markdown
... throughput dropped 40% ([[docs/core-sources/260415-bench|260415-bench]], p.4).
```

Uncited claims are allowed only for definitions and connective prose.

**Inside a table cell, escape the wikilink's pipe as `\|`** — an unescaped one splits the cell and
silently breaks the table:

```markdown
| `SESSION_TTL` | `86400` | [[docs/core-sources/260710-ops-runbook\|260710-ops-runbook]] |
```

**A citation is a claim that a document was read.** Cite only sources and pages actually opened
in this run.

Use the most precise anchor the format supports:

| Format | Anchor |
|---|---|
| PDF | page — `p.4` |
| Markdown, spec, doc | section — `§4.2` |
| Source code | path and line range — `src/auth/session.php:112-140` |
| Transcript, video | timestamp — `14:20` |
| Web page | section plus capture date |
| Anything else | the file alone |

**Never invent an anchor.** An unseen page number, an unverified line range, and a guessed commit
hash are fabricated provenance. **Falling back to the bare file name is always correct.**

## Contradictions

**Contradictions are never silently resolved.** Record both sides on the topic page that owns the
question, and set `claim_type: contradiction` while it stands:

```markdown
> [!warning] Contradiction
> [[docs/core-sources/260415-auth-spec|260415-auth-spec]] (§4.2) gives session TTL as 30 minutes.
> [[docs/core-sources/260710-ops-runbook|260710-ops-runbook]] (p.2) gives it as 24 hours.
> Unresolved as of 2026-08-30.
```

The agent must not close a contradiction by reasoning that one source looks newer, more official,
or more detailed. Recency decides only when a project rule below says so. Absent such a rule, it
stays open until a human rules. → [Human review](#human-review)

## Retrieval and context

`ask-docs` must never load the whole knowledge base.

**Selection:** match the question against `docs/README.md` titles and descriptions, read those
pages, then follow their wikilinks one hop out — two if the question is broad. Then stop. Drop to
`docs/core-sources/` only when the pages are thin, a verbatim quote is needed, or a page is suspected
stale, and say so. Scope may be constrained with `--topic`, `--entity`, `--source`.

**Budget:** roughly half the context window, estimated before reading at ~4 characters per token.

**Over budget is a graceful failure, not a truncation.** Say what was found, why it does not fit,
and what would narrow it. Dropping sources silently and answering as though the picture were
complete is the worst available outcome — it is indistinguishable from a correct answer.

**Answer states:** every answer separates **Known** (cited evidence), **Inferred** (agent
reasoning, always labelled), **Contradicted** (both sides shown), and **Unknown** (say so, name
the gap). *"I don't know based on the available knowledge"* is a correct answer. An unsupported
inference presented as fact is a defect regardless of whether it is right.

## Security

**The agent never reads, ingests, summarizes, or quotes from:**

```
.env, .env.*                       secrets/**, credentials/**, private/**
*.pem, *.key, *.p12, *.pfx         **/*.secret, **/*secrets*.y*ml
id_rsa, id_dsa, id_ecdsa, id_ed25519    .aws/**, .ssh/**, .netrc
```

Add project paths under [Out of scope](#out-of-scope). If a source turns out to contain
credentials, **stop**: no summary, no quotes, tell the user. Do not redact and continue — the
secret is already in context, and a partial summary normalizes the leak.

These exclusions are instructions, not enforcement. Nothing in a Markdown file can stop an agent
from reading a path.

## Git conflicts

A merge conflict inside `docs/` is a **semantic** conflict. Git knows a human wrote X and an agent
wrote Y; it does not know which is true.

**An agent never auto-resolves a conflict in `docs/topics/`, `docs/entities/`, or
`docs/summaries/`.** Surface both sides and stop. Two exceptions, because neither carries meaning:
`docs/CHANGELOG.md` (keep both, date order) and `docs/README.md` (keep the union, re-sort).

If a conflict is genuinely two claims about the same thing, the resolution is a contradiction
callout, not a choice.

## Naming and dates

- Slugs are lowercase kebab-case ASCII. `docs/entities/vector-database.md`.
- **Any date in a file name is `YYMMDD`** — no separators, no exceptions. Only genuinely dated
  files get one; evergreen topic and entity pages get none. A summary inherits its source's
  prefix verbatim.
- **Dates inside files are `YYYY-MM-DD`** — front matter, changelog headings, prose.

## Bookkeeping

- `docs/README.md` — the index. Every page listed exactly once, grouped by type, one line each.
  Updated in the same pass that creates or removes a page.
- `docs/CHANGELOG.md` — append-only, newest first. One entry per run. Never rewrite an entry.

## Style

- Present tense, plain language, no hedging filler.
- Short pages that link out beat long pages that repeat.
- If two pages overlap by more than roughly half, merge them and leave a redirect line.

## Out of scope

Paths the agent never reads, beyond the security exclusions. Replace with this project's own:

```
docs/core-sources/archive/**
```

## Project-specific rules

These are the defaults shipped with the starter kit. `/init-docs` rewrites this section from an
interview; **if it still reads like the text below, that interview never happened.**

- Every topic page ends with an **Open questions** section. An empty one is a signal, not a
  defect: nothing is unresolved yet.
- Every entity page opens with a one-sentence definition, so a reader following a wikilink
  mid-sentence gets oriented in one line.
- Ignore `docs/core-sources/archive/**` unless the user names a file in it explicitly.
- Figures carry their unit and as-of date: `$4.2M (FY2025)`, `340ms p99 (2026-04-15)`. A bare
  number in a topic page is a lint finding.
- Cap summary pages at roughly 600 words. A source needing more should produce topic pages, not a
  longer summary.

**End of the contract.** Everything below is elaboration.

---

# Part 2 — Reference

Worked examples, workflows, and rationale. No new rules.

## Marking claims inside a page

A page has one dominant `claim_type` in front matter. Individual claims are marked with Obsidian
callouts, which render in Obsidian and grep cleanly:

```markdown
Sessions expire after 30 minutes ([[docs/core-sources/260415-auth-spec|260415-auth-spec]], §4.2).

> [!check] Decision
> Session storage moved to Redis on 2026-03-11. Rejected: in-process cache (loses state on
> deploy), Postgres (write amplification under load). — approved by @fenton

> [!note] Assumption
> Traffic stays under 5k concurrent sessions. Not measured; carried over from the 2025 design.

> [!question] Open question
> Nothing in the sources describes session revocation on password change.
```

## Human review

The path out of `contradiction` and `open-question`. **The evidence is preserved, never
overwritten.**

1. A human reads both sides and the underlying sources.
2. They rule.
3. The page is rewritten to the shape below — ruling on top, history intact underneath.
4. `claim_type` becomes `decision` (a project choice) or `fact` (the evidence settled it).
   `status` becomes `active`.
5. The change is committed, so the ruling has an author and a date in git.

```markdown
---
type: topic
title: Session TTL
status: active
claim_type: decision
updated: 2026-08-30
sources: [docs/core-sources/260415-auth-spec.pdf, docs/core-sources/260710-ops-runbook.pdf]
---

## Decision

Session TTL is 24 hours. — @fenton, 2026-08-30

## Rationale

The spec and the runbook disagreed. The runbook reflects what production has run since the
March migration; the spec was not updated. The runbook is authoritative for operational values.

## Evidence

- [[docs/core-sources/260415-auth-spec|260415-auth-spec]] (§4.2) — 30 minutes. Superseded by the
  March 2026 migration.
- [[docs/core-sources/260710-ops-runbook|260710-ops-runbook]] (p.2) — 24 hours. Matches production.
```

What makes this auditable is that the disagreement is still on the page after it is settled.
Deleting the losing side destroys the reason the decision exists.

## Retrieval, in full

```
question → select candidates → read candidates → estimate budget
                                                        │
                                            ┌───────────┴───────────┐
                                       within budget          over budget
                                            │                      │
                                          answer          ask for narrower scope
                                            │
                                        citations
```

The selection step is deliberately replaceable — swapping how candidates are found must not
change anything about the knowledge format.

A graceful overflow looks like:

```
The evidence for this question exceeds the context budget: 34 pages across 6 topics
(~180k tokens). Narrow the scope and I can answer precisely:
  --topic authentication     (7 pages)
  --topic rate-limiting      (5 pages)
  --source 260415-auth-spec.pdf
```

## Provenance for versioned sources

A claim depending on a specific revision records it:

```markdown
Sessions are stored in Redis (`src/auth/session.php:112-140`, as of `a3f9c21`).
```

Use a commit hash only when it can actually be read from git. Do not fabricate hashes.

## Links

Obsidian wikilinks: `[[vector-database]]`, or `[[vector-database|vector DBs]]` to relabel. Link on
first mention of any concept that has (or should have) its own page. **A link to a page that does
not exist yet is encouraged** — it is a to-do, and `lint-docs` collects them as gaps.

This is Obsidian Flavored Markdown, so `docs/` opens directly as a vault — graph view, backlinks,
and unresolved links all work without conversion.

> To use plain relative Markdown links instead, change this section and tell the agent to
> migrate; the skills read this file for link style rather than assuming one.

Note that a summary carries its source's file name, so the two share a basename. That is why
source citations use the full-path form `[[docs/core-sources/foo|foo]]` — a bare `[[foo]]` would be
ambiguous.

## Preset: documenting a codebase

If this knowledge base documents the project it lives in — the most common use — replace the page
types in Part 1 with these, and delete this section once you have.

| Type | Path | One page per | Purpose |
|---|---|---|---|
| Summary | `docs/summaries/<module>.md` | module or package | What it does, its public surface, what it depends on, what depends on it. |
| Topic | `docs/topics/<slug>.md` | decision, flow, or invariant | Why it is built this way. How a request moves through. What must stay true. |
| Entity | `docs/entities/<slug>.md` | service, table, endpoint, job, env var, external API | Stable facts, plus every place it is referenced. |

Then set the source layer. Code is read in place rather than copied:

```markdown
- Treat `src/**` as sources, in addition to `docs/core-sources/`.
- Out of scope: `node_modules/**`, `dist/**`, `**/*.generated.*`, `vendor/**`, test fixtures.
```

Rules worth adding for code:

- A summary page names the file paths it covers, so `lint-docs` can tell when the code moved.
- Never document intended behavior as actual behavior. If code and comment disagree, that is a
  contradiction — record both.
- Architectural decisions get a topic page with `claim_type: decision` and the alternatives that
  were rejected. A decision without its discarded options is a description, not documentation.
- Environment variables and endpoints are entities, never bullet lists buried in a summary. They
  are referenced from too many places to live inside one page.
