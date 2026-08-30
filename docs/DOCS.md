# DOCS.md — the knowledge governance contract

This file is the contract between you and the agent. The agent reads it before every
`scan-docs`, `ask-docs`, `lint-docs`, and `eval-docs` run, and `init-docs` writes it. Edit it to
change how the knowledge base behaves; do not edit the agent's skills for project-specific rules.

It governs six things: **what a page is**, **what kind of claim it carries**, **where that claim
came from**, **whether it is still current**, **who is allowed to decide**, and **what the agent
may read**.

## Layers

Everything lives under `docs/`, but the layers are still strictly separate. **`docs/sources/`
is carved out of the agent's territory** — nesting it here is a filing decision, not a
permission change.

| Layer | Path | Who writes it |
|---|---|---|
| Sources | `docs/sources/` | **Humans only.** Immutable. The agent reads it, never edits, renames, or deletes. |
| Pages | `docs/summaries/`, `docs/topics/`, `docs/entities/` | **Agent only.** Every file is derived from `docs/sources/` or from an explicit instruction. |
| Evals | `docs/evals/` | **Humans.** Questions the knowledge base must answer correctly. |
| Bookkeeping | `docs/README.md`, `docs/CHANGELOG.md` | **Agent only.** Index and log. |
| Guidance | `docs/DOCS.md` | **Humans.** The schema. The root `AGENTS.md` points here. |

Two consequences that are easy to get wrong now that `sources/` is nested:

- **`docs/sources/` is exempt from every page rule below.** Front matter, slugs, wikilinks,
  page types — none of it applies to raw material. Only the `YYMMDD` file-name rule does.
- **Checks that sweep `docs/` skip `docs/sources/`.** A PDF there is not an orphan page, and
  a source with no summary is a `lint-docs` finding of its own kind, not a schema violation.

If a fact is not traceable to a source or to a human instruction, it does not belong in a page.

## Authority

The order below is the source-of-truth hierarchy. Higher wins when two levels disagree.

```
Human decision recorded in a page   ← highest authority
        ↑
Source material in docs/sources/
        ↑
Agent-generated page content
        ↑
The agent's background knowledge    ← no authority; must be labelled if used at all
```

What follows from it:

- **The agent may discover, summarize, classify, link, flag, and propose. It may not decide.**
  Establishing that something is true for this project is a human act.
- **`confidence: high` is not authority.** Confidence describes how well evidence supports a
  claim. It never promotes an agent's interpretation into a project decision.
- **A summary is not a source.** Never cite `docs/summaries/x.md` as the origin of a fact.
  Cite the source the summary was derived from.
- An agent that believes a human decision is wrong records that as an open question or a
  contradiction. It does not edit the decision.

## Page types

| Type | Path | One page per | Purpose |
|---|---|---|---|
| Summary | `docs/summaries/<source-slug>.md` | source file | Faithful condensation of one source. The only page allowed to paraphrase at length. |
| Topic | `docs/topics/<slug>.md` | idea, question, or theme | Synthesis across sources. Where claims are compared and contradictions surface. |
| Entity | `docs/entities/<slug>.md` | person, org, product, place, dataset | Stable facts about one thing, plus links to where it appears. |

Slugs are lowercase kebab-case, ASCII. `docs/entities/vector-database.md`, not
`Vector Databases (2026).md`.

### Dates in file names

**Any date in a file name is `YYMMDD`.** No separators, no other order, no exceptions —
`docs/sources/260830-quarterly-review.pdf`, `docs/summaries/260830-quarterly-review.md`. It sorts
chronologically by default and keeps slugs short.

Only include a date when the file is genuinely dated — a dated report, a transcript, a
snapshot. Evergreen topic and entity pages carry no date at all.

A summary page inherits its source's date prefix verbatim, so the two always line up.

This governs file names only. **Dates *inside* files stay `YYYY-MM-DD`** — front matter
`created` / `updated`, changelog headings, and dates in prose.

## Front matter

Front matter identifies and governs the document. **The knowledge itself lives in the Markdown
body, never in YAML.** This split is deliberate: agents corrupt nested YAML far more readily
than they corrupt prose, so the metadata is kept small enough to be hard to break.

