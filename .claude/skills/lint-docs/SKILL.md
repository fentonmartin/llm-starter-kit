---
name: lint-docs
description: Health-check docs/ for orphans, gaps, contradictions, stale pages, duplicates, broken citations, and schema violations, then fix the mechanical ones. Use when the user says lint, audit the docs, check the knowledge base, or asks what is broken or missing.
---

# Lint

A knowledge base rots quietly. This is the sweep that catches it.

Read `docs/DOCS.md` **Part 1** first — it defines every rule you are checking against, and it is
complete on its own. Part 2 is examples and rationale, read on demand.

**The declarations.** Part 1 of `docs/DOCS.md` opens with three paths. Every path in this skill is
written using their defaults; read them as whatever Part 1 declares:

| Written here | Declared as |
|---|---|
| `docs/…` | **Root** — `docs/` unless the base was scaffolded beside a published docs site. |
| `docs/core-sources/` | **Source folder** — one folder: `docs/core-sources/`, a top-level `sources/`, or wherever the material already lives, including outside the root. Immutable to you wherever it is. Part 1 may also list *read-in-place sources*, which you cite but never file. |
| `docs/INDEX.md` | **Index** — `docs/INDEX.md` or `docs/README.md`. |

If the user passed a root as an argument, read `<root>/DOCS.md` and use that base only.

**Page layers may have subfolders.** `docs/topics/auth/session-management.md` is a topic page like
any other: the layer folder decides what a page is, however deep it sits. Slugs are unique across a
whole layer, so a wikilink resolves by slug and does not care about the path. The layers, their
names, and their rules never change with any declaration.

**Commits.** Part 1 also declares whether you commit your own work — `none` by default, meaning
you write files and stop. At `per-run` or `per-file`, stage only the paths this run wrote, never
`git add -A`, and never push. Never commit when a security exclusion fired or a conflict is
unresolved. Committing never replaces reporting.

**Address.** Part 1 declares what to call the owner. Use it when you speak to them directly —
opening a report, asking for a ruling, flagging something that needs them — and not in every
sentence. If it reads `(not set)`, say *"you"* and never guess a name from git history, a commit
author, or an email address. It never appears inside a page.

## Scope

"Pages" means files in `docs/summaries/`, `docs/topics/`, and `docs/entities/`.

**Every check below skips the source folder.** It is raw material, not pages — exempt from front
matter, slugs, wikilinks, and page types. Only checks 13 and 14 look inside it, and only the
`YYMMDD` file-name rule applies there. Read-in-place paths are skipped by every check including
13 and 14; they are cited, not governed.

**Also skip every file at the top of the root**, whatever it is called: `DOCS.md`, the declared
index (`INDEX.md` or `README.md`), `CHANGELOG.md`, and an `AGENTS.md` if one sits there. None of
them is a page. `DOCS.md` in particular contains **example** wikilinks — `[[vector-database]]`,
`[[tokens]]` — and sweeping it reports those examples as gaps in every single run, which is how a
gap list stops being read.

## Severity

Every finding carries one:

| Level | Means | Example |
|---|---|---|
| **ERROR** | Mechanically certain and factually wrong. No judgment involved. | Unparseable front matter, citation to a file that does not exist, `superseded` without `superseded_by`. |
| **WARNING** | Certain, but tolerable, or a schema field that post-dates the page. | Missing `status`, an orphan page, a stale page. |
| **INFO** | Worth knowing, needs the owner. | An open contradiction, a gap with five inbound links, a possible duplicate. |

ERROR is reserved for things that are true regardless of who reads the file. **An open
contradiction is never an ERROR** — it is the system working. If this project gates a merge on
lint, gate on ERROR only.

## Checks

