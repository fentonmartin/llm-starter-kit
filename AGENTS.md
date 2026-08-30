# AGENTS.md

Instructions for any coding agent working with the documentation in this repository.

**Read [`docs/DOCS.md`](docs/DOCS.md) before writing anything in `docs/`.** It is the governance
contract — page types, claim types, lifecycle, file naming, front matter, provenance, citations,
retrieval limits, security exclusions, and this project's own rules — and it overrides
everything below.

`docs/` is a knowledge base, not an application. There is no build and nothing to run here.

## The one idea

The repository is the durable knowledge layer. You are not.

Models change, sessions end, chat history is discarded. The Markdown in `docs/` is what
survives, so it has to be trustworthy without you in the room: every claim traceable, every
disagreement visible, every stale page marked as stale.

That is why the rules below constrain what you may assert rather than what you may write.

## Hard rules

1. **Never create, edit, rename, or delete anything under `docs/sources/`.** It is human-owned
   and immutable to you. Everything else in `docs/` is yours to maintain. If a source is wrong,
   say so in a page; do not touch the file.
2. **Every claim in a page traces to a source or an explicit human instruction.** Do not fill
   gaps with background knowledge. `confidence: low` is a valid answer; inventing is not.
3. **Never resolve a contradiction by overwriting.** Record both claims using the callout
   format in `docs/DOCS.md`. Two sources disagreeing is a finding, not a bug.
4. **You propose; humans decide.** You may discover, summarize, classify, link, flag, and
   suggest. You may not establish a project decision. Never set `claim_type: decision` or
   `claim_type: instruction`, and never set `status: superseded`, `deprecated`, or `archived` —
   those six values are human acts. If you think a decision was made, write `open-question`
   and ask.
5. **Never fabricate provenance.** Cite only documents you actually opened in this run, and
   only with an anchor you actually saw. A bare file name is always an acceptable fallback; an
   invented page number is not.
6. **Never silently truncate.** If the evidence a question needs exceeds your context, say so
   and ask for a narrower scope. An answer built on a quietly dropped source looks exactly like
   a correct one, which is what makes it the worst outcome available.
7. **Never read the excluded paths.** `.env*`, `*.pem`, `*.key`, `secrets/`, `credentials/`,
   `private/`, `.ssh/`, `.aws/` — see the security section of `docs/DOCS.md` for the full list.
   If a source turns out to contain credentials, stop and tell the user; do not summarize it.
8. **Never auto-resolve a git conflict in `docs/summaries/`, `docs/topics/`, or
   `docs/entities/`.** Git can tell you two people wrote different text. It cannot tell you
   which is true. Surface both sides and stop. `docs/CHANGELOG.md` (keep both, date order) and
   `docs/README.md` (keep the union, re-sort) are the only exceptions.
9. **`docs/CHANGELOG.md` is append-only.** Add at the top; never rewrite or delete an entry.
10. **`docs/README.md` lists every generated page exactly once.** Update it in the same pass
    that creates or removes a page.
11. **Dates in file names are `YYMMDD`. Dates inside files are `YYYY-MM-DD`.** Slugs are
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
| `eval-docs` | Checking whether the knowledge base still answers its known questions correctly. |

If a request does not clearly match one of the five, ask which is meant rather than improvising
a sixth.

## Repository layout

```
AGENTS.md                  this file — the agent entry point
README.md                  human-facing documentation
CHANGELOG.md               this kit's release history
docs/
  DOCS.md                  the governance contract. Read first, always.
  README.md                the index of every generated page
  CHANGELOG.md             append-only log of scan / lint / ask / eval runs
  sources/                 raw material. Human-owned, immutable to you.
  summaries/               one page per source
  topics/                  one page per idea, question, or theme
  entities/                one page per person, org, product, dataset
  evals/questions.yaml     questions the knowledge base must answer correctly
examples/example-project/  a small worked example of the whole loop
.claude/skills/            the five skills, as plain Markdown
.claude/commands/          slash-command aliases
```

## Changing this kit

If you are working on `llm-starter-kit` itself rather than using it:

- **`docs/DOCS.md` at the repo root is the shipped template.** Editing it changes what every
  future `/init-docs` writes. Do not put this repository's own project rules in it.
- **Keep the schema minimal.** Front matter is six required fields and three optional ones.
  Every field added is a field a future agent will forget, reorder, or corrupt. Structure
  belongs in Markdown sections, not in nested YAML.
- **Keep the skills agent-agnostic.** No Claude-specific, OpenAI-specific, or Cursor-specific
  instructions in `SKILL.md` bodies. The repository is the interface; any filesystem-capable
  agent must be able to follow them.
- **Do not add infrastructure.** No vector database, no embeddings service, no API server, no
  web UI, no runtime dependency. Markdown, YAML front matter, git, and the filesystem are the
  whole stack, and that is a design decision rather than a limitation to be fixed.
- **Changes to the schema go in `CHANGELOG.md` with migration guidance**, and stay backward
  compatible unless there is no alternative. Someone's knowledge base is already using the old
  shape.
- Keep `README.md`, `AGENTS.md`, `docs/DOCS.md`, and the skills describing the same behavior.
  Documentation that claims a capability the skills do not implement is the failure mode this
  kit is supposed to prevent.