Six required fields:

```yaml
---
type: summary | topic | entity
title: Human readable title
status: draft | active | stale | superseded | deprecated | archived
claim_type: fact | decision | assumption | hypothesis | open-question | contradiction | instruction
updated: YYYY-MM-DD
sources: [docs/sources/foo.pdf]     # summaries: exactly one. topics/entities: all that back it.
---
```

Three optional fields, added only when they carry information:

```yaml
created: YYYY-MM-DD
confidence: high | medium | low     # how well the evidence supports the page. Not authority.
superseded_by: docs/topics/authentication-v2.md   # required when status is superseded
```

Rules:

- **Flat scalars and flat lists only.** No nested maps, no lists of maps, no YAML anchors. If a
  relationship needs structure, it goes in a Markdown section, not in front matter.
- **`sources:` is the one list**, and it holds plain paths. Section and line detail belongs in
  the inline citation, not here.
- Unknown extra fields are tolerated and ignored. Missing required fields are a lint finding.
- On a summary page, `claim_type` defaults to `fact` and may be omitted — a summary condenses
  one source and carries whatever that source carries. On topic and entity pages it is required.

### Backward compatibility

Pages written before `status` and `claim_type` existed remain valid. `lint-docs` reports a page
missing either field as **WARNING**, not ERROR, and offers to add them:

- `status` defaults to `active`.
- `claim_type` defaults to `fact` on summaries and to `open-question` on a topic or entity page
  the agent cannot classify from its content. It never guesses `decision`.

No migration is required. Pages are upgraded as they are next touched.

## Claim types

These are not interchangeable, and collapsing them is the most consequential error this kit
exists to prevent. All four of these sentences are different, and a knowledge base that stores
them identically is lying:

| Sentence | Claim type |
|---|---|
| Authentication uses Redis. | `fact` |
| The team chose Redis for session storage. | `decision` |
| We believe Redis is required here. | `assumption` |
| Redis might reduce p99 latency; untested. | `hypothesis` |

The full set:

| `claim_type` | Means | Established by |
|---|---|---|
| `fact` | The sources support this as true of the world. | Evidence |
| `decision` | A human chose this for this project. | **Human only** |
| `assumption` | Taken as true to make progress; not evidenced. | Either, if labelled |
| `hypothesis` | Proposed, testable, untested. | Either, if labelled |
| `open-question` | Nobody knows yet. Evidence is missing. | Either |
| `contradiction` | Sources disagree and it is unresolved. | Either |
| `instruction` | A rule for how work is done here. | **Human only** |

**An agent may never set `claim_type: decision` or `claim_type: instruction` on its own.** Those
two are human acts. An agent that thinks a decision was made writes `open-question` and asks.

### Marking claims inside a page

A page has one dominant `claim_type` in its front matter. Individual claims inside the body are
marked with Obsidian callouts, which render in Obsidian and grep cleanly:

```markdown
Sessions expire after 30 minutes ([[docs/sources/260415-auth-spec|260415-auth-spec]], §4.2).

> [!check] Decision
> Session storage moved to Redis on 2026-03-11. Rejected: in-process cache (loses state on
> deploy), Postgres (write amplification under load). — approved by @fenton

> [!note] Assumption
> Traffic stays under 5k concurrent sessions. Not measured; carried over from the 2025 design.

> [!question] Open question
> Nothing in the sources describes session revocation on password change.
```

Unmarked prose is a `fact` and must be cited. The callout types are `[!check]` decision,
`[!note]` assumption, `[!abstract]` hypothesis, `[!question]` open question, `[!warning]`
contradiction, `[!important]` instruction.

## Lifecycle

Obsolete knowledge must not look identical to current knowledge.

```
draft ──→ active ──→ stale ──→ active        (re-scanned against current sources)
             │          │
             │          └────→ superseded ──→ archived
             └────→ deprecated
```

