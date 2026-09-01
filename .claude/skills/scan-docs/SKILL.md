---
name: scan-docs
description: Read new or changed files in docs/core-sources/ and fold them into the rest of docs/ — writing summary pages, updating the topic and entity pages they touch, flagging contradictions, and appending to the changelog. Use when the user says scan, process my sources, add this article/paper/transcript to the knowledge base, or drops a new file into docs/core-sources/.
---

# Scan

Turn raw material in `docs/core-sources/` into linked pages in `docs/`. This is the only operation that
creates summary pages.

**The root.** Every path in this skill is written as `docs/…`. `docs/DOCS.md` Part 1 opens by
declaring the knowledge base root; if it names anything other than `docs/`, read every `docs/…`
path below as that folder instead. If the user passed a root as an argument, read `<root>/DOCS.md`
and use that base only. The layers, their names, and their rules never change with the root.

## Before you start

1. Read `docs/DOCS.md` **Part 1** in full — it is the complete rules, and it overrides anything
   here: page types, slugs, front matter, claim types, provenance, link style, security
   exclusions, and any project-specific rules at the end. Part 2 is examples and rationale,
   read on demand.
2. Read `docs/README.md` to learn what already exists. Never create a page without checking
   the index for one that covers the same ground.
3. Determine the work set:
   - Explicit file(s) named by the user, or
   - Everything in `docs/core-sources/` with no corresponding `docs/summaries/<slug>.md`, or
   - Files whose mtime is newer than their summary's `updated` field.

If the work set is empty, say so and stop. Do not invent work.

**Before reporting an empty work set, check for a legacy `docs/sources/`.** Knowledge bases built
on `1.x` used that path. If it exists and `docs/core-sources/` does not, say so and offer the
rename rather than reporting that there is nothing to do — an empty work set on a base full of
material is the one failure here that looks like success:

```
docs/core-sources/ does not exist, but docs/sources/ holds 14 files.
This base predates the 2.0 rename. To migrate:
  git mv docs/sources docs/core-sources
  then update the sources: paths in existing pages — /lint-docs will do it.
```

**Check the work set against the security exclusions in `docs/DOCS.md` first.** Skip excluded
paths without reading them. If a source you have already opened turns out to contain
credentials, stop: do not write a summary, do not quote it, and tell the user which file. Do
not redact and continue — the secret is already in your context, and a partial summary
normalizes the leak.

## Per source

Work one source at a time, all the way through, before starting the next. A half-scanned
source is worse than an un-scanned one.

### 1. Read it fully

"Scan" is the name of the command, not the depth of the reading — never skim. Do not summarize
from the first page. For long PDFs, read in ranges until you reach the end. For a URL in
`docs/core-sources/*.url` or a link file, fetch it.

Note the anchors as you go — page numbers, section headings, timestamps, line ranges. You
cannot reconstruct them afterwards, and you must never guess them.

### 2. Write the summary page

`docs/summaries/<source-slug>.md`. The slug is the source's file name minus its extension,
carried over verbatim — including any `YYMMDD` date prefix. Never reformat a date in a file
name; `docs/DOCS.md` fixes file names at `YYMMDD` and dates inside files at `YYYY-MM-DD`.

Front matter per `docs/DOCS.md`, with `sources:` naming the one file:

```yaml
---
type: summary
title: Attention Is All You Need
status: active
claim_type: fact
created: 2026-08-30
updated: 2026-08-30
sources: [docs/core-sources/260415-attention-is-all-you-need.pdf]
---
```

Structure: what it is, its main claims, its evidence, its limits. Wikilink every concept and
entity worth its own page — including ones that do not exist yet.

### 3. Type the claims

A summary reports what its source says, so it is `claim_type: fact` by default. The distinction
matters on the topic and entity pages you touch next, where a source's *"we chose X"* must not
land as *"X is true."*

Mark individual claims in the body with the callouts from `docs/DOCS.md`:

| The source says | Write it as |
|---|---|
| Something is the case | Plain cited prose |
| A human chose something for this project | `> [!check] Decision` — and only if a human actually decided it |
| Something is taken as true but unevidenced | `> [!note] Assumption` |
| Nobody knows yet | `> [!question] Open question` |
| Two sources disagree | `> [!warning] Contradiction` |

**Never set `claim_type: decision` in front matter.** That is a human act, and the only claim
value reserved to one. A source describing a decision is evidence that one was made — record it
as a `[!check] Decision` callout citing the source, and leave the page's front matter as `fact`,
or as `open-question` if you cannot tell whether the decision still stands.

### 4. Cite with real anchors

Every substantive claim gets the most precise anchor the source format actually supports —
page, section, timestamp, or line range, per the provenance table in `docs/DOCS.md`:

```markdown
Throughput dropped 40% under batching ([[docs/core-sources/260415-bench|260415-bench]], p.4).
```

**Falling back to the bare file name is always correct. Inventing an anchor never is.** A page
number you did not see is worse than no page number, because it survives review.

### 5. Update what it touches

For each linked topic and entity:

- **Page exists** → integrate the new claim and add the citation. If it conflicts with what is
  already there, use the contradiction callout from `docs/DOCS.md`, set the page's
  `claim_type: contradiction`, and do not overwrite. Do not decide that the newer source wins
  unless `docs/DOCS.md` contains a rule that says so.
- **Page does not exist** → create it if the source gives you two or more substantive facts.
  One passing mention is a link to a page you leave unwritten; `lint-docs` surfaces it later.
  **Name it for the thing, not the source or the question**: `session-management`, not
  `auth-spec-sessions` or `how-sessions-work`. Prefer the noun the sources themselves use, and
  check `docs/README.md` for an existing page under a near-synonym before inventing a slug —
  two agents naming the same subject differently is the most common way this knowledge base
  grows duplicates.
- Bump `updated` on every page you touch. Leave `status: active` unless the page is genuinely
  incomplete, in which case `draft`.
- **Never change a `status` of `superseded`**, and never edit the
  Decision or Rationale section of a page a human has ruled on. Add your new evidence under
  Evidence and flag it in your report.

### 6. Update the index

`docs/README.md` gets a line for any new page, in the right group.

## After the work set

Append one entry to the top of `docs/CHANGELOG.md`:

```markdown
## 2026-08-30 — scan
- Scanned: docs/core-sources/260415-attention-is-all-you-need.pdf
- Created: docs/summaries/260415-attention-is-all-you-need.md, docs/topics/attention.md
- Updated: docs/entities/transformer.md, docs/README.md
- Flagged: contradiction on parameter count between [[attention]] and [[scaling-laws]]
- Skipped: docs/core-sources/.env.backup (security exclusion)
```

Then report to the user: what came in, what pages moved, and anything that needs their
judgment. Lead with what only a human can settle:

- contradictions you opened, and what each one turns on
- decisions the source describes that you recorded as evidence rather than as project decisions
- pages you left as `open-question` because you could not classify them
- low-confidence claims, and sources you could not fully read

Contradictions and low-confidence claims go in the report, not just in the files.

## Rules

- Never edit, rename, or delete anything under `docs/core-sources/`.
- Never read a path excluded by the security section of `docs/DOCS.md`.
- Never assert something the source does not support. `confidence: low` is a valid answer.
- Never invent provenance — no unseen page numbers, no unverified line ranges, no guessed
  commit hashes.
- Never resolve a contradiction, and never overwrite a human ruling.
- Prefer touching many pages lightly over rewriting one page heavily.