| # | Check | Severity | How |
|---|---|---|---|
| 1 | **Malformed front matter** | ERROR | Front matter that does not parse: unclosed `---`, tabs used for indentation, unquoted `:` in a value, a broken list. See below — this check must not crash you. |
| 2 | **Missing required fields** | ERROR / WARNING | `type`, `title`, `updated`, `sources` missing → ERROR. `status` or `claim_type` missing → WARNING (the page predates them). |
| 3 | **Invalid field values** | ERROR | A `type`, `status`, or `claim_type` outside the sets in `docs/DOCS.md`. `status: superseded` with no `superseded_by`, or pointing at a file that does not exist. |
| 4 | **Agent-set owner-only values** | WARNING | `claim_type: decision` or `status: superseded` on a page with no ruling recorded and no sign of one in `docs/CHANGELOG.md` or git history. |
| 5 | **Broken citations** | ERROR | A citation naming a source path that does not exist. Also a code citation whose line range now runs past the end of the file. |
| 6 | **Uncited claims** | INFO | Substantive claims in topic and entity pages with no citation. Definitions and connective prose are exempt. |
| 7 | **Schema and naming** | WARNING | Wrong page type for its layer, non-kebab-case slugs or subfolder names, summaries with more or fewer than one `sources:` entry, nested YAML where `docs/DOCS.md` requires flat. Any date in a file name must be `YYMMDD`, and a summary's date prefix must match its source's. Subfolders are fine at any depth; the layer folder decides the type. |
| 8 | **Staleness** | WARNING | Summary pages whose source file is newer than the page's `updated`. Topic and entity pages whose `updated` predates a source they cite. |
| 9 | **Orphans** | WARNING | Pages absent from the index, or present in it but with no inbound wikilink from any other page. |
| 10 | **Gaps** | INFO | Wikilinks pointing at pages that do not exist. Group by how many pages link to each — a target with five inbound links is a real hole. |
| 11 | **Contradictions** | INFO | Existing contradiction callouts still open. Plus new ones: claims across pages that conflict on the same fact. |
| 12 | **Duplicates** | INFO | Pages with heavily overlapping titles or subject matter. |
| 13 | **Unread sources** | WARNING | Files in the declared source folder with no summary page. Read-in-place paths are skipped — they produce no summaries by design. |
| 14 | **Excluded material** | ERROR | Anything in the declared source folder matching the security exclusions in `docs/DOCS.md`, and any page that quotes from one. |
| 15 | **Stale layout paths** | WARNING | A `docs/sources/` folder, or `sources:` values and citations pointing at `docs/sources/` — the base predates the 2.0 rename. Also any `sources:` value or citation that does not match the root and source folder declared in `docs/DOCS.md` Part 1, which means one of them moved and the paths have not caught up. See below. |
| 16 | **Declarations** | ERROR | A declared source location that does not exist, more than one declared, or one overlapping a page layer. Also an index declaration naming a file that does not exist, or both `README.md` and `INDEX.md` present in the base and serving as indexes. A missing `## Commits` section, or one whose value is not `none`, `per-run` or `per-file`. A missing `## Who you are working with` section. See below. |
| 17 | **Slug collisions** | ERROR | Two pages in the same layer with the same slug in different subfolders. `[[slug]]` resolves to whichever was seen first, so every inbound link to one of them is silently wrong. Report both paths; the fix is a rename or a merge, and it is the owner's decision. |

### Check 1 in detail — do not crash on bad YAML

Malformed front matter is common; everyone breaks it sooner or later. Handle it as a finding, not
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
         sources: [docs/core-sources/260415-auth-spec.pdf]
         ---

       Not checked: this page was skipped by checks 2-15.
```

### Check 15 in detail — a 1.x base, or a moved path

Bases built on `1.x` keep their material in `docs/sources/`. Nothing about them is wrong; the
folder was renamed in `2.0`. Report it once, at the top, and offer the migration:

```
WARNING  legacy layout
         docs/sources/ exists (14 files). 2.0 renamed it to docs/core-sources/.

         Migrate:
           git mv docs/sources docs/core-sources

         Then re-run /lint-docs and I will rewrite the 31 `sources:` values and
         citations that still point at the old path.