| `status` | Means | Who sets it |
|---|---|---|
| `draft` | Being written; incomplete. Not yet trustworthy. | Agent or human |
| `active` | Current and believed accurate. | Agent (on write), human (on approval) |
| `stale` | A source it depends on changed after `updated`. Content may still be right — nobody has checked. | Agent |
| `superseded` | Replaced by another page. **Requires `superseded_by`.** | Human |
| `deprecated` | Describes something no longer in use. Kept for history. | Human |
| `archived` | Retired from active retrieval. Still readable, still auditable. | Human |

Rules:

- **An agent may set `draft`, `active`, and `stale`.** Only a human sets `superseded`,
  `deprecated`, or `archived` — those are judgments about the project, not about the files.
- A `superseded` page keeps its content. It is not emptied, and it is not deleted. Add a line at
  the top pointing at the replacement and leave the rest for the audit trail.
- `ask-docs` retrieves `active` and `stale` pages, labels `stale` ones in the answer, and
  excludes `superseded`, `deprecated`, and `archived` unless the user asks for history.

## Provenance

The goal is to move from *"this came from foo.md"* to *"this came from this version of foo.md,
at this section."*

Cite with the most precise anchor the source format actually supports, and stop there:

| Source format | Anchor | Example |
|---|---|---|
| PDF | page | `(260415-bench, p.4)` |
| Markdown, spec, doc | section heading | `(260415-auth-spec, §4.2)` |
| Source code | path and line range | `(src/auth/session.php:112-140)` |
| Transcript, video | timestamp | `(260302-standup, 14:20)` |
| Web page | section, plus capture date | `(260110-redis-docs, "Expiration", captured 2026-01-10)` |
| Anything else | the file alone | `(260415-notes)` |

In a page, the file part of the anchor is a wikilink, as shown under *Citations* below.

**Never invent an anchor.** A page number you did not see, a line range you did not verify, and
a section heading you paraphrased are all fabricated provenance, which is worse than no
provenance. Falling back to the bare file name is always correct.

For sources under version control, a claim that depends on a specific revision records it:

```markdown
Sessions are stored in Redis (`src/auth/session.php:112-140`, as of `a3f9c21`).
```

Use a commit hash only when the agent can actually read it from git. Do not fabricate hashes.

## Freshness

A page is `stale` when something it depends on changed after its `updated` date:

```
source file changes  →  summary of that source goes stale
                     →  every topic/entity page citing that source goes stale
```

Detection is by comparison, not by a dependency graph:

- A summary is stale if its source's modification time (or last commit date) is newer than the
  page's `updated`.
- A topic or entity page is stale if any path in its `sources:` list changed after `updated`.
- For code sources, the cited file path changing is the signal. A cited line range that no longer
  exists is a **broken citation**, not staleness.

`lint-docs` sets `status: stale` and reports it. Re-scanning the source is what clears it. Staleness
means *unverified*, not *wrong* — an agent answering from a stale page says so rather than
withholding the answer.

## Links

Use Obsidian wikilinks: `[[vector-database]]`, or `[[vector-database|vector DBs]]` to relabel.
Link on first mention of any concept that has (or should have) its own page. A link to a page
that does not exist yet is **encouraged** — it is a to-do, and `lint-docs` collects them as gaps.

This is Obsidian Flavored Markdown, so `docs/` opens directly as a vault — graph view,
backlinks, and unresolved links all work without conversion.

> To use plain relative Markdown links instead, change this section and tell the agent to
> migrate; the skills read this file for link style rather than assuming one.

## Citations

Claims in topic and entity pages carry a source, anchored per the provenance table above:

```markdown
... throughput dropped 40% ([[docs/sources/260415-bench|260415-bench]], p.4).
```

Uncited claims are allowed only for definitions and connective prose.

**A citation is a claim that a document was read.** An agent may cite only sources and pages it
actually opened in the current run. Citing a plausible-looking path it did not read is
fabrication, and `eval-docs` treats it as a hard failure.

## Contradictions

**Contradictions are never silently resolved.** Two sources disagreeing is a finding, not a bug —
and the ability to surface it is the main thing this kit offers over plain retrieval.

Record both sides, on the topic page that owns the question:

