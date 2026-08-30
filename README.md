# llm-starter-kit

A drop-in, agent-agnostic documentation system. Install it in any project and it generates a
`docs/` folder that AI agents build, maintain, and read from — so every agent that touches the
repo starts with the same understanding of it.

```
/init-docs      set it up in this project — interviews you, writes the schema
/scan-docs      fold new material into the pages
/ask-docs       answer a question from the pages, with citations
/lint-docs      health-check and repair
```

This is not RAG. Nothing is embedded, chunked, or retrieved at question time. The synthesis
happens once, when material comes in, and the result is a durable artifact you can read, edit,
diff, review in a pull request, and hand to the next agent. Asking a question is reading — not
searching.

The trade-off is honest: scanning costs more time and tokens up front than embedding does.
What you get back survives. A vector index is not something you can read.

---

## The layers

```
docs/sources/  ──scan──►  docs/summaries|topics|entities  ──ask──►  your answer
   (yours)                        (the agent's)
```

| Layer | Path | Written by |
|---|---|---|
| **Sources** | `docs/sources/` | you — append-only, the agent never edits it |
| **Pages** | `docs/summaries/`, `docs/topics/`, `docs/entities/` | the agent, continuously |
| **Schema** | `docs/DOCS.md` | you, whenever the rules should change |

Everything sits under `docs/` so the knowledge base is one self-contained folder you can copy
into any project. `docs/sources/` is nested for filing only — it stays read-only to the agent
and exempt from every page rule.

## The operations

| Command | Does |
|---|---|
| `/init-docs` | Surveys the project, interviews you, scaffolds `docs/`, writes a `DOCS.md` for *this* codebase. Run once. Merges safely if `docs/` already exists. |
| `/scan-docs` | Reads new files in `docs/sources/`, writes a summary page for each, updates every topic and entity page they touch, flags contradictions. |
| `/ask-docs` | Answers a question from the pages with citations. Drops to `docs/sources/` only when the pages fall short, and says when it does. |
| `/lint-docs` | Orphans, gaps, contradictions, stale pages, duplicates, schema violations. Fixes the mechanical ones, reports the rest. |

`/init`, `/scan`, `/ask`, `/lint` work as short aliases. The long names are canonical because
this installs globally — `/lint` alone would collide with code linters and misfire on "lint my
code."

Each is a skill under [.claude/skills/](.claude/skills/), so an agent also picks the right one
when you just describe what you want: "document this repo" runs init-docs.

---

## Install

```
/plugin marketplace add fentonmartin/llm-starter-kit
/plugin install llm-starter-kit
```

Then, from inside the project you want to document:

```
/init-docs
```

No plugin, or a different agent? Copy `docs/` and `.claude/` from this repo into your project,
plus the root `AGENTS.md`. The kit is self-contained; nothing else is needed.

**Commit before you run anything.** Every scan changes many files at once, and `git diff` is
the fastest way to see what the agent actually did.

### What init-docs does

It surveys before it asks — your README, your manifest, your layout, any existing agent config
— then asks five things: what the knowledge base is for, who reads it, what recurring things
deserve a page each (*your* nouns), what must never be got wrong, and what is out of scope.

From that it scaffolds `docs/` and writes `docs/DOCS.md`: the page types and the rules, in your
vocabulary.

**Read the rules it proposes and fix them before going further.** This is the step people skip
and the main way the kit disappoints — a generic schema produces generic pages. Rules that earn
their keep are specific and checkable:

```markdown
- A summary page names the file paths it covers, so lint can tell when the code moved.
- Architectural decisions get a topic page including the alternatives that were rejected.
- Never document intended behavior as actual behavior; if code and comment disagree, that is
  a contradiction, and both go on the page.
- Every claim about a person carries the date it was true.
```

### If the project already has docs/

It merges. It never overwrites and never deletes.

Existing pages are reorganized into the structure: explanations and design notes become topics,
single-subject pages become entities, and specs, exports, and vendor material move to
`docs/sources/`. Anything it cannot confidently classify also goes to `docs/sources/`, where the
next scan turns it into proper pages — nothing is lost by that default.

