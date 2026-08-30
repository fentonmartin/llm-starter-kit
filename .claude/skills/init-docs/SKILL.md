---
name: init-docs
description: Set up the knowledge base in a project — interview the user about what they are documenting, scaffold docs/, and write a DOCS.md schema tailored to this project. Handles a docs/ folder that already exists by merging and reorganizing rather than overwriting. Use when the user says init docs, set up the knowledge base, install this kit here, build docs for this project, or start documenting this repo.
---

# Init docs

The front door. Run once per project. It produces a `docs/` folder and, more importantly, a
`docs/DOCS.md` written for *this* project rather than a generic template.

**A generic `DOCS.md` is the main way this kit fails.** The interview is not a formality — it
is where the value is created. Do not skip it, and do not answer the questions yourself.

## 1. Survey before asking

Look before you interview, so your questions are specific and you never ask what you can see.

- What kind of project is this? Read `README.md`, and the manifest if there is one
  (`package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `composer.json`).
- What is the top-level layout? Where does the real code live?
- Does `docs/` already exist? If so, list what is in it and read enough to classify it.
- Is there existing agent config — `CLAUDE.md`, `AGENTS.md`, `.cursorrules`, `.github/copilot-instructions.md`?

Say what you found in one short paragraph before your first question. It shows the user you
looked and it lets them correct you early.

## 2. Interview

Ask these. Batch them — do not drip one question at a time. Accept short answers and move on;
you can propose sensible defaults and let the user correct them.

1. **What is this knowledge base for?** Documenting this codebase, collecting outside research,
   or both? This determines everything else.
2. **Who reads it?** You alone, a team, future contributors, or mostly AI agents working in
   this repo? Answers change how much context each page must restate.
3. **What are the recurring things worth a page each?** For code: services, modules, database
   tables, endpoints, environment variables, architectural decisions. For research: people,
   organizations, papers, concepts, datasets. Get their actual nouns, not the generic list.
4. **What must never be got wrong?** Domain rules, terms with a specific local meaning, things
   previous documentation kept getting wrong. These become the project-specific rules.
5. **What is out of scope?** Vendored code, generated files, archives, anything they do not
   want read.

If the user gives thin answers, propose a concrete draft from what you found in step 1 and ask
them to correct it. A draft they edit beats an interrogation they abandon.

## 3. Scaffold

Create, without overwriting anything that exists:

```
docs/
  DOCS.md          written from the interview — see below
  README.md        the index, empty to start
  CHANGELOG.md     the log, empty to start
  sources/         empty, for raw material
  summaries/  topics/  entities/
```

Four files and four folders. Nothing else — the schema carries the rules, so there is no
second guidance file to drift out of sync with it.

Also copy the kit's root `AGENTS.md` if the project has none. If it already has one, **append a
short section** pointing to `docs/DOCS.md` rather than replacing the file.

If the project has a `CLAUDE.md`, add one line to it pointing at `docs/DOCS.md`. Do not move
its contents.

## 4. When docs/ already exists

The rule is **merge and reorganize, never overwrite**. Existing documentation is someone's
work and often the best material in the repo.

Classify every existing file, then act:

| What it is | Do this |
|---|---|
| Reads like a topic — an explanation, a design doc, an architecture note | Move it to `docs/topics/`, add front matter, keep the prose. Rename to a kebab-case slug. |
| Reads like an entity — one service, one table, one endpoint, one tool | Move it to `docs/entities/`, add front matter. |
| Raw or external material — specs, vendor PDFs, exports, meeting notes | Move it to `docs/sources/`. It becomes material to scan. |
| You cannot confidently classify it | Move it to `docs/sources/`. **This is the safe default** — nothing is lost, and the next scan will read it and produce proper pages. |
| Generated output — API reference, coverage reports, build artifacts | Leave it exactly where it is. Note it in `DOCS.md` as out of scope. |
| `README.md`, `CONTRIBUTING.md`, `LICENSE`, or anything the project's tooling reads by path | Leave it. Moving these breaks things. |

Rules while reorganizing:

- **Never delete and never rewrite the prose.** You may add front matter, fix the file name,
  and add wikilinks. The content is not yours to improve in this pass.
- **Fix inbound links.** Anything in the repo that linked to a moved file must be updated —
  grep for the old path across the whole project, not just `docs/`.
- **Show the plan before moving anything.** List every move as `old path → new path` and get
  confirmation. This is the one step where a wrong guess costs the user real work.
- If the project's `docs/` is large or clearly load-bearing (a published site, a docs
  generator with a config file like `mkdocs.yml`, `docusaurus.config.js`, `book.toml`), do
  **not** rearrange it. Scaffold the knowledge base at `docs/kb/` instead, say why, and treat
  the existing docs as sources.

## 5. Write DOCS.md

Start from the kit's `docs/DOCS.md` and rewrite three parts with what you learned:

- **Page types** — replace the generic descriptions with this project's actual nouns from
  interview question 3. If they said "one page per service and one per Postgres table," say
  exactly that.
- **Project-specific rules** — replace the shipped defaults entirely. Write four to six rules
  from questions 4 and 5. Vague rules do nothing; each one should be checkable.
- **Out of scope** — list the paths from question 5.

Keep everything else — layers, front matter, links, citations, contradictions, dates. Those
are the parts that make the kit work.

For a codebase, a good starting shape:

```markdown
| Summary | `docs/summaries/<module>.md` | module or package | What it does, its public surface, its dependencies |
| Topic   | `docs/topics/<slug>.md`      | decision or flow  | Why it is built this way; how a request moves through |
| Entity  | `docs/entities/<slug>.md`    | service, table, endpoint, env var | Stable facts and every place it is referenced |
```

## 6. Finish

1. Append the first `docs/CHANGELOG.md` entry: what you scaffolded, what you moved, what you
   left alone.
2. Report to the user:
   - the tree you created
   - every file you moved, and anything you deliberately did not touch
   - **the rules you wrote in `DOCS.md`, quoted** — ask them to read and correct these now,
     while it is cheap
3. Tell them what to do next: put material in `docs/sources/`, then run `/scan-docs`. If you
   moved existing files into `docs/sources/`, say that a scan will turn them into pages.

Do not run a scan yourself. Init sets up; scan is a separate decision.

## Rules

- Never delete a file. Move, or leave alone.
- Never overwrite an existing `DOCS.md`, `AGENTS.md`, or `CLAUDE.md` — merge or append.
- Never rearrange a docs folder that a site generator builds from.
- If the project already has a `docs/DOCS.md`, this kit is already installed. Say so, offer
  `/lint-docs` instead, and stop.
