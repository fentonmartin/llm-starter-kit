---
name: init-docs
description: Set up the knowledge base in a project — interview the user about what they are documenting, scaffold docs/, and write a DOCS.md schema tailored to this project. Handles a docs/ folder that already exists by merging and reorganizing rather than overwriting, upgrades an older install, and restructures a base that has grown. Use when the user says init docs, set up the knowledge base, install this kit here, build docs for this project, start documenting this repo, upgrade the kit, change the schema or presets, move or rename the docs root, or split the knowledge base in two.
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
   operating a running system, holding a library of documents, or a mix? This determines
   everything else — it selects the preset in `docs/DOCS.md` Part 2 that you will build Part 1
   from:

   | Answer | Preset |
   |---|---|
   | This codebase | *Preset: documenting a codebase* |
   | Outside research, a reading pile | *Preset: outside research* |
   | Running a system — incidents, runbooks, on-call | *Preset: operations and runbooks* |
   | The documents *are* the project — a book, a spec set, a policy library, a pile of notes | *Preset: a document library* |
   | A mix | Take the closest and rename its page types. Do not merge two presets. |

   Say which preset you are proposing and why, before asking question 2. It is the cheapest
   correction the user will ever make, and getting it wrong means every page is shaped wrong.

   **Tell them what the preset does not change.** The folders, the page types, the front matter,
   and every command are identical for all four — the preset only supplies the starting nouns and
   rules. A user who thinks they are choosing a layout will worry about the choice far more than
   it deserves, and will hesitate to change it later. It is a cheap decision, reversible at any
   time: see *Changing the shape later* in Part 2.
2. **Who reads it?** You alone, a team, future contributors, or mostly AI agents working in
   this repo? Answers change how much context each page must restate.
3. **What are the recurring things worth a page each?** For code: services, modules, database
   tables, endpoints, environment variables, architectural decisions. For research: people,
   organizations, papers, concepts, datasets. Get their actual nouns, not the generic list.
4. **What must never be got wrong?** Domain rules, terms with a specific local meaning, things
   previous documentation kept getting wrong. These become the project-specific rules.
5. **What is out of scope, and what must never be read?** Vendored code, generated files,
   archives — and separately, anything sensitive. The security exclusions in `docs/DOCS.md`
   cover the usual suspects (`.env*`, `*.pem`, `secrets/`); ask what this project has *beyond*
   them. A customer export, a licensed corpus, an HR folder.
6. **What would you be most embarrassed to get wrong?** Two or three questions the knowledge
   base must answer correctly. These become `docs/scenarios/questions.yaml`, and they are worth
   more than they look — including the ones the answer to is *"nobody knows yet."*

If the user gives thin answers, propose a concrete draft from what you found in step 1 and ask
them to correct it. A draft they edit beats an interrogation they abandon.

## 3. Scaffold

Create, without overwriting anything that exists:

```
docs/
  DOCS.md               written from the interview — see below
  README.md             the index, empty to start
  CHANGELOG.md          the log, empty to start
  core-sources/         empty, for raw material
  summaries/  topics/  entities/
  scenarios/questions.yaml  what this base must answer — empty to start
```

Five files and five folders. Nothing else — the schema carries the rules, so there is no
second guidance file to drift out of sync with it.

**This is the tree for every project, whatever the preset.** Do not add a folder because the
project seems to want one, do not drop `scenarios/` because the user had no answer to question 6,
and do not rename a layer to match the project's vocabulary — the vocabulary goes in the page
types table, where it is meant to. Say this to the user when you show them the tree: the shape is
fixed, the words in it are theirs.

**The root.** Scaffold at `docs/` unless one of these applies:

| Situation | Root |
|---|---|
| Normal — anything else | `docs/` |
| The project's `docs/` is built by a site generator (`mkdocs.yml`, `docusaurus.config.js`, `book.toml`) | `docs/kb/` — see step 4 |
| The user asked for a second, separate knowledge base in this project | the root they named |

If you scaffold anywhere other than `docs/`, **set the root-binding line at the top of Part 1 of
the new `DOCS.md` to that root** and say so in your report. That line is the only place the root
is declared; without it every path in the contract and in the four other skills still points at
`docs/`, and nothing will work.

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
| Raw or external material — specs, vendor PDFs, exports, meeting notes | Move it to `docs/core-sources/`. It becomes material to scan. |
| You cannot confidently classify it | Move it to `docs/core-sources/`. **This is the safe default** — nothing is lost, and the next scan will read it and produce proper pages. |
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
  the existing docs as sources. **Then set the root-binding line in `docs/kb/DOCS.md` to
  `docs/kb/`** — the whole contract and all four other skills read that one line to find the base.
  The published pages stay where they are and are cited as sources, at their real paths; they are
  not copied into `docs/kb/core-sources/`.

