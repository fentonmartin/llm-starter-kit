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

**Every knowledge base built with this kit has the same shape: one root folder, five layers,
three page types.** It does not vary by what you are documenting. A codebase, a reading pile, a
runbook set, and a library of documents all get the identical structure — what changes is what a
page is *about*, not where it lives. That is deliberate: the checks in `lint-docs`, the retrieval
order in `ask-docs`, and every rule below are written against these paths, and one shape means
one contract to learn, one set of habits, and a base you can move between projects.

**This knowledge base is rooted at `docs/`.** Every path in this file and in the agent's skills
is written with that prefix. If the sentence in bold above names a folder other than `docs/`,
read every `docs/…` path in this file *and in every skill* as that folder instead: `docs/topics/`
means `<root>/topics/`, and a check that skips `docs/core-sources/` skips `<root>/core-sources/`.
Nothing else changes.
The root is the only part of the layout that is yours to choose, and `docs/` is the default and
the right answer for almost every project — see *Choosing a different root* in Part 2 for the two
cases that need another.

Everything lives under the root, but the layers are strictly separate.

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

**The agent never adds, renames, or removes a layer, and never adds a page type.** This project's
own vocabulary belongs in *Page types* below, not in folder names. Changing the shape is a human
decision, recorded in `docs/CHANGELOG.md` — Part 2's *Changing the shape later* says what each
kind of change costs.

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
status: draft | active | stale | superseded
claim_type: fact | decision | assumption | open-question | contradiction
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
| `assumption` | Taken as true to make progress; not evidenced. Covers untested proposals too. | Either, if labelled |
| `open-question` | Nobody knows yet. Evidence is missing. | Either |
| `contradiction` | Sources disagree and it is unresolved. | Either |

*"Authentication uses Redis"*, *"the team chose Redis"*, and *"we believe Redis is required"* are
three different claims. Storing them identically is how a knowledge base starts lying.

**An agent may never set `claim_type: decision`.** One that thinks a decision was made writes
`open-question` and asks. That is the only claim value reserved to humans.