It shows every move as `old path → new path` before touching anything, then fixes inbound links
across the repo. Generated output, `README.md`, `CONTRIBUTING.md`, and anything your tooling
reads by path stay where they are.

If your `docs/` is built by a site generator — `mkdocs.yml`, `docusaurus.config.js`,
`book.toml` — it will not rearrange it at all. It scaffolds at `docs/kb/` and treats your
published docs as sources.

---

## Using it

**Add material.** Put files in `docs/sources/`, or point the schema at your code. Names are
lowercase kebab-case; genuinely dated files get a `YYMMDD` prefix. For a web page, save the
article as markdown or leave a `.url` file with the link on the first line.

Start with three to five files on one subject. A first scan of forty unrelated files produces
forty disconnected summaries and no synthesis — the value is in the cross-references, and
cross-references need overlap.

**Scan.** `/scan-docs` reads each source fully, writes `docs/summaries/<same-name>.md`, updates
the topic and entity pages it touches, flags contradictions, and updates the index and log.
Then read the diff — `git diff` after a first scan shows you exactly where the agent's judgment
differs from yours, which is what your rules exist to correct.

**Ask.** `/ask-docs` answers from the pages, citing them, and each page cites its source. It
drops back to `docs/sources/` only when the pages are thin — and says so, which is your signal
that something needs re-scanning. Questions that test the structure: *What do my sources
disagree about? What rests on only one source? What would I need to read to answer X?*

**Lint.** `/lint-docs` runs eight checks and fixes the mechanical ones. Pay attention to the
**gaps** section: dangling wikilinks ranked by inbound count are a reading list, generated for
free.

### A rhythm that works

- Add sources as you find them. No need to scan immediately.
- Scan when a few have piled up, and read the diff.
- Ask whenever you would otherwise have gone digging.
- Lint every fifth scan or so.
- Commit after each scan, so you can always roll back.
- Revisit `docs/DOCS.md` monthly. Every rule you add improves every future scan.

The knowledge base compounds. Month one it is a filing cabinet; month six it answers things you
would never have found by searching, because the connection was made when the material came in.

### When things go wrong

| Symptom | Cause | Fix |
|---|---|---|
| Pages are generic and say little | `DOCS.md` still has the shipped defaults, or sources were too few and unrelated | Rewrite the rules; scan more material on one subject |
| Same concept split across two pages | Slugs differed on first mention | Lint reports duplicates; merge, then add a naming rule |
| Agent asserts things no source supports | Rules too loose | Add an explicit rule; re-scan the offending source |
| Pages drifted from their sources | Sources edited after scanning | Lint flags stale pages; re-scan them |
| Summaries are shallow | Agent skimmed a long PDF | Scan that source alone, and say "read it in full" |
| Too many dangling links | Normal and healthy | They are your reading list |

---

## With any coding LLM

Nothing here is Claude-specific — the kit is markdown and a folder convention.

| Tool | How |
|---|---|
| Claude Code | Works as-is, as a plugin or a copied folder. |
| Codex, opencode, Cursor, Windsurf | Read the root [AGENTS.md](AGENTS.md) automatically, which points to [docs/DOCS.md](docs/DOCS.md). Nothing to configure. |
| GitHub Copilot | Copy `AGENTS.md` to `.github/copilot-instructions.md`. |
| Gemini CLI | Copy `AGENTS.md` to `GEMINI.md`. |
| Anything else | Paste the bootstrap prompt below. |

The skills in [.claude/skills/](.claude/skills/) are plain markdown instruction files. Any agent
that can read a file can follow them.

<details>
<summary><b>Bootstrap prompt</b> — for an agent with no repo awareness</summary>

