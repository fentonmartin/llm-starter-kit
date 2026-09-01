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

Three paths are declared rather than assumed. **This block is the declaration** — every rule in
this file and every path in the agent's skills resolves against it.

> **Root: `docs/`**
> **Source folder: `docs/core-sources/`**
> **Index: `docs/INDEX.md`**

Every path written below, and in every skill, uses those defaults. If a declaration names
something else, read the corresponding path that way throughout — `docs/topics/` means
`<root>/topics/`, a check that skips the source folder skips it wherever it is declared, and the
index is whichever file the third line names. Nothing else changes: the layers, their names, and
their rules are the same at any path.

**Only one of the three is a choice.**

- **The root is not configured.** It is `docs/`. The single exception is a project whose `docs/`
  is already built by a site generator, where the base is scaffolded at `docs/kb/` and the root
  declaration records that — a fallback the agent applies on its own, never a question put to the
  user. Do not offer a root, and do not ask for one.
- **The source folder is the one real choice**, because material is often already filed somewhere.
  See *Choosing the source folder* in Part 2. With nothing configured it is
  `docs/core-sources/`.
- **The index name follows a rule, not a preference** — `docs/INDEX.md` unless the project has no
  root `README.md` of its own and nothing else claims the name, in which case `docs/README.md`
  reads more naturally. See *The index* below.

Everything else lives under the root, and the layers are strictly separate.

| Layer | Path | Who writes it |
|---|---|---|
| Core sources | `docs/core-sources/` — or wherever the declaration above points | **Yours only.** Immutable. The agent reads it, never edits, renames, or deletes. |
| Pages | `docs/summaries/`, `docs/topics/`, `docs/entities/` | **Agent only.** Every file derives from a source or an explicit instruction. |
| Scenarios | `docs/scenarios/` | **Yours.** Questions the knowledge base must answer correctly. |
| Bookkeeping | the index, `docs/CHANGELOG.md` | **Agent only.** Index and log. |
| Guidance | `docs/DOCS.md` | **Yours.** This file. |

The source folder is exempt from every page rule below — front matter, slugs, wikilinks, page
types. Only the `YYMMDD` file-name rule applies there. Checks that sweep `docs/` skip it.

Three constraints on the declaration:

- **Exactly one location.** One summary page per source file is the backbone of provenance, and
  two source folders make that mapping ambiguous the first time a file name repeats. Material that
  lives in several places gets filed into one folder, or read in place — see below. Three forms
  are valid:

  | Declaration | Means |
  |---|---|
  | `docs/core-sources/` | The default. Inside the base, so the whole knowledge base is one folder to copy or open as a vault. |
  | `sources/` | At the top of the project, beside `docs/`. More visible, easier to drop files into. Chosen at setup; identical in every other respect. |
  | any other folder path | A folder that already holds the material, wherever it is, including outside the root. Subfolders inside it are read. |

  It must be a folder. There is no glob form and no way to declare "the project root" — a
  declaration that swept the whole repository would put `package.json` and every stray file into
  the provenance chain, and the immutability rule would then cover half the project.
- **It must not overlap a page layer.** A source folder that contains `summaries/`, `topics/`, or
  `entities/` makes the agent's own output look like source material, and there is no recovering
  from that automatically.
- **It stays yours wherever it is.** Declaring a folder as the source layer does not make it
  safer to write to; it makes it forbidden to write to. Point the declaration at a folder the
  agent should never author in, never at one it maintains.

**If a fact is not traceable to a source or to an instruction from you, it does not belong in a page.**

**The agent never adds, renames, or removes a layer, and never adds a page type.** This project's
own vocabulary belongs in *Page types* below, not in folder names. Changing the shape is your
decision, recorded in `docs/CHANGELOG.md` — Part 2's *Changing the shape later* says what each
kind of change costs.

### Sources that will not fit, and sources that cannot be read

A source larger than the agent can hold is **not** summarized from the part it managed to read.
It reports, and you choose: split the file into parts that each become their own source, name a
range to summarize as an explicit partial, or leave it unread. A partial summary is always marked
as one — `status: draft`, the range named in the first line, the remainder an open question.