## 5. Write DOCS.md

Start from the kit's `docs/DOCS.md`. **Apply the preset chosen in question 1**, then rewrite
three parts of Part 1 with what you learned:

- **Page types** — start from the preset's table, then replace its nouns with this project's
  actual ones from question 3. If they said "one page per service and one per Postgres table,"
  say exactly that. The preset is a starting shape, not an answer.
- **Project-specific rules** — take the preset's suggested rules as a base, keep the ones that
  apply, and add rules from questions 4 and 5 until you have four to six. Replace the shipped
  defaults entirely. Vague rules do nothing; each one should be checkable.
- **Out of scope** — list the paths from question 5, plus the preset's. Append this project's
  sensitive paths to the security exclusion list rather than replacing what is already there.

Then **delete the whole `## Presets` section from Part 2**, including the preset you used and
*Writing your own preset*. It is a menu for `init-docs`, not documentation for the project —
leaving three unused presets in a project's contract is four page-type tables competing to be the
real one.

**Keep the rest of Part 2**, and *Choosing a different root* and *Changing the shape later* in
particular. Those two are how the project's owner adjusts the base later without guessing, and
Part 1 links to them. A contract that explains only how to use the base and not how to change it
is how a knowledge base gets abandoned instead of edited.

**Keep everything else exactly as shipped** — layers, authority, front matter, claim types,
lifecycle, provenance, freshness, links, citations, contradictions, human review, retrieval and
context, answer states, security, git conflicts, dates. Those sections are the governance
contract, they are what make the kit work, and they are not the place for local taste. If a
project genuinely needs one changed, change it deliberately and note it in
`docs/CHANGELOG.md` — do not quietly trim it because the file is long.

Then write `docs/scenarios/questions.yaml` from interview question 6 — three to five cases, in the
format the shipped template documents. Include at least one `expect_state: unknown`. If the
user had no answer to question 6, leave the file with `questions: []` and its comments intact,
and say it is theirs to fill in.

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
3. Tell them what to do next: put material in `docs/core-sources/`, then run `/scan-docs`. If you
   moved existing files into `docs/core-sources/`, say that a scan will turn them into pages.
4. Say in one line that the rules are theirs to change at any time and that most changes cost
   nothing — `docs/DOCS.md` Part 2, *Changing the shape later*, says which are free and which need
   a migration. The most common way this kit is abandoned is a user who decides the schema was
   wrong and does not know they were allowed to edit it.

Do not run a scan yourself. Init sets up; scan is a separate decision.

## 7. Upgrading an existing install

If `docs/DOCS.md` already exists, the kit is installed and **this is an upgrade, not an init.**
Do not interview, do not scaffold, and do not overwrite their `docs/`.

Say which version they are on and what will change, then work through the steps below in order.
Stop at any point they want to review.

**Which version.** Read their `docs/DOCS.md`:

| What you see | Version |
|---|---|
| No `Part 1` / `Part 2` headings, front matter without `status` and `claim_type` | `1.x` |
| `docs/sources/` still in the layers table | `1.x` |
| `Part 1 — The contract` present, no root-binding line under `## Layers` | `2.0` |
| A root-binding line under `## Layers` | `2.1` — already current |

**From `2.0` to `2.1` there is nothing to migrate.** Replace the kit files (step 1) and stop.
Their `docs/` is already correct: the root defaults to `docs/`, which is where their base already
is. Offer, and do not do unasked, two optional touches: add the root-binding line to their Part 1
so the declaration is explicit, and mention that `Changing the shape later` and the fourth preset
now exist in the kit's Part 2 if they ever want to reshape the base. Skip steps 2, 3 and 4
entirely — there is no folder to rename and no lint sweep to run.

The steps below are the `1.x` → current path.

### Step 1 — kit files

`.claude/` and the root `AGENTS.md` are kit-owned. **Replace them wholesale**; there is nothing
of the user's in them. If they installed as a plugin, the plugin update does this. If they copied
files, re-copy and overwrite.

**Never touch `docs/` in this step.** That is their knowledge base, not kit files.

### Step 2 — rename the sources folder

```bash
git mv docs/sources docs/core-sources
```

Do the paths afterwards, in step 4. Rewriting paths first breaks every citation in the base.

### Step 3 — upgrade `docs/DOCS.md`

**Start from the shipped 2.0 file and port their customizations into it. Do not patch their old
file.** `2.0` adds roughly a dozen governance sections; merging those into a `1.x` file by hand
is where the mistakes live, while carrying three sections the other way is mechanical.

Copy the kit's `docs/DOCS.md` over theirs, then port back, verbatim:

1. their **page types** table
2. their **project-specific rules**
3. their **out of scope** paths — appended to the shipped security exclusions, never replacing
   them

Then delete every preset section, as in step 5 of an init.

