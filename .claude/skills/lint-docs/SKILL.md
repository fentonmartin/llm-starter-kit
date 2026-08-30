---
name: lint-docs
description: Health-check docs/ for orphans, gaps, contradictions, stale pages, duplicates, broken citations, and schema violations, then fix the mechanical ones. Use when the user says lint, audit the docs, check the knowledge base, or asks what is broken or missing.
---

# Lint

A knowledge base rots quietly. This is the sweep that catches it.

Read `docs/DOCS.md` **Part 1** first — it defines every rule you are checking against, and it is
complete on its own. Part 2 is examples and rationale, read on demand.

## Scope

"Pages" means files in `docs/summaries/`, `docs/topics/`, and `docs/entities/`.

**Every check below skips `docs/sources/`.** It is raw material, not pages — exempt from front
matter, slugs, wikilinks, and page types. Only checks 13 and 14 look inside it, and only the
`YYMMDD` file-name rule applies there. Also skip the files at the top of `docs/`: `DOCS.md`,
`AGENTS.md`, `README.md`, `CHANGELOG.md`.

## Severity

Every finding carries one:

| Level | Means | Example |
|---|---|---|
| **ERROR** | Mechanically certain and factually wrong. No judgment involved. | Unparseable front matter, citation to a file that does not exist, `superseded` without `superseded_by`. |
| **WARNING** | Certain, but tolerable, or a schema field that post-dates the page. | Missing `status`, an orphan page, a stale page. |
| **INFO** | Worth knowing, needs a human. | An open contradiction, a gap with five inbound links, a possible duplicate. |

ERROR is reserved for things that are true regardless of who reads the file. **An open
contradiction is never an ERROR** — it is the system working. If this project gates a merge on
lint, gate on ERROR only.

## Checks

| # | Check | Severity | How |
|---|---|---|---|
| 1 | **Malformed front matter** | ERROR | Front matter that does not parse: unclosed `---`, tabs used for indentation, unquoted `:` in a value, a broken list. See below — this check must not crash you. |
| 2 | **Missing required fields** | ERROR / WARNING | `type`, `title`, `updated`, `sources` missing → ERROR. `status` or `claim_type` missing → WARNING (the page predates them). |
| 3 | **Invalid field values** | ERROR | A `type`, `status`, or `claim_type` outside the sets in `docs/DOCS.md`. `status: superseded` with no `superseded_by`, or pointing at a file that does not exist. |
| 4 | **Agent-set human values** | WARNING | `claim_type: decision` or `instruction`, or `status: superseded`/`deprecated`/`archived`, on a page with no human ruling recorded and no sign of one in `docs/CHANGELOG.md` or git history. |
| 5 | **Broken citations** | ERROR | A citation naming a source path that does not exist. Also a code citation whose line range now runs past the end of the file. |
| 6 | **Uncited claims** | INFO | Substantive claims in topic and entity pages with no citation. Definitions and connective prose are exempt. |
| 7 | **Schema and naming** | WARNING | Wrong page type for its folder, non-kebab-case slugs, summaries with more or fewer than one `sources:` entry, nested YAML where `docs/DOCS.md` requires flat. Any date in a file name must be `YYMMDD`, and a summary's date prefix must match its source's. |
| 8 | **Staleness** | WARNING | Summary pages whose source file is newer than the page's `updated`. Topic and entity pages whose `updated` predates a source they cite. |
| 9 | **Orphans** | WARNING | Pages absent from `docs/README.md`, or present in the index but with no inbound wikilink from any other page. |
| 10 | **Gaps** | INFO | Wikilinks pointing at pages that do not exist. Group by how many pages link to each — a target with five inbound links is a real hole. |
| 11 | **Contradictions** | INFO | Existing contradiction callouts still open. Plus new ones: claims across pages that conflict on the same fact. |
| 12 | **Duplicates** | INFO | Pages with heavily overlapping titles or subject matter. |
| 13 | **Unread sources** | WARNING | Files in `docs/sources/` with no summary page. |
| 14 | **Excluded material** | ERROR | Anything in `docs/sources/` matching the security exclusions in `docs/DOCS.md`, and any page that quotes from one. |

