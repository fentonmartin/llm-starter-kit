---
name: lint-docs
description: Health-check docs/ for orphans, gaps, contradictions, stale pages, duplicates, and schema violations, then fix the mechanical ones. Use when the user says lint, audit the docs, check the knowledge base, or asks what is broken or missing.
---

# Lint

A knowledge base rots quietly. This is the sweep that catches it.

Read `docs/DOCS.md` first — it defines every rule you are checking against.

## Scope

"Pages" means files in `docs/summaries/`, `docs/topics/`, and `docs/entities/`.

**Every check below skips `docs/sources/`.** It is raw material, not pages — exempt from front
matter, slugs, wikilinks, and page types. Only check 8 looks inside it, and only the `YYMMDD`
file-name rule applies there. Also skip the files at the top of `docs/`: `DOCS.md`,
`AGENTS.md`, `README.md`, `CHANGELOG.md`.

## Checks

| # | Check | How |
|---|---|---|
| 1 | **Orphans** | Pages absent from `docs/README.md`, or present in the index but with no inbound wikilink from any other page. |
| 2 | **Gaps** | Wikilinks pointing at pages that do not exist. Group by how many pages link to each — a target with five inbound links is a real hole. |
| 3 | **Contradictions** | Existing contradiction callouts still open. Plus new ones: claims across pages that conflict on the same fact. |
| 4 | **Staleness** | Summary pages whose source file is newer than the page's `updated` field. Topic pages whose `updated` predates a source they cite. |
| 5 | **Duplicates** | Pages with heavily overlapping titles or subject matter. |
| 6 | **Schema** | Missing or malformed front matter, wrong page type for its folder, non-kebab-case slugs, summaries with more or fewer than one `sources:` entry. Also date format: any date in a file name must be `YYMMDD`, and a summary's date prefix must match its source's. |
| 7 | **Uncited** | Substantive claims in topic and entity pages with no citation. |
| 8 | **Unread sources** | Files in `docs/sources/` with no summary page. |

## What to fix, what to report

Fix without asking — mechanical, reversible, no judgment:

- Add missing pages to `docs/README.md`; remove index lines pointing at deleted files.
- Repair front matter fields and slug casing.
- Correct wikilinks broken by a rename you can confirm from `docs/CHANGELOG.md`.

Report, do not fix — anything requiring judgment:

- Contradictions, duplicates worth merging, stale pages needing re-scan, gaps worth
  writing, uncited claims. For each, say what you would do and wait.
- Any file name with a malformed date. Renaming a page breaks inbound links, and files under
  `docs/sources/` are immutable to you regardless — report the bad name and the corrected `YYMMDD`
  form you propose.

## Output

Report grouped by check, most consequential first, with counts and file paths. Then append to
`docs/CHANGELOG.md`:

```markdown
## 2026-08-30 — lint
- Fixed: 3 index omissions, 1 malformed front matter
- Open: 2 contradictions, 4 gaps (most-linked: [[retrieval-eval]], 5 inbound), 1 stale summary
```

If everything is clean, say so in one line. Do not manufacture findings.