A source that cannot be read at all — a scanned PDF with no text layer, a format the agent cannot
open, a link it cannot fetch — produces **no page**. It stays unread and lint keeps reporting it.
An unread source is a known gap; an invented summary is a false one, and nothing downstream can
tell the difference.

### Keeping the repository usable

The source layer fills with things git is bad at. A few hundred PDFs is a repository nobody
enjoys cloning, and unlike pages, sources never get smaller.

- **Text beats binary** where you have the choice. A Markdown export of a spec diffs, greps and
  reviews; the PDF of the same spec does none of that.
- **Git LFS is the normal answer** once binaries are large or numerous, and it changes nothing
  here: paths stay the same, so citations stay valid.
- **Gitignoring the source folder is allowed, and it has a price.** The base still works for you,
  but every citation now points at a file nobody else can open — provenance that cannot be
  checked by anyone but you. If you do it, say so at the top of the source folder's README, and
  keep the material somewhere it can be produced on request.
- Whatever you choose, **the paths in `sources:` and in citations must stay stable**. Provenance
  is path-based; moving material to satisfy a storage decision breaks every page that cited it,
  and repairing that is a lint run you did not need to have.

### Read-in-place sources

Optional, and empty by default. Some material should be cited but not filed: a source tree that
changes every commit, or a published docs folder this base was scaffolded beside. List those paths
here and the agent reads and cites them without copying them in:

```
(none)
```

They differ from the source folder in three ways. They produce no summary page, so *unread source*
checks skip them. They are versioned, so a citation to one records the commit it was read at. And
they are not curated — nobody chose to file them, which is why a claim from one carries less
weight than the same claim from the source folder.

### Why "core"

The prefix names a **role**, not a file type: the root of the provenance chain, the one layer every
page is reproducible from. Deleting a page loses work; deleting from here loses the truth. The
practical payoff is that the hard rule reads as a boundary rather than a category — *never write to
the source folder*.

**The name is a default, the role is not.** A project that calls it `sources/` or `notes/` or
`research-papers/` loses nothing, as long as one location holds the curated material and the agent
never writes there. What cannot vary is that the role exists and that exactly one location fills
it. Keep `core-sources/` unless the project has a reason; a reader who has seen the kit before
knows what it is on sight.

Part 2's *Why the structure is shaped this way* has the rest — what every layer's name is doing,
why the layers are separate, and how an agent reads the base without reading all of it.

## Authority

```
Your decision, recorded in a page    ← highest
Source material in docs/core-sources/
Agent-generated page content
The agent's background knowledge    ← none; must be labelled if used at all
```

- **The agent discovers, summarizes, classifies, links, flags, and proposes. It does not decide.**
- **`confidence` is not authority.** It describes how well evidence supports a claim.
- **A summary is not a source.** Cite the source a summary came from, never the summary.
- An agent that believes one of your decisions is wrong records an open question. It does not edit
  the decision.

## Page types

| Type | Path | One page per | Purpose |
|---|---|---|---|
| Summary | `docs/summaries/<source-slug>.md` | source file | Faithful condensation of one source. The only page allowed to paraphrase at length. |
| Topic | `docs/topics/<slug>.md` | idea, question, theme | Synthesis across sources. Where contradictions surface. |
| Entity | `docs/entities/<slug>.md` | person, org, product, place, dataset | Stable facts about one thing, plus where it appears. |

### Subfolders

**Group with subfolders once a layer passes roughly thirty pages.** A path like
`docs/topics/auth/session-management.md` reads better than forty files in one directory, and it
gives a reader an outline before they open anything. Below thirty, a flat layer is easier to scan
than a shallow tree.

Rules that make grouping safe:

- **Group by subject, never by type or date.** `topics/auth/`, `entities/services/`,
  `summaries/vendor-docs/`. Never `topics/2026/`, and never a subfolder that re-states the layer.
- **The layer folder stays the layer.** A subfolder never changes what a page *is*: everything
  under `docs/topics/` is a topic page with `type: topic`, however deep. Never nest one layer
  inside another.