### Check 1 in detail — do not crash on bad YAML

Malformed front matter is common: agents and humans both break it. Handle it as a finding, not
as a failure.

```markdown
---
status: ACTIVE
claim_type: FACT
missing_closing_dashes_or_invalid_yaml
# Document body starts here improperly
```

Rules for this check:

- **Never abort the run.** One unparseable file is one ERROR; the other checks continue over
  every other page.
- **Never ingest partial metadata.** If the block does not parse, the page has no usable front
  matter at all. Do not salvage the two lines that happened to look like YAML — a page treated
  as half-valid is worse than one reported as broken.
- **Never guess the fix.** Report what is wrong and what the corrected block should be; do not
  silently rewrite a block you could not parse. The body may have been swallowed by it.
- Case matters. `status: ACTIVE` is invalid — the values in `docs/DOCS.md` are lowercase. That
  is a check 3 ERROR on a page that parses, and mentioning it is part of the fix you propose.

Report it with the fix spelled out:

```
ERROR  docs/topics/authentication.md
       Front matter does not parse: no closing `---` before the body.
       Line 4 `missing_closing_dashes_or_invalid_yaml` is not a key/value pair.

       Fix — replace lines 1-4 with:
         ---
         type: topic
         title: Authentication
         status: active
         claim_type: fact
         updated: 2026-08-30
         sources: [docs/sources/260415-auth-spec.pdf]
         ---

       Not checked: this page was skipped by checks 2-14.
```

## What to fix, what to report

Fix without asking — mechanical, reversible, no judgment:

- Add missing pages to `docs/README.md`; remove index lines pointing at deleted files.
- Add `status: active` and a defaulted `claim_type` to pages that predate those fields
  (`fact` on a summary, `open-question` on a topic or entity you cannot classify). **Never
  default to `decision` or `instruction`.**
- Lowercase field values that are valid but miscased, and fix slug casing.
- Set `status: stale` on pages check 8 found stale.
- Correct wikilinks broken by a rename you can confirm from `docs/CHANGELOG.md`.

Report, do not fix — anything requiring judgment:

- Unparseable front matter (check 1). Propose the block; do not write it.
- Contradictions, duplicates worth merging, stale pages needing re-scan, gaps worth writing,
  uncited claims. For each, say what you would do and wait.
- Broken citations. A citation pointing at a missing file may mean the source moved, or that
  the claim was never sourced. Those need opposite fixes, and only a human can tell which.
- Anything under check 4 or check 14. A page claiming human authority it may not have, and a
  secret that reached the knowledge base, are both escalations.
- Any file name with a malformed date. Renaming a page breaks inbound links, and files under
  `docs/sources/` are immutable to you regardless — report the bad name and the corrected
  `YYMMDD` form you propose.

## Output

Report grouped by severity, ERROR first, then by check. Give counts and file paths, and make
every ERROR actionable — what is wrong, and the exact edit that fixes it.

```
ERROR    2   broken citation (1), malformed front matter (1)
WARNING  6   missing status (4), stale (1), orphan (1)
INFO     5   contradictions (2), gaps (3)
```

Then append to `docs/CHANGELOG.md`:

```markdown
## 2026-08-30 — lint
- Fixed: 3 index omissions, 4 pages defaulted to status: active
- Errors: 1 broken citation (docs/topics/auth.md → docs/sources/260101-old-spec.pdf, missing),
  1 malformed front matter (docs/topics/authentication.md)
- Open: 2 contradictions, 3 gaps (most-linked: [[retrieval-eval]], 5 inbound), 1 stale summary
```

If everything is clean, say so in one line. Do not manufacture findings.

## Rules

- Never edit, rename, or delete anything under `docs/sources/`.
- Never resolve a contradiction. Surfacing it is the whole point of the check.
- Never rewrite front matter you could not parse.
- Never promote a page to `claim_type: decision` or `instruction`, or to a human-only `status`,
  to make a finding go away.