```
This project is a knowledge base with three layers:

- docs/sources/ — raw material. Read it. NEVER create, edit, rename, or delete anything
                  in it. It is exempt from all page rules.
- docs/         — everything else here is markdown pages you write and maintain. Each must
                  trace back to a source or to something I explicitly told you.
- docs/DOCS.md  — the schema. Read it fully before writing anything. It overrides these
                  instructions.

Four operations, fully specified in .claude/skills/<name>/SKILL.md — read the relevant file
before you begin:

- init-docs → .claude/skills/init-docs/SKILL.md   (set up, once per project)
- scan-docs → .claude/skills/scan-docs/SKILL.md
- ask-docs  → .claude/skills/ask-docs/SKILL.md
- lint-docs → .claude/skills/lint-docs/SKILL.md

Non-negotiable:
- Never resolve a contradiction by overwriting. Record both claims.
- docs/CHANGELOG.md is append-only, newest first.
- docs/README.md lists every generated page exactly once. Keep it current.
- Dates in file names are YYMMDD. Dates inside files are YYYY-MM-DD.

Start with: init-docs, scan-docs, ask-docs QUESTION, or lint-docs
```

In a chat window without file access, upload `docs/DOCS.md` and the one skill file you need,
and paste the results back into the repo yourself. Clumsy, but the format holds.

</details>

---

## Layout

```
README.md                what this is, and how to use it
AGENTS.md                agent entry point — the rules, and a pointer to the schema
LICENSE
docs/
  DOCS.md                the schema — page types, front matter, links, citations, your rules
  README.md              the index. every generated page, once.
  CHANGELOG.md           append-only log of every run.
  sources/               raw material. yours. read-only to the agent.
  summaries/  topics/  entities/
.claude/skills/          init-docs, scan-docs, ask-docs, lint-docs
.claude/commands/        the four, plus short aliases
.claude-plugin/          install this repo as a Claude Code plugin
```

The `docs/` folder here is the template. `/init-docs` reproduces it inside your project.

## Design notes

**Why "docs" and not "wiki".** A wiki is a destination you visit. Docs sit in the repo next to
the thing they describe, get reviewed in pull requests, and are the natural target when you tell
an agent *"build docs for this project."*

**Why the agent never writes to `docs/sources/`.** One-way flow means you can always regenerate
the pages from scratch and compare. If the agent could edit its own inputs, that guarantee is
gone.

**Why contradictions are recorded, not resolved.** Overwriting a conflicting claim destroys the
most valuable signal the knowledge base produces. Two sources disagreeing is a finding.

**Why the changelog is append-only.** It is the audit trail for everything the agent did, and
what `lint-docs` uses to distinguish a rename from a deletion.

**Why linking to pages that do not exist is encouraged.** A dangling wikilink is a cheap,
in-context to-do. `lint-docs` ranks them by inbound link count, so the gaps that matter most
surface first.

**Why one schema file instead of per-agent config.** `docs/DOCS.md` survives switching agents
and can be reviewed like any other spec. `AGENTS.md` points at it rather than restating it.

## Sources & inspiration

- [Andrej Karpathy — *LLM Wiki*](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
  — the three-layer pattern (raw sources / wiki / schema) and the ingest–query–lint loop this
  kit is built on, here renamed scan–ask–lint. His `index.md` and `log.md` became
  `docs/README.md` and `docs/CHANGELOG.md`.
- [kepano/obsidian-skills](https://github.com/kepano/obsidian-skills) — the Agent Skills
  packaging model (`skills/<name>/SKILL.md`, `.claude-plugin/`) and the Obsidian Flavored
  Markdown conventions: wikilinks, callouts, YAML properties.
- [Agent Skills specification](https://code.claude.com/docs/en/skills) — the skill format.
- [Obsidian Flavored Markdown](https://help.obsidian.md/syntax) — link, embed, and callout syntax.

## What is different here

- Built to be installed into an existing project, with `/init-docs` writing a schema from an
  interview rather than shipping a template you are expected to edit later.
- Merges with documentation that already exists instead of demanding a clean slate.
- One vocabulary — `docs/`, `docs/sources/`, `docs/DOCS.md` — so it reads naturally pointed at
  a codebase, not only at a reading pile.
- Explicit page types with required front matter, giving `lint-docs` something concrete to check
  rather than vibes.
- A stated split in `lint-docs` between what the agent fixes silently and what it must escalate.
- Agent-agnostic by construction: plain-markdown skills and one root `AGENTS.md`.
- Obsidian-compatible out of the box: point a vault at `docs/` and the graph works.

## License

MIT