```markdown
> [!warning] Contradiction
> [[docs/sources/260415-auth-spec|260415-auth-spec]] (§4.2) gives session TTL as 30 minutes.
> [[docs/sources/260710-ops-runbook|260710-ops-runbook]] (p.2) gives it as 24 hours.
> Unresolved as of 2026-08-30.
```

Set `claim_type: contradiction` on the page while it stands.

The agent must not close a contradiction by reasoning that one source looks newer, more
official, or more detailed. Recency is only decisive when this file says so — for example, a
project rule reading *"the runbook supersedes the spec for operational values."* Absent such a
rule, it stays open until a human rules on it.

## Human review

This is the documented path out of `contradiction` and `open-question`. Both are resolved the
same way, and **the evidence is preserved, never overwritten**.

1. A human reads both sides and the underlying sources.
2. They rule.
3. The page is rewritten to the shape below — the ruling on top, the history intact underneath.
4. `claim_type` becomes `decision` (a project choice) or `fact` (the evidence actually settled
   it). `status` becomes `active`.
5. The change is committed, so the ruling has an author and a date in git.

```markdown
---
type: topic
title: Session TTL
status: active
claim_type: decision
updated: 2026-08-30
sources: [docs/sources/260415-auth-spec.pdf, docs/sources/260710-ops-runbook.pdf]
---

## Decision

Session TTL is 24 hours. — @fenton, 2026-08-30

## Rationale

The spec and the runbook disagreed. The runbook reflects what production has run since the
March migration; the spec was not updated. The runbook is authoritative for operational values.

## Evidence

- [[docs/sources/260415-auth-spec|260415-auth-spec]] (§4.2) — 30 minutes. Superseded by the
  March 2026 migration.
- [[docs/sources/260710-ops-runbook|260710-ops-runbook]] (p.2) — 24 hours. Matches production.
```

What makes this auditable is that the disagreement is still on the page after it is settled.
Deleting the losing side destroys the reason the decision exists.

## Retrieval and context

`ask-docs` must never load the whole knowledge base. At a thousand pages that is impossible; at
a hundred it is already wasteful and it buries the relevant page in noise.

Retrieval is a fixed sequence, and the mechanism is deliberately replaceable — swapping how
candidates are found must not change anything about the knowledge format:

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

**Selection** is by index and links, not by reading everything: match the question against
`docs/README.md` titles and descriptions, read those pages, then follow their wikilinks one hop
(two if the question is broad). Drop to `docs/sources/` only when the pages are thin, a verbatim
quote is needed, or a page is suspected stale.

**The budget is roughly half the agent's context window**, leaving room for the sources, the
reasoning, and the answer. Estimate before reading, at roughly 4 characters per token — file
sizes are enough to do this, and a PDF costs far more than its byte count suggests.

**Over budget is a graceful failure, not a truncation.** Say what was found, why it does not
fit, and what would narrow it:

```
The evidence for this question exceeds the context budget: 34 pages across 6 topics
(~180k tokens). Narrow the scope and I can answer precisely:
  --topic authentication     (7 pages)
  --topic rate-limiting      (5 pages)
  --source 260415-auth-spec.pdf
```

Dropping sources silently and answering as though the picture were complete is the worst
available outcome. It is indistinguishable from a correct answer and it is unauditable.

Scope may be constrained explicitly: `--topic <slug>`, `--entity <slug>`, `--source <path>`.

## Answer states

Every answer separates what is supported from what is not:

- **Known** — supported by cited evidence.
- **Inferred** — the agent's reasoning across sources. Always labelled. Never presented as fact.
- **Contradicted** — sources disagree. Both sides shown.
- **Unknown** — no source covers it. Say so and name the gap.

"I don't know based on the available knowledge" is a correct and useful answer. An unsupported
inference presented as a fact is a defect, regardless of whether it happens to be right.

## Security and exclusions

**Not every file in a repository is safe to read into an LLM.** The agent never reads, ingests,
summarizes, or quotes from these paths, even when a source or a user's question points at them:

```
.env, .env.*
*.pem, *.key, *.p12, *.pfx
id_rsa, id_dsa, id_ecdsa, id_ed25519
secrets/**, credentials/**, private/**
**/*.secret, **/*secrets*.y*ml
.aws/**, .ssh/**, .netrc
```

