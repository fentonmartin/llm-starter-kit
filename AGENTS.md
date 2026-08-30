# AGENTS.md

Instructions for any coding agent working with the documentation in this repository.

**Read [`docs/DOCS.md`](docs/DOCS.md) before writing anything in `docs/`.** It is the schema —
page types, file naming, front matter, link style, citations, and this project's own rules —
and it overrides everything below.

`docs/` is a knowledge base, not an application. There is no build and nothing to run here.

## Hard rules

1. **Never create, edit, rename, or delete anything under `docs/sources/`.** It is human-owned
   and immutable to you. Everything else in `docs/` is yours to maintain. If a source is wrong,
   say so in a page; do not touch the file.
2. **Every claim in a page traces to a source or an explicit human instruction.** Do not fill
   gaps with background knowledge. `confidence: low` is a valid answer; inventing is not.
3. **Never resolve a contradiction by overwriting.** Record both claims using the callout
   format in `docs/DOCS.md`. Two sources disagreeing is a finding, not a bug.
4. **`docs/CHANGELOG.md` is append-only.** Add at the top; never rewrite or delete an entry.
5. **`docs/README.md` lists every generated page exactly once.** Update it in the same pass
   that creates or removes a page.
6. **Dates in file names are `YYMMDD`. Dates inside files are `YYYY-MM-DD`.** Slugs are
   lowercase kebab-case. A summary page carries its source's file name, minus the extension.

## Operations

Full instructions live in `.claude/skills/<name>/SKILL.md`. They are plain markdown — read the
relevant one before starting, whatever agent you are.

| Operation | Run it when |
|---|---|
| `init-docs` | Setting the knowledge base up in a project for the first time. Once only. |
| `scan-docs` | New or changed files in `docs/sources/` need folding into the pages. |
| `ask-docs` | A question needs answering from the pages, with citations. |
| `lint-docs` | Health-checking for orphans, gaps, contradictions, stale pages, schema violations. |

If a request does not clearly match one of the four, ask which is meant rather than improvising
a fifth.
