# DOCS.md — the schema

This file is the contract between you and the agent. The agent reads it before every
`scan-docs`, `ask-docs`, and `lint-docs` run, and `init-docs` writes it. Edit it to change how
the knowledge base behaves; do not edit the agent's skills for project-specific rules.

## Layers

Everything lives under `docs/`, but the layers are still strictly separate. **`docs/sources/`
is carved out of the agent's territory** — nesting it here is a filing decision, not a
permission change.

| Layer | Path | Who writes it |
|---|---|---|
| Sources | `docs/sources/` | **Humans only.** Immutable. The agent reads it, never edits, renames, or deletes. |
| Pages | `docs/summaries/`, `docs/topics/`, `docs/entities/` | **Agent only.** Every file is derived from `docs/sources/` or from an explicit instruction. |
| Bookkeeping | `docs/README.md`, `docs/CHANGELOG.md` | **Agent only.** Index and log. |
| Guidance | `docs/DOCS.md` | **Humans.** The schema. The root `AGENTS.md` points here. |

Two consequences that are easy to get wrong now that `sources/` is nested:

- **`docs/sources/` is exempt from every page rule below.** Front matter, slugs, wikilinks,
  page types — none of it applies to raw material. Only the `YYMMDD` file-name rule does.
- **Checks that sweep `docs/` skip `docs/sources/`.** A PDF there is not an orphan page, and
  a source with no summary is a `lint-docs` finding of its own kind, not a schema violation.

If a fact is not traceable to a source or to a human instruction, it does not belong in a page.

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

## Bookkeeping files

- `docs/README.md` — the index. Every page in `summaries/`, `topics/`, and `entities/` is listed here exactly once, grouped by
  type, with a one-line description. A page not in the index is an orphan and `lint-docs` reports it.
- `docs/CHANGELOG.md` — append-only. One entry per `scan-docs` or `lint-docs` run, newest at the top.
  Never rewrite or delete past entries.

## Front matter

Every page starts with:

```yaml
---
type: summary | topic | entity
title: Human readable title
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [docs/sources/foo.pdf]     # summaries: exactly one. topics/entities: all that back it.
confidence: high | medium | low
---
```

## Links

Use Obsidian wikilinks: `[[vector-database]]`, or `[[vector-database|vector DBs]]` to relabel.
Link on first mention of any concept that has (or should have) its own page. A link to a page
that does not exist yet is **encouraged** — it is a to-do, and `lint-docs` collects them as gaps.

> To use plain relative Markdown links instead, change this section and tell the agent to
> migrate; the skills read this file for link style rather than assuming one.

## Citations

Claims in topic and entity pages carry a source: `... throughput dropped 40% ([[docs/sources/260415-bench|260415-bench]], p.4).`
Uncited claims are allowed only for definitions and connective prose.

## Contradictions

Never silently overwrite a claim that conflicts with an existing one. Record both:

```markdown
> [!warning] Contradiction
> [[docs/sources/a]] reports X. [[docs/sources/b]] reports not-X. Unresolved as of 2026-08-30.
```

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
- Architectural decisions get a topic page with the alternatives that were rejected. A
  decision without its discarded options is not documentation, it is a description.
- Environment variables and endpoints are entities, never bullet lists buried in a summary.
  They get referenced from too many places to live inside one page.

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