Mark individual claims in the body with callouts: `[!check]` decision, `[!note]` assumption,
`[!question]` open question, `[!warning]` contradiction. Unmarked prose is a `fact` and must be
cited. → [Examples](#marking-claims-inside-a-page)

Five values, deliberately. A sixth distinction you cannot apply consistently at 2am is worse than
no distinction, because it makes the field unreliable rather than merely coarse. Rules for how
work is done here belong in this file, not on a page — a page that carries a rule is a second,
unenforced home for it.

## Lifecycle

| `status` | Means | Who sets it |
|---|---|---|
| `draft` | Being written; incomplete. | Agent or human |
| `active` | Current and believed accurate. | Agent or human |
| `stale` | A source it depends on changed after `updated`. Unverified, not wrong. | Agent |
| `superseded` | Replaced by another page. **Requires `superseded_by`.** | **Human only** |

- **An agent may set only `draft`, `active`, and `stale`.** `superseded` is the one status
  reserved to humans, because retiring a page is a judgment about the project.
- A `superseded` page keeps its content. It is not emptied and not deleted.
- `ask-docs` retrieves `active` and `stale` pages, labels the stale ones, and excludes
  `superseded` unless asked for history.

Four values, deliberately. *"No longer in use"* is a fact about the world and belongs in the page
body with a citation; retiring a page from view is a filesystem decision. Neither needs its own
lifecycle state, and both were previously easy to confuse with `superseded`.

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

## Presets

Four starting shapes. `/init-docs` picks one from what you say the knowledge base is for, then
rewrites the page types, out-of-scope paths, and project rules in Part 1 to match. They differ
only in those three sections — **the structure and the contract are identical for all of them.**
A preset is not a different layout; it is a different vocabulary poured into the same one.

Pick the one whose nouns match yours. If none do, take the closest and rename its page types.

### Preset: documenting a codebase

If this knowledge base documents the project it lives in — the most common use — replace the page
types in Part 1 with these.

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

### Preset: outside research

For a reading pile — papers, articles, vendor documentation, transcripts, competitor material.
The knowledge base is *about* something the project does not own.

| Type | Path | One page per | Purpose |
|---|---|---|---|
| Summary | `docs/summaries/<source-slug>.md` | paper, article, talk, thread | What it argues, what evidence it gives, where it is weak. |
| Topic | `docs/topics/<slug>.md` | question or debate | What the field believes, who disagrees, and what is still unsettled. |
| Entity | `docs/entities/<slug>.md` | person, lab, company, model, dataset, benchmark | Stable facts, and every place it appears. |

Rules worth adding for outside material:

- **A claim carries whose claim it is.** *"Latency drops 40%"* is not a fact; *"the vendor
  reports latency drops 40%"* is. Attribute in the sentence, not only in the citation.
- **Record the method before the result.** A number without a benchmark, sample size, or date is
  not usable evidence, and a topic page that repeats it is laundering it.
- **A vendor is a source, not a referee.** Marketing material is evidence of what a vendor
  claims. Never let it settle a contradiction against an independent source.
- Disagreement between researchers is the normal state, not a defect. Expect topic pages to
  carry standing contradictions for a long time.
- Capture the date you read a web source. Pages change silently and the citation is all you will
  have.

### Preset: a document library

For a project whose documents *are* the project — a book or spec set being written, a policy
library, a pile of notes used as the knowledge base. There is little or no code, and the material
is the user's own rather than something read from outside.

| Type | Path | One page per | Purpose |
|---|---|---|---|
| Summary | `docs/summaries/<source-slug>.md` | document, chapter, note file | What this document says, condensed, with its own structure preserved. |
| Topic | `docs/topics/<slug>.md` | subject, question, theme | The synthesis layer, and where the real value is: what the library says about one subject, across every document that touches it. |
| Entity | `docs/entities/<slug>.md` | person, org, product, term, place | Stable facts about one thing, plus every document that mentions it. |

Rules worth adding for a document library:

- **Topics carry the weight.** A library is navigated by subject, not by file name. If a scan
  produces summaries and almost no topic pages, the base is a filing cabinet, not a knowledge
  base — that is the failure mode to watch for here.
- **The documents are authoritative, not third-party.** Unlike outside research, a claim does not
  need attributing to whoever made it; the freshness and authority rules apply to the documents
  directly. When two of your own documents disagree, it is a real contradiction needing a human
  ruling, not a difference of opinion between sources.
- **A term with a specific local meaning is an entity, not a glossary line.** Libraries drift on
  vocabulary before they drift on facts.
- **Undated documents are the norm here.** The `YYMMDD` prefix applies to anything with a
  meaningful date — a dated memo, a meeting note, a revision. Evergreen documents carry no
  prefix, and that is not a lint finding.
- Out of scope: build output of whatever renders these documents — `site/**`, `_build/**`,
  `.obsidian/**`, `*.docx` lock files.

### Preset: operations and runbooks

For how a running system is actually operated — incidents, procedures, configuration, on-call
knowledge. The distinguishing risk is that a page can be *out of date and confident* while
someone is following it at 3am.

| Type | Path | One page per | Purpose |
|---|---|---|---|
| Summary | `docs/summaries/<source-slug>.md` | incident report, runbook, postmortem, audit | What happened or what the procedure says, verbatim enough to act on. |
| Topic | `docs/topics/<slug>.md` | procedure, failure mode, or invariant | How to do it, what breaks, what must stay true. |
| Entity | `docs/entities/<slug>.md` | service, queue, alert, dashboard, config key, on-call rota | Stable facts, plus every place it is referenced. |

Rules worth adding for operations:

- **Every configuration value carries its unit and its as-of date.** `86400 seconds (24 hours,
  since 2026-06-02)`, never a bare number. This is the rule that catches drift first.
- **Never state a designed value as a running value.** A specification says what should be true;
  a runbook says what is. When they differ that is a contradiction, and it is the most common one
  this preset produces.
- **A procedure page names its blast radius** — what this action takes down, and who notices.
- **Staleness is not cosmetic here.** A `stale` procedure page should say so in its first line,
  not only in front matter, because the person reading it at 3am is scrolling, not auditing.
- Incidents are dated sources and keep their `YYMMDD` prefix. Procedures are evergreen and carry
  no date.

### Writing your own preset

If none of the four fit, write a fifth. A preset may set exactly three things, and they are the
same three `/init-docs` rewrites from an interview:

1. the **page types** table — the nouns, one row per page type
2. the **out of scope** paths
3. the **project-specific rules** — four to six, each one checkable

It may set nothing else. The layers, the three page types themselves, front matter, claim types,
authority, lifecycle, provenance, links, citations, contradictions, retrieval, and the security
rules are the contract, and they are what make a base lintable and testable. A preset that wants
a fourth page type or a renamed folder is not a preset — it is a fork, and `lint-docs` and
`test-docs` will not understand it.

Write it as a `### Preset: <name>` section here, in the same shape as the four above. `/init-docs`
deletes all of Part 2 including the preset it used, so a preset lives in the kit, never in a
project's own contract.

## Choosing a different root

`docs/` is the default and the right answer for almost every project. Two cases need another:

| Situation | Root | Why |
|---|---|---|
| The project's `docs/` is a published site (`mkdocs.yml`, `docusaurus.config.js`, `book.toml`) | `docs/kb/` | Rearranging a folder a generator builds from breaks the build. The published docs become sources. |
| The project needs two bases that must not mix | `docs/` and a second root, e.g. `research/` | See below. |

Whatever you choose, say it in the root-binding line at the top of Part 1. That line is the only
place the root is declared, and every skill reads this file before it reads anything else.

### Two knowledge bases in one project

Legitimate when a project has two bodies of knowledge with different authority — a codebase base
whose facts come from the code, and a research base whose facts come from other people's papers.
Mixing them is what the split prevents: a vendor claim must never settle a question about your own
system.

- Each base is **completely self-contained**: its own root, its own `DOCS.md` with its own
  root-binding line and its own preset, its own five folders, its own `README.md` index and
  `CHANGELOG.md`.
- Run `/init-docs` once per base, telling it which root. The interview runs again — the second
  base gets its own page types and its own rules, which is the entire point.
- Every command takes the root as an argument: `/scan-docs research/`, `/lint-docs research/`,
  `/ask-docs --root research/ what does the field think about X?`. With no argument, the root is
  `docs/`.
- **A run never spans two bases.** An answer cites pages from one base. Do not wikilink across
  roots — a link out of a base is a citation to material the other base's contract does not
  govern, and lint will read it as a gap forever.
- If you find yourself asking questions that need both bases, they are one base. Merge them.

## Changing the shape later

A knowledge base that grows will need adjusting, and the layout is designed so that most
adjustments are free. Sorted by cost:

**Free — change `DOCS.md` and carry on. No migration, no lint run needed.**

- Rename the nouns in the page types table. "one page per module" → "one page per service".
- Add, remove, or reword project-specific rules.
- Add or remove out-of-scope paths.

New pages follow the new wording; existing pages are still valid, because none of these change
the schema. Note the change in `docs/CHANGELOG.md` so the next reader knows when the rule started.

**Cheap — change `DOCS.md`, then run `/lint-docs` and commit the diff.**

- **Switching preset** (the base turned out to be about something else). Replace the three
  sections from the new preset, then lint: existing pages keep their front matter and their
  provenance, and get reclassified as they are next touched. Do not bulk-rewrite pages to match a
  new preset — the pages were true when written, and the rules govern what happens next.
- **Reorganizing within a layer** — splitting one overgrown topic page into three, promoting a
  section to its own entity page. Leave a redirect line in the page you emptied, and lint will
  catch the inbound wikilinks that need repointing.
- **Changing link style** (wikilinks ↔ relative Markdown). Change the *Links* section, then tell
  the agent to migrate; the skills read this file for link style rather than assuming one.

**Real work — plan it, do it in one commit, lint immediately after.**

- **Moving the root** (`docs/` → `docs/kb/`, say):
  1. Move the folder whole; never lift one layer out from under the others. A move *into* a
     subfolder of itself needs a staging step, because git will not do it in one:

     ```bash
     mkdir kb-tmp && git mv docs/* kb-tmp/
     mkdir -p docs && git mv kb-tmp docs/kb
     ```

     Any other move is the one-liner you would expect: `git mv docs wiki`.
  2. Change the root-binding line in Part 1 to the new root. This one line is the entire
     re-pointing of the contract and the skills.
  3. Run `/lint-docs`. Check 15 rewrites every `sources:` value and every citation that still
     points at the old root, and reports the ones it cannot.
  4. Grep the rest of the repo for the old path — `CLAUDE.md`, `AGENTS.md`, CI config, and README
     links do not fix themselves.
- **Splitting one base into two.** Decide the boundary first, by authority — which facts come
  from where — never by volume or by folder size. Then `/init-docs` the second root, `git mv` the
  pages that belong to it, and lint both. Cross-base wikilinks broken by the split must be
  resolved by duplicating the fact with its own citation in the new base, not by linking across.
- **Merging two bases into one.** `git mv` the second base's four layers into the first, port its
  project-specific rules into the survivor's Part 1 (they were written for a reason), then lint.
  Expect duplicate findings — two bases that were worth merging were covering the same ground.

**Never.** Renaming or adding a layer, adding a fourth page type, changing the front-matter
schema, or moving `core-sources/` out from under the root. These are what the skills check
against. Changing them locally means every future version of the kit fights your base.