Add project-specific paths under *Out of scope* below. If a source file turns out to contain
credentials, the agent stops, does not write a summary, and tells the user — it does not
redact-and-continue, because the secret is already in its context and a partial summary
normalizes the leak.

Pages may carry an optional sensitivity marker in front matter when a project needs one:

```yaml
sensitivity: public | internal | confidential
```

It is documentation, not enforcement — it tells a human what they are about to share. Nothing
in this kit can stop a determined agent from reading a file. The exclusion list is the real
protection; the marker is a label.

## Git conflicts

A merge conflict inside `docs/` is a **semantic** conflict, and git cannot resolve it.

Git can tell you that a human wrote X and an agent wrote Y in the same place. It cannot tell you
which is true. An agent that resolves the textual conflict has silently made a knowledge
decision it has no authority to make.

**An agent never auto-resolves a conflict in `docs/topics/`, `docs/entities/`, or
`docs/summaries/`.** It surfaces both sides and stops:

```
conflict → human reads both sides → checks the underlying sources
        → resolves the Markdown → /lint-docs → commit
```

Two exceptions the agent may resolve itself, because neither carries meaning:

- `docs/CHANGELOG.md` — append-only, so keep both entries in date order.
- `docs/README.md` — an index, so keep the union of the lines and re-sort.

If a conflict in a page is genuinely two claims about the same thing, the resolution is a
contradiction callout, not a choice.

## Style

- Present tense, plain language, no hedging filler.
- Short pages that link out beat long pages that repeat.
- Prefer editing an existing page over creating a near-duplicate. If two pages overlap by
  more than roughly half, merge them and leave a redirect line in the retired one.

## Preset: documenting a codebase

If this knowledge base documents the project it lives in — the most common use — replace the
page types above with these, and delete this section once you have.

| Type | Path | One page per | Purpose |
|---|---|---|---|
| Summary | `docs/summaries/<module>.md` | module or package | What it does, its public surface, what it depends on, what depends on it. |
| Topic | `docs/topics/<slug>.md` | decision, flow, or invariant | Why it is built this way. How a request moves through. What must stay true. |
| Entity | `docs/entities/<slug>.md` | service, table, endpoint, job, env var, external API | Stable facts, plus every place it is referenced. |

Then set the source layer. Code is read in place rather than copied:

```markdown
- Treat `src/**` as sources, in addition to `docs/sources/`.
- Out of scope: `node_modules/**`, `dist/**`, `**/*.generated.*`, `vendor/**`, test fixtures.
```

Rules worth adding for code:

- A summary page names the file paths it covers, so `lint-docs` can tell when the code moved.
- Never document intended behavior as actual behavior. If the code and a comment disagree,
  that is a contradiction — record both.
- Architectural decisions get a topic page with `claim_type: decision` and the alternatives that
  were rejected. A decision without its discarded options is not documentation, it is a
  description.
- Environment variables and endpoints are entities, never bullet lists buried in a summary.
  They get referenced from too many places to live inside one page.

## Out of scope

Paths the agent never reads, beyond the security exclusions above. Replace with this project's own:

```
docs/sources/archive/**
```

## Project-specific rules

These are the defaults shipped with the starter kit. They are deliberately few — edit, delete,
or replace them for your own knowledge base. `/init-docs` rewrites this section from an
interview; if it still reads like the text below, that interview never happened.

- Every topic page ends with an **Open questions** section. An empty one is a signal, not a
  defect: it means nothing is unresolved yet.
- Every entity page opens with a one-sentence definition before anything else, so a reader who
  follows a wikilink mid-sentence gets oriented in one line.
- Ignore `docs/sources/archive/**` unless the user names a file in it explicitly.
- Figures carry their unit and their as-of date: `$4.2M (FY2025)`, `340ms p99 (2026-04-15)`.
  A bare number in a topic page is a lint finding.
- Cap summary pages at roughly 600 words. If a source needs more, that is a sign it should
  produce topic pages, not a longer summary.