- **Two levels is the practical limit.** Deeper and the path becomes the taxonomy, which is a
  filing system nobody maintains. If two levels are not enough, the pages are too small — merge
  them.
- **Slugs stay globally unique within a layer**, not merely unique within their subfolder.
  `topics/auth/tokens.md` and `topics/billing/tokens.md` are a wikilink ambiguity, and Obsidian
  resolves `[[tokens]]` to whichever it saw first. Two pages that want the same slug are usually
  one page, or want more specific names.
- **A summary's path mirrors its source's.** A source at `core-sources/vendor/acme-spec.pdf` gets
  `summaries/vendor/acme-spec.md`. That is what lets lint pair the two, and how a moved source
  is detected rather than silently orphaned.
- **Moving a page between subfolders is free** — the slug and the wikilinks do not change, so
  nothing breaks. Regroup whenever the shape stops fitting; the index is rebuilt either way.

## Front matter

**Front matter identifies and governs the document. The knowledge lives in the Markdown body,
never in YAML.**

Six required fields:

```yaml
---
type: summary | topic | entity
title: Readable title
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
sensitivity: public | internal | confidential      # a label for people, not enforcement
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
| `decision` | You chose this for this project. | **Yours only** |
| `assumption` | Taken as true to make progress; not evidenced. Covers untested proposals too. | Either, if labelled |
| `open-question` | Nobody knows yet. Evidence is missing. | Either |
| `contradiction` | Sources disagree and it is unresolved. | Either |

*"Authentication uses Redis"*, *"the team chose Redis"*, and *"we believe Redis is required"* are
three different claims. Storing them identically is how a knowledge base starts lying.

**An agent may never set `claim_type: decision`.** One that thinks a decision was made writes
`open-question` and asks. That is the only claim value reserved to you.

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
| `draft` | Being written; incomplete. | Agent or you |
| `active` | Current and believed accurate. | Agent or you |
| `stale` | A source it depends on changed after `updated`. Unverified, not wrong. | Agent |
| `superseded` | Replaced by another page. **Requires `superseded_by`.** | **Yours only** |

- **An agent may set only `draft`, `active`, and `stale`.** `superseded` is the one status
  reserved to you, because retiring a page is a judgment about the project.
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
stays open until you rule. → [Review and ruling](#review-and-ruling)

## Retrieval and context

`ask-docs` must never load the whole knowledge base.

**Selection:** match the question against the index titles and descriptions, read those
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

## Who you are working with

This base has one **owner**: the person who decides what it says. The rules throughout this file
say *the owner* where authority matters — who may write to a layer, who rules on a contradiction,
whose decision `claim_type: decision` records.

When you speak to them, use their name rather than a role:

> **Address: (not set)**

- `init-docs` asks once, offering whatever `git config user.name` reports. Whatever they answer
  goes here verbatim, including a form of address they choose for themselves.
- **If it is `(not set)`, use "you" and nothing else.** Never guess a name from a git log, an
  email address, or a commit author — that is a stranger's name as often as it is theirs — and
  never invent a form of address.
- Use it where a person would: greeting a report, asking for a ruling, flagging something that
  needs them. Not on every line. A name repeated in every sentence reads worse than no name at
  all.

It changes nothing about authority. A name is how you address the owner, not evidence of who
they are, and it never appears in a page — pages record what the sources say and what the owner
ruled, and a ruling is attributed from git, where it can be checked.

## Commits

Whether the agent commits its own work, declared here and nowhere else:

> **Commits: `none`**

| Value | Behaviour |
|---|---|
| `none` | **The default.** The agent writes files and stops. You review the working tree and commit. |
| `per-run` | One commit at the end of each command run, covering only what that run wrote. |
| `per-file` | One commit per file written. Noisy, but every page is individually revertable. |

**`none` is the default for a reason, and it is not timidity.** Reading `git diff` after a scan is
how you find out where the agent's judgment differs from yours, and it is the feedback loop that
turns a generic base into a good one. Auto-committing does not remove that review — it removes the
*moment* that prompts it. Turn it on once the rules in this file have stopped changing, not before.

Rules that hold at every setting:

- **Stage only what the run wrote.** Named paths, never `git add -A` or `git add .`. A knowledge
  base run must never sweep up unrelated work in progress, and someone with a half-finished branch
  open is the normal case, not the exception.
- **Never push.** Committing is local and reversible; pushing is neither. Pushing is always your
  act, whatever this setting says.
- **Never commit when a security exclusion fired.** If a run finds excluded material in the source
  layer, it stops and reports rather than committing anything from that run.
- **Never commit a conflict.** Unresolved conflict markers are reported, never staged.
- **One line, then the detail.** `docs(kb): scan — 3 sources, 7 pages` on the subject line; what
  needs your judgment in the body. The changelog entry is still written either way — a commit message is
  not a substitute for it, since one lives in git and the other lives in the base.
- **The setting never suppresses a report.** Whatever gets committed, the run still says what it
  did and what needs judgment. A commit is not a substitute for telling someone.

## Git conflicts

A merge conflict inside `docs/` is a **semantic** conflict. Git knows someone wrote X and an agent
wrote Y; it does not know which is true.

**An agent never auto-resolves a conflict in `docs/topics/`, `docs/entities/`, or
`docs/summaries/`.** Surface both sides and stop. Two exceptions, because neither carries meaning:
`docs/CHANGELOG.md` (keep both, date order) and the index (keep the union, re-sort).

If a conflict is genuinely two claims about the same thing, the resolution is a contradiction
callout, not a choice.

## Naming and dates

- Slugs are lowercase kebab-case ASCII. `docs/entities/vector-database.md`. Subfolder names follow
  the same rule, and a slug must be unique across its whole layer, subfolders included.
- **Any date in a file name is `YYMMDD`** — no separators, no exceptions. Only genuinely dated
  files get one; evergreen topic and entity pages get none. A summary inherits its source's
  prefix verbatim.
- **Dates inside files are `YYYY-MM-DD`** — front matter, changelog headings, prose.

## Bookkeeping

- The index, at whatever name the declaration gives it — `docs/INDEX.md` by default. Every page listed
  exactly once, grouped by type, one line each. Updated in the same pass that creates or removes
  a page.
- `docs/CHANGELOG.md` — append-only, newest first. One entry per run. Never rewrite an entry.

### The index

The index is the entry point. `ask-docs` reads it before it reads any page, and it is how a base
stays answerable as it outgrows anyone's context: selection is by index and links, never by
reading everything. A page missing from it is invisible in practice, which is why an omission is
a lint finding rather than a matter of tidiness.

**`INDEX.md` unless `README.md` is clearly better.** The file is an index and has never been a
readme, and two cases make the accurate name the necessary one:

- The project's own root `README.md` is the front door for the whole repository, and a second
  README one level down reads as competing with it. `INDEX.md` says what the file is, and no
  reader has to work out which README is authoritative.
- A `README.md` already exists in the base and *is* a readme — an introduction to the folder,
  written for people, not a list of pages. Do not convert it; declare `INDEX.md` and leave it
  alone.

Since almost every project has a root README, `INDEX.md` is the usual outcome. `README.md` is
still valid and is what bases built before `2.3` use: either name works, the declaration settles
which, and nothing in this contract treats them differently. There is no reason to rename an
existing one except preference.

**The project's own root `README.md` is not part of the knowledge base**, and the agent does not
maintain it. It is the repository's front door, and a knowledge base is a thing the front door
should point *at*: one line saying the base exists, what it covers, and which commands read it.
`init-docs` adds that line, or writes a minimal root README if the project has none, and touches
nothing else in it.

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

## Why the structure is shaped this way

Worth reading once. Every folder name here is doing a job, and the jobs are what the rules in
Part 1 protect.

### How an agent actually reads this base

Not by reading it. That is the whole design:

```
DOCS.md          →  the rules, every run, in full
index            →  what exists, and where
2–8 pages        →  selected by index and wikilinks
source folder    →  only when the pages are thin or a quote is needed
```

An agent that globs `docs/**/*.md` has already failed, because a base that is working outgrows any
context window. So the structure is optimized for *selection*: the index says what exists in one
screen, wikilinks say what is related without reading either page, front matter says whether a
page is worth opening (`status`, `claim_type`, `updated`) before its body is loaded, and citations
say where to go when the pages are not enough. Every one of those is a way to *avoid* reading.

That is also why front matter holds no knowledge. Knowledge in YAML cannot be linked, quoted, or
contradicted — only parsed. The body is for what the base knows; the front matter is for deciding
whether to read the body.

### Why each layer is named what it is

| Name | Why not something else |
|---|---|
| `core-sources/` | Not `sources/`, because "source" is already three things here: the `sources:` field, the citation, and source code. The `core-` prefix names a **role** — the root of the provenance chain, the one layer everything else is reproducible from. It also separates curated material from read-in-place paths, which is exactly where the immutability rule has to be sharp. |
| `summaries/` | Not `notes/` or `digests/`. A summary has one job and one source, and the name says both. It is the only page type allowed to paraphrase at length, because faithfulness to one document is what it is *for*. |
| `topics/` | Not `concepts/` or `articles/`. A topic is a question or theme that outlives any one source, and it is where sources are compared — so it is the only layer where a contradiction can exist. `articles/` would suggest prose written for its own sake. |
| `entities/` | Not `glossary/` or `things/`. An entity is one thing with stable facts and many mentions: a service, a person, a table, a term. `glossary/` implies definitions only, and the point of the layer is the *backlinks* — every place the thing appears. |
| `scenarios/` | Not `tests/`. Nothing here executes, and calling it tests promises a green tick this kit cannot give. A scenario is a question the base must answer correctly, checked by reading. |
| the index | Not a generated file. It is the entry point an agent reads before anything else, so it is maintained in the same pass that changes a page — a stale index is a base that has quietly lost pages. |

### Why the layers are separate at all

Because who may write to a file is the only durable way to keep provenance honest. Sources are
yours and immutable; pages are agent-maintained and derived; scenarios are yours and
never edited by the agent; bookkeeping is agent-owned. Merge any two and the question *"where did
this claim come from?"* stops having an answer — which is the failure this whole structure exists
to prevent.

The three page types are separate for a different reason: they fail differently. A wrong summary
misreads one document. A wrong topic page draws a false conclusion from several. A wrong entity
page spreads one bad fact everywhere it is linked. Keeping them apart is what makes a lint finding
specific enough to act on.

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

## Review and ruling

The path out of `contradiction` and `open-question`. **The evidence is preserved, never
overwritten.**

1. You read both sides and the underlying sources.
2. You rule.
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

Then set the source layer. Code is cited but not filed, so it goes in *Read-in-place sources* in
Part 1 rather than in the source folder:

```markdown
src/**
```

```markdown
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
  directly. When two of your own documents disagree, it is a real contradiction needing your
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

Whatever you choose, say it in the `Root:` declaration at the top of Part 1. That line is the only
place the root is declared, and every skill reads this file before it reads anything else.

## Choosing the source folder

`docs/core-sources/` is the default and needs no thought. Reasons to name something else:

| Situation | Source folder | Why |
|---|---|---|
| Normal | `docs/core-sources/` | Filed inside the base, so the whole thing copies as one folder. |
| A fresh start where the user wants the folder visible | `sources/` | The only difference is how far down it sits. Someone who will actually drop files into a top-level folder beats a tidier tree they never fill. |
| The material already lives somewhere the project cares about — `research-papers/`, `contracts/`, `notes/` | that folder | Moving material people already file by hand, into a folder the kit invented, is a cost with no return. Point at it instead. |
| The base was scaffolded beside a published docs site (`docs/kb/`) | `docs/` | The published pages *are* the material. They stay where the generator expects them and are cited at their real paths. |
| The material is your own writing, and the repo is the writing | that folder | A document library's corpus is usually already organized. Declare it; do not refile it. |

Two things this is not for. It is not a way to point the agent at a folder it also writes to —
declaring a folder as the source layer makes it forbidden to write to, and pointing it at
`docs/topics/` breaks the base in a way nothing detects. And it is not a way to have several
source folders: exactly one folder holds filed material, and anything else that needs citing goes
in *Read-in-place sources*.

**Filed or read in place?** File it if someone chose to put it there and it will not change on its
own — a PDF, a spec, an exported transcript. Read it in place if it changes without anyone
deciding it should, like a source tree, or if it is already published somewhere with its own
lifecycle. Filed material gets a summary page and is swept for staleness; read-in-place material
gets neither, and its citations record the commit they were read at.

### Two knowledge bases in one project

Legitimate when a project has two bodies of knowledge with different authority — a codebase base
whose facts come from the code, and a research base whose facts come from other people's papers.
Mixing them is what the split prevents: a vendor claim must never settle a question about your own
system.

- Each base is **completely self-contained**: its own root, its own `DOCS.md` with its own
  own declarations and its own preset, its own five folders, its own index and
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
- **Grouping a layer into subfolders**, or regrouping one. Free in the sense that matters —
  wikilinks resolve by slug, so moving a page between subfolders breaks no link — but the index
  needs rebuilding, so lint and commit. Summary pages move with their sources, not independently.
- **Renaming the index** between `README.md` and `INDEX.md`. `git mv` it, change the declaration,
  and lint: it rebuilds the index and repoints anything that linked to the old name.
- **Changing link style** (wikilinks ↔ relative Markdown). Change the *Links* section, then tell
  the agent to migrate; the skills read this file for link style rather than assuming one.
- **Promoting read-in-place paths to filed sources, or the reverse.** Change the declaration, then
  lint. Promoting produces *unread source* warnings for everything that now wants a summary, which
  is a scan's worth of work, not a defect. Demoting leaves summary pages whose source is no longer
  filed: keep them, and let their citations carry a commit hash from then on.

**Real work — plan it, do it in one commit, lint immediately after.**

- **Moving the root** (`docs/` → `docs/kb/`, say):
  1. Move the folder whole; never lift one layer out from under the others. A move *into* a
     subfolder of itself needs a staging step, because git will not do it in one:

     ```bash
     mkdir kb-tmp && git mv docs/* kb-tmp/
     mkdir -p docs && git mv kb-tmp docs/kb
     ```

     Any other move is the one-liner you would expect: `git mv docs wiki`.
  2. Change the `Root:` declaration in Part 1. This one line is the entire
     re-pointing of the contract and the skills.
  3. Run `/lint-docs`. Check 15 rewrites every `sources:` value and every citation that still
     points at the old root, and reports the ones it cannot.
  4. Grep the rest of the repo for the old path — `CLAUDE.md`, `AGENTS.md`, CI config, and README
     links do not fix themselves.
- **Moving or renaming the source folder.** `git mv` it, change the declaration, then run
  `/lint-docs`: check 15 rewrites every `sources:` value and citation pointing at the old path.
  If you are pointing at a folder that already existed rather than moving one, there is nothing to
  move — change the declaration and expect check 13 to report every file in it as unread, because
  it is. That is a scan queue, not breakage.
- **Splitting one base into two.** Decide the boundary first, by authority — which facts come
  from where — never by volume or by folder size. Then `/init-docs` the second root, `git mv` the
  pages that belong to it, and lint both. Cross-base wikilinks broken by the split must be
  resolved by duplicating the fact with its own citation in the new base, not by linking across.
- **Merging two bases into one.** `git mv` the second base's four layers into the first, port its
  project-specific rules into the survivor's Part 1 (they were written for a reason), then lint.
  Expect duplicate findings — two bases that were worth merging were covering the same ground.

**Never.** Renaming or adding a layer, adding a fourth page type, changing the front-matter schema,
declaring more than one source folder, or pointing the source declaration at a folder the agent
writes to. These are what the skills check against. Changing them locally means every future
version of the kit fights your base.