**Show them the diff before writing it.** Their old `DOCS.md` was written from an interview, and
those three sections are the only part of this kit they authored. Losing them silently is the
worst outcome of an upgrade.

If they customized anything *else* — a changed link style, an edited citation format — port that
too and say you did. Do not assume the shipped text is what they want back.

### Step 4 — run `/lint-docs`

It rewrites the `sources:` values and citations still pointing at `docs/sources/` (check 15),
adds `status: active` and a defaulted `claim_type` to every page that predates those fields
(check 2), and reports what needs a human.

Expect a large diff and a lot of WARNINGs on a base of any size. That is the upgrade landing, not
breakage.

### Step 5 — scenarios, optional

`docs/scenarios/questions.yaml` is new and empty by default. Offer to write three to five
scenarios from interview question 6 — including at least one `expect_state: unknown` — or leave
the template and say it is theirs to fill in.

### Finish

Append one `docs/CHANGELOG.md` entry recording the upgrade, then report: the version they moved
from, the three sections you ported, the page count lint touched, and anything left open.

## 8. Restructuring a base that has grown

A base that is being used will need adjusting, and most adjustments are not a restructure at all.
Find out which kind this is before touching anything. `docs/DOCS.md` Part 2, *Changing the shape
later*, is the reference; this is what to do when a user asks.

**First, check it is not free.** Renaming the nouns in the page types table, rewording or adding
project-specific rules, and changing out-of-scope paths need no migration and no lint run — edit
Part 1, note it in `docs/CHANGELOG.md`, done. Say so plainly rather than proposing a project.
Users routinely ask for a restructure when they want a rule changed.

**Switching preset.** The base turned out to be about something else. Take the three sections from
the new preset in the kit's Part 2, rewrite them with the project's nouns as in step 5, then run
`/lint-docs`. **Do not bulk-rewrite existing pages to match.** They were true when written and
their provenance is intact; the new rules govern what happens next, and pages get reshaped as they
are next touched. A migration that rewrites the whole base loses the thing the base is for.

**Moving the root.** In this order, in one commit:

1. Move the folder whole — never lift one layer out from under the others. Most moves are
   `git mv docs wiki`; moving a folder *into* a subfolder of itself needs a staging step, because
   git refuses it in one:

   ```bash
   mkdir kb-tmp && git mv docs/* kb-tmp/
   mkdir -p docs && git mv kb-tmp docs/kb
   ```
2. Change the root-binding line in Part 1 to the new root.
3. Run `/lint-docs`. Check 15 rewrites the `sources:` values and citations still pointing at the
   old root and reports what it cannot.
4. Grep the whole repo for the old path. `CLAUDE.md`, `AGENTS.md`, CI config, and README links do
   not fix themselves, and lint does not look outside the base.

Show the user steps 1 and 2 before running them, and say how many files step 3 will touch.

**Splitting into two bases.** Only when the two halves have genuinely different authority — facts
that come from the project itself versus facts that come from other people's material. Never split
because a folder got large; a large base is what success looks like, and `ask-docs` selects by
index rather than reading everything.

If it is a real split: decide the boundary first and say it out loud, `/init-docs` the second root
(a full interview — the second base gets its own preset and its own rules), `git mv` the pages that
belong to it, then lint both. Wikilinks broken by the split are resolved by restating the fact in
the new base with its own citation, **not** by linking across roots.

**Merging two bases.** `git mv` the second base's four layers into the first, port its
project-specific rules into the survivor's Part 1 rather than discarding them, then lint. Expect
duplicate findings: two bases worth merging were covering the same ground.

**Refuse these**, and say why in one sentence: renaming or adding a layer, adding a fourth page
type, changing the front-matter schema, moving `core-sources/` out from under the root. They are
what `lint-docs` and `test-docs` check against, and a local change to them means every future
version of the kit fights this base. If the user insists after hearing that, do it, note it
prominently in `docs/CHANGELOG.md`, and tell them which checks will now misfire.

Whatever you do here, append one `docs/CHANGELOG.md` entry recording the before and after shape.
A base whose structure changed without a log entry is one a future reader cannot make sense of.

## Rules

- Never delete a file. Move, or leave alone.
- Never overwrite an existing `DOCS.md`, `AGENTS.md`, or `CLAUDE.md` — merge or append.
- Never rearrange a docs folder that a site generator builds from.
- Never invent, rename, or omit a layer. Every base gets the same five folders and the same three
  page types; only the nouns and the rules are project-specific.
- If you scaffold anywhere other than `docs/`, set the root-binding line in Part 1 to match.
- If the project already has a `docs/DOCS.md`, the kit is installed. **Do not init — follow
  step 7, Upgrading an existing install.** Never re-interview a project that already has a
  schema, and never overwrite the three sections its owner authored.
