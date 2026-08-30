---
name: scan-docs
description: Read new or changed files in docs/sources/ and fold them into the rest of docs/ — writing summary pages, updating the topic and entity pages they touch, flagging contradictions, and appending to the changelog. Use when the user says scan, process my sources, add this article/paper/transcript to the knowledge base, or drops a new file into docs/sources/.
---

# Scan

Turn raw material in `docs/sources/` into linked pages in `docs/`. This is the only operation that
creates summary pages.

## Before you start

1. Read `docs/DOCS.md`. It overrides anything here — page types, slugs, front matter, link style,
   and any project-specific rules at the bottom.
2. Read `docs/README.md` to learn what already exists. Never create a page without checking
   the index for one that covers the same ground.
3. Determine the work set:
   - Explicit file(s) named by the user, or
   - Everything in `docs/sources/` with no corresponding `docs/summaries/<slug>.md`, or
   - Files whose mtime is newer than their summary's `updated` field.

If the work set is empty, say so and stop. Do not invent work.

## Per source

Work one source at a time, all the way through, before starting the next. A half-scanned
source is worse than an un-scanned one.

1. **Read it fully.** "Scan" is the name of the command, not the depth of the reading — never
   skim. Do not summarize from the first page. For long PDFs, read in ranges until you reach
   the end. For a URL in `docs/sources/*.url` or a link file, fetch it.
2. **Write `docs/summaries/<source-slug>.md`.** The slug is the source's file name minus its
   extension, carried over verbatim — including any `YYMMDD` date prefix. Never reformat a
   date in a file name; `docs/DOCS.md` fixes the format at `YYMMDD` and dates inside files at
   `YYYY-MM-DD`. Front matter per `docs/DOCS.md`, `sources:` naming the one file. Structure:
   what it is, its main claims, its evidence, its limits. Wikilink every concept and entity
   worth its own page — including ones that do not exist yet.
3. **Update what it touches.** For each linked topic and entity:
   - Page exists → integrate the new claim. Add the citation. If it conflicts with what is
     already there, use the contradiction callout from `docs/DOCS.md`; do not overwrite.
   - Page does not exist → create it if the source gives you two or more substantive facts.
     One passing mention is a link to a page you leave unwritten. `lint-docs` will surface it later.
4. **Update `docs/README.md`** with any new pages, in the right group, one line each.

## After the work set

Append one entry to the top of `docs/CHANGELOG.md`:

```markdown
## 2026-08-30 — scan
- Scanned: docs/sources/260415-attention-is-all-you-need.pdf
- Created: docs/summaries/260415-attention-is-all-you-need.md, docs/topics/attention.md
- Updated: docs/entities/transformer.md, docs/README.md
- Flagged: contradiction on parameter count between [[attention]] and [[scaling-laws]]
```

Then report to the user: what came in, what pages moved, and anything that needs their
judgment. Contradictions and low-confidence claims go in the report, not just the files.

## Rules

- Never edit or delete anything under `docs/sources/`.
- Never assert something the source does not support. `confidence: low` is a valid answer.
- Prefer touching many pages lightly over rewriting one page heavily.