```

**Do the folder move only when the user asks** — it is a rename in their repository, and
`docs/sources/` belongs to the owner regardless of what it is called. Rewriting the *paths inside
pages* after the folder has moved is mechanical, and belongs in the fix-silently list below.

If both `docs/sources/` and `docs/core-sources/` exist, do not merge them. Report it and stop:
a half-finished migration needs the owner to say which file wins.

**The same check catches a moved root or source folder.** If Part 1 declares a root other than
`docs/` — `docs/kb/`, say — or a source folder other than `docs/core-sources/`, then `sources:`
values and citations still written the old way are stale in exactly the same way, and rewriting
them to the declared paths is equally mechanical. Rewrite them and say how many. Do not move any
folder to make the paths true; the declaration wins, and if the user declared it by mistake that
is a one-line fix in `DOCS.md`, not a repository move.

### Check 16 in detail — the declarations

Worth an ERROR because everything downstream is silently wrong when one is broken, and nothing
else notices.

**The source location:**

- **It does not exist.** Usually a typo, sometimes a folder moved without updating Part 1. Report
  the declared path and what you found near it; do not create the folder and do not guess which
  one was meant.
- **More than one declared.** Report both and stop. One summary page per source file is the
  backbone of provenance, and you cannot maintain it across two locations once a file name
  repeats. The fix is the owner's decision: file everything into one, or move the extra to
  *read-in-place sources*.
- **It contains a page layer**, or is a page layer, or is the root itself. This makes the agent's
  own output look like source material. Report it and stop — a scan against this declaration will
  start summarizing summaries.
- **It is not a folder** — a glob, a single file, or the repository root. There is no glob form:
  the declaration names one folder. Report what it names and stop.

**The index:**

- **The declared index does not exist.** Report it; do not create one silently, because writing a
  fresh index over a base whose real index is at the other name loses the grouping someone chose.
- **Both `README.md` and `INDEX.md` exist and both list pages.** Report both and stop. Two indexes
  drift, and an agent reading the wrong one silently cannot see half the base. If one is a genuine
  readme — prose for people, not a page list — that is fine and not a finding; say which you
  judged to be which.

**The commit setting and the address:**

- **`## Commits` is missing, or its value is not one of the three.** Report it and treat the base
  as `none` for this run — the safe reading, since `none` writes nothing to git. A base upgraded
  from `2.0` will hit this until the section is added, so say that rather than implying damage.
- **`## Who you are working with` is missing.** Report it; treat the address as `(not set)` and
  say *"you"*. Never fill it in from git history — the owner is asked, never inferred.
- **`Address:` holds something that is not a form of address** — a full email, a URL, a sentence.
  Report it and use *"you"* until it is corrected. Do not guess at what part of it is the name.

Never fix any of these silently. Each means the contract says something its author did not intend,
and writing to the base before that is settled makes it worse. The two above are the exception
only in that a missing section has a safe default to fall back on while you report it.

### Check 17 in detail — slug collisions

Only possible once a layer uses subfolders, and worth an ERROR because the damage is invisible.
Wikilinks resolve by slug, so `topics/auth/tokens.md` and `topics/billing/tokens.md` make every
`[[tokens]]` in the base point at one of them arbitrarily — and the other's inbound links are
wrong without any broken-link symptom.

Report both paths and their titles, and say how many inbound links each has. Do not rename either:
two pages that want the same slug are usually one page that got written twice, and merging is a
the owner's call. Compare their `sources:` first — the same sources on both is strong evidence of a
duplicate rather than two things needing better names.

## What to fix, what to report

Fix without asking — mechanical, reversible, no judgment:

- Add missing pages to the index; remove index lines pointing at deleted files.
- Add `status: active` and a defaulted `claim_type` to pages that predate those fields
  (`fact` on a summary, `open-question` on a topic or entity you cannot classify). **Never
  default to `decision`.**
- Lowercase field values that are valid but miscased, and fix slug casing.
- Set `status: stale` on pages check 8 found stale.
- Correct wikilinks broken by a rename you can confirm from `docs/CHANGELOG.md`.
- Rewrite `docs/sources/` to `docs/core-sources/` in `sources:` values and citations — but only
  once the folder itself has actually moved. Rewriting paths that still point at real files
  breaks every citation in the base.

Report, do not fix — anything requiring judgment:

- Unparseable front matter (check 1). Propose the block; do not write it.
- Contradictions, duplicates worth merging, stale pages needing re-scan, gaps worth writing,
  uncited claims. For each, say what you would do and wait.
- Broken citations. A citation pointing at a missing file may mean the source moved, or that
  the claim was never sourced. Those need opposite fixes, and only the owner can tell which.
- Anything under check 4 or check 14. A page claiming authority it may not have, and a
  secret that reached the knowledge base, are both escalations.
- Any file name with a malformed date. Renaming a page breaks inbound links, and files under
  `docs/core-sources/` are immutable to you regardless — report the bad name and the corrected
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
- Errors: 1 broken citation (docs/topics/auth.md → docs/core-sources/260101-old-spec.pdf, missing),
  1 malformed front matter (docs/topics/authentication.md)
- Open: 2 contradictions, 3 gaps (most-linked: [[retrieval-benchmark]], 5 inbound), 1 stale summary
```

If everything is clean, say so in one line. Do not manufacture findings.

## Rules

- Never edit, rename, or delete anything under the declared source folder, or any read-in-place
  path. Both belong to the owner; the declaration does not change that, it is what establishes it.
- Never resolve a contradiction. Surfacing it is the whole point of the check.
- Never rewrite front matter you could not parse.
- Never promote a page to `claim_type: decision` or `status: superseded` to make a finding go
  away.
