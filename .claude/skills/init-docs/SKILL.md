---
name: init-docs
description: Set up the knowledge base in a project — interview the user about what they are documenting, scaffold docs/, and write a DOCS.md schema tailored to this project. Handles a docs/ folder that already exists by merging and reorganizing rather than overwriting, upgrades an older install, and restructures a base that has grown. Use when the user says init docs, set up the knowledge base, install this kit here, build docs for this project, start documenting this repo, upgrade the kit, change the schema or presets, move or rename the docs root, or split the knowledge base in two.
---

# Init docs

The front door. Run once per project. It produces a `docs/` folder and, more importantly, a
`docs/DOCS.md` written for *this* project rather than a generic template.

**A generic `DOCS.md` is the main way this kit fails.** The interview is not a formality — it
is where the value is created. Do not skip it, and do not answer the questions yourself.

## 1. Survey, and pick the mode

Look before you interview, so your questions are specific and you never ask what you can see.

- What kind of project is this? Read `README.md`, and the manifest if there is one
  (`package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `composer.json`).
- What is the top-level layout? Where does the real code live?
- Does `docs/` already exist? If so, list what is in it and read enough to classify it.
- Is there documentation or filed material anywhere *else* — `notes/`, `research-papers/`,
  `contracts/`, `wiki/`, loose Markdown or PDFs at the top of the repo?
- Is there existing agent config — `CLAUDE.md`, `AGENTS.md`, `.cursorrules`, `.github/copilot-instructions.md`?

**Then say which of two modes this is.** Everything after this follows from it, so get it right
and say it out loud before asking anything:

| | **Fresh start** | **Merge** |
|---|---|---|
| When | The project has no documentation and no filed material worth keeping | There is already documentation, or a pile of material somewhere |
| Sources | The user chooses where they will go — step 2, question 2 | Wherever the material already is. Nothing moves. |
| Existing files | None to worry about | Classified and either pointed at or moved, never overwritten — step 4 |
| Risk | None. Nothing exists to damage. | Someone's work is already here. This is where care is owed. |

Say what you found and which mode you are in, in one short paragraph, before your first question.
It shows the user you looked, it lets them correct you early, and *"I found 40 PDFs in
`research-papers/`, so this is a merge"* is a far better opening than a generic question.

**When in doubt, it is a merge.** Treating an existing base as a fresh start is the one mistake
in this skill that destroys work; treating a fresh start as a merge just means you asked a
question with an empty answer.

## 2. Interview

**Five or six questions, and most of them depend on the answer to the first.** Ask question 1,
then question 2, then only the block for the kind of project they said it is. Do not ask the other
blocks' questions and do not read them out as options — a user answering *"what are your
services?"* when they have no services is being interviewed by a form, not a colleague.

Batch each block rather than dripping one question at a time. Accept short answers and move on;
propose sensible defaults from the survey and let the user correct them.

### Question 1 — what kind of project is this?

Everything else follows from this. It picks the preset in `docs/DOCS.md` Part 2 that you build
Part 1 from, and it decides which block of questions comes next.

| Ask it like this | Preset | Then ask |
|---|---|---|
| **A codebase** — you want this project's own code documented | *documenting a codebase* | Block A |
| **Documents you write** — a book, a spec set, a policy library, your own notes | *a document library* | Block B |
| **A knowledge centre** — material collected from elsewhere: papers, articles, vendor docs, transcripts | *outside research* | Block C |
| **A system you operate** — incidents, runbooks, on-call knowledge | *operations and runbooks* | Block D |
| A mix | Take the closest and use its block. Do not merge two presets. | that one's block |

Say which one you are proposing and why, from what the survey found, before you ask anything else.
It is the cheapest correction the user will ever make, and getting it wrong means every page is
shaped wrong.

**Say what the preset does not change**, in one line. The folders, the page types, the front
matter and every command are identical for all four; the preset supplies the starting nouns and a
few rules. A user who thinks they are choosing a layout will weigh the decision far more heavily
than it deserves and then hesitate to change it. It is reversible at any time — see *Changing the
shape later* in Part 2.
### Question 2 — where will the source material live?

The mode from step 1 decides the shape of this question.

**Fresh start — a real choice, and the only layout decision the user makes.** Two options, and
nothing else. Put it to them plainly:

> Where should I keep the material you want documented?
>
> **A. `docs/core-sources/`** *(default)* — everything lives under `docs/`, so the whole
> knowledge base is one folder you can copy, move, or open in Obsidian as a vault.
>
> **B. `sources/`** — a folder at the top of the project, beside `docs/`. More visible, and
> easier to drag files into without going two levels down.
>
> Same behaviour either way: I read it, I never write to it. It's a one-line change later if
> you pick wrong.

Take **A** if they have no opinion. Take **B** without argument if they want it — the only
difference is how far down the folder sits, and someone who will actually drop files into a
visible folder beats a tidier tree they never fill.

**Merge — usually not a question at all.** The material is already somewhere, and the answer
is to point at it rather than move it. State what you are doing:

> You already have 40 PDFs in `research-papers/`. I'll point the knowledge base at that folder
> rather than moving them — you keep filing them where you file them now, and the agent reads
> them there. It never writes to that folder either way.

Only ask when the survey found material in two or more places, because there is exactly one
source location and someone has to choose which. Say what you would pick and why.

Two follow-ups in either mode. Is anything in there they would rather the agent not read? And
is there material that should be **cited but not filed** — a source tree, or docs published by
a generator — which becomes *read-in-place sources* instead.

**Never ask about the root.** It is `docs/`, in both modes. The one exception is decided by what
you found rather than by the user: a `docs/` already built by a site generator means the base
goes to `docs/kb/`, and you say so rather than offering a choice. Asking invites a decision with
no upside and a migration if it goes wrong.
### Ask one block, not all four

Each block is two questions. The first gets the nouns that become the **page types**; the second
gets the failure this particular kind of base has, which becomes the **project-specific rules**.
Both want their actual answers, not a category — *"one page per Postgres table and one per
background job"* is usable, *"the usual code things"* is not.

#### Block A — a codebase

1. **What are the units someone would look up by name?** Services, modules, packages, database
   tables, endpoints, environment variables, background jobs, feature flags. Which of those does
   this project actually have, and what does it call them?
2. **What do you keep re-explaining, and what does the code do that the docs get wrong?** The
   decisions re-litigated every few months, and the places where a comment or a README says one
   thing and the code does another. The first becomes decision pages; the second becomes the rule
   that intended behaviour is never written as actual behaviour.

#### Block B — documents you write

1. **What are the documents, and what subjects cut across them?** The subjects matter more than
   the file list here: a document library is navigated by topic, and if you cannot name four or
   five subjects the base will end up a filing cabinet of summaries.
2. **Which terms mean something specific here, and who decides when two documents disagree?**
   Local vocabulary drifts before facts do, so those terms become entity pages. And a library of
   your own documents produces real contradictions rather than differences of opinion — knowing
   whose ruling settles one is what makes them resolvable.

#### Block C — a knowledge centre

1. **What are you actually trying to answer, and whose material is this?** The standing questions
   the collection exists to answer, and who produced the material — vendors, researchers,
   competitors, conference talks. Both shape the topic pages, and the second decides how much
   weight each source carries.
2. **Which sources do you not trust, and what gets quoted at you as fact?** Marketing numbers,
   benchmarks with no method, a vendor's claims about a competitor. These become the attribution
   rules — the ones that keep *"the vendor reports 40% lower latency"* from becoming *"latency is
   40% lower."*

#### Block D — a system you operate

1. **Which systems, and what do people get paged for?** Services, queues, alerts, dashboards,
   config keys, on-call rotas — and the failures that actually wake someone up. The pages worth
   writing are the ones someone reads at 3am.
2. **Which values drift, and where does the spec differ from production?** Timeouts, TTLs, limits,
   replica counts — anything with a number that changed without the document changing. This is the
   preset's central risk, and the answers become the rules about units, as-of dates, and never
   stating a designed value as a running one.

### Then two questions everyone gets

1. **What must never be read?** The security exclusions in `docs/DOCS.md` already cover the usual
   suspects — `.env*`, `*.pem`, `secrets/`. Ask what this project has *beyond* them: a customer
   export, a licensed corpus, an HR folder. Ask separately what is merely out of scope — vendored
   code, generated files, archives — since one is a security rule and the other is a filter.
2. **What would you be most embarrassed to get wrong?** Two or three questions the base must
   answer correctly. These become `docs/scenarios/questions.yaml`, and they are worth more than
   they look — including, especially, the ones whose honest answer is *"nobody knows yet."*

**On who reads it:** do not ask. Infer it. A solo project, a team repo and a base that mostly
serves agents differ in how much context each page restates, and the survey plus the answers above
tell you which this is. If it genuinely matters and you cannot tell, fold it into your proposal —
*"I'll write these for someone new to the project"* — and let them correct you.

If the user gives thin answers, propose a concrete draft from what you found in step 1 and ask
them to correct it. A draft they edit beats an interrogation they abandon.

## 3. Scaffold

Create, without overwriting anything that exists:

```
docs/
  DOCS.md               written from the interview — see below
  INDEX.md              the index, empty to start — or README.md, see below
  CHANGELOG.md          the log, empty to start
  core-sources/         empty, for raw material — omit if sources go elsewhere
  summaries/  topics/  entities/
  scenarios/questions.yaml  what this base must answer — empty to start
```

Four files and five folders — one fewer folder when question 2 pointed the source layer somewhere
else. Nothing beyond that: the schema carries the rules, so there is no second guidance file to
drift out of sync with it.

Leave the page layers flat. Subfolders are worth it past roughly thirty pages in a layer and are
free to add later, so grouping an empty base is guessing at a shape it has not grown into yet.
Mention that they exist when you report the tree; do not build them.

**This is the tree for every project, whatever the preset.** Do not add a folder because the
project seems to want one, do not drop `scenarios/` because the the user had no answer to the scenarios question,
and do not rename a layer to match the project's vocabulary — the vocabulary goes in the page
types table, where it is meant to. Say this to the user when you show them the tree: the shape is
fixed, the words in it are theirs.

Set all three declarations in Part 1 of the new `DOCS.md`, and say what you set them to. That block
is the only place the paths are declared; a path you used but did not declare is the one mistake
here that looks like it worked.

**Root — decided by you, never asked.** `docs/`, unless the project's `docs/` is built by a site
generator (`mkdocs.yml`, `docusaurus.config.js`, `book.toml`), in which case `docs/kb/` — see step
4. A user who explicitly asked for a second, separate base gets the root they named.

**Source folder — from question 2.**

| Mode and answer | Create | Declare |
|---|---|---|
| Fresh, option A | `docs/core-sources/`, empty | `docs/core-sources/` |
| Fresh, option B | `sources/` at the project root, empty | `sources/` |
| Merge | nothing | the folder the material is already in |

Record any read-in-place paths in Part 1 too. Never declare a folder the agent writes to, and
never more than one location.

If you created `sources/` at the root, add a one-line `sources/README.md` saying what the folder
is for and that the agent never writes to it. A bare empty folder at the top of someone's project
is a puzzle three months later, and it is the one place this layout is not self-explaining.

**Commits — write `none`, and say so.** The agent writes files and leaves them in the working tree
for the user to review and commit. Do not ask during the interview; it is a preference, not
knowledge about their project, and it belongs at the end where it will not crowd out a question
that shapes the base. Step 6 puts it to them once, with `none` already set.

**Index — decided by a rule.** `docs/INDEX.md`. The file is an index, not a readme, and almost
every project has a root `README.md` that a second one would compete with. Use `docs/README.md`
only when the project has no root README of its own and nothing else claims the name.

If a `docs/README.md` already exists and reads as a readme — prose for people, not a page list —
leave it exactly as it is and put the index in `INDEX.md` alongside it.

Say which you chose and why in one line; it is the kind of thing a user notices later and wonders
about.

**The project's root `README.md` is the front door, and the base should be visible from it.** Add
one short section — what the knowledge base covers, where it lives, and the commands. If the
project has no root README at all, write a minimal one: what the project is, in a line or two, and
that section. Change nothing else in it, ever. It is the most-read file in the repository and it is
not yours.

Also copy the kit's root `AGENTS.md` if the project has none. If it already has one, **append a
short section** pointing to `docs/DOCS.md` rather than replacing the file.

If the project has a `CLAUDE.md`, add one line to it pointing at `docs/DOCS.md`. Do not move
its contents.

## 4. Merge mode — when documentation already exists

Skip this whole section on a fresh start; there is nothing to merge.

The rule is **merge and reorganize, never overwrite**. Existing documentation is someone's
work and often the best material in the repo.

Two things worth saying before you start. Material already filed in its own folder is **pointed
at, not moved** — that is question 2, and it is settled before you get here. What this section
handles is everything else: a `docs/` folder with a mix of explanations, specs and stray notes in
it, where each file needs classifying one at a time.

Classify every existing file, then act:

| What it is | Do this |
|---|---|
| Reads like a topic — an explanation, a design doc, an architecture note | Move it to `docs/topics/`, add front matter, keep the prose. Rename to a kebab-case slug. |
| Reads like an entity — one service, one table, one endpoint, one tool | Move it to `docs/entities/`, add front matter. |
| Raw or external material — specs, vendor PDFs, exports, meeting notes | Move it to `docs/core-sources/`. It becomes material to scan. |
| You cannot confidently classify it | Move it to `docs/core-sources/`. **This is the safe default** — nothing is lost, and the next scan will read it and produce proper pages. |
| Generated output — API reference, coverage reports, build artifacts | Leave it exactly where it is. Note it in `DOCS.md` as out of scope. |
| `README.md`, `CONTRIBUTING.md`, `LICENSE`, or anything the project's tooling reads by path | Leave it. Moving these breaks things. |
| A `docs/README.md` that reads as a readme — an introduction for people, not a page list | Leave it. Declare the index as `docs/INDEX.md` and write the index there. |
| A `docs/README.md` that is already a list of the folder's contents | Keep it as the index. Declare it, rewrite it in the index format, and say you did. |

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
  the existing docs as sources. **Then set both declarations in `docs/kb/DOCS.md`: the root to
  `docs/kb/`, and the source folder to `docs/`** — the whole contract and all four other skills
  read those two lines to find the base and its material. The published pages stay where they are
  and are cited at their real paths; nothing is copied into `docs/kb/core-sources/`, which is not
  created at all.

## 5. Write DOCS.md

Start from the kit's `docs/DOCS.md`. **Apply the preset chosen in question 1**, then rewrite
three parts of Part 1 with what you learned:

- **Page types** — start from the preset's table, then replace its nouns with this project's
  actual ones, from the first question of the block you asked. If they said "one page per service
  and one per Postgres table," say exactly that. The preset is a starting shape, not an answer.
- **Project-specific rules** — take the preset's suggested rules as a base, keep the ones that
  apply, then add rules from the block's second question and from the security question until you
  have four to six. Replace the shipped defaults entirely. Vague rules do nothing; each one should
  be checkable.
- **Out of scope** — list the paths from the security question, plus the preset's. Append this
  project's sensitive paths to the security exclusion list rather than replacing what is there.
- **The declarations** — root, source location and index, per step 3. The codebase preset's
  `src/**` belongs in read-in-place, not in the source folder.

Then **delete the whole `## Presets` section from Part 2**, including the preset you used and
*Writing your own preset*. It is a menu for `init-docs`, not documentation for the project —
leaving three unused presets in a project's contract is four page-type tables competing to be the
real one.

**Keep the rest of Part 2**, and *Choosing a different root* and *Changing the shape later* in
particular. Those two are how the project's owner adjusts the base later without guessing, and
Part 1 links to them. A contract that explains only how to use the base and not how to change it
is how a knowledge base gets abandoned instead of edited.

**Keep everything else exactly as shipped** — layers, authority, front matter, claim types,
lifecycle, provenance, freshness, links, citations, contradictions, review and ruling, retrieval and
context, answer states, security, git conflicts, dates. Those sections are the governance
contract, they are what make the kit work, and they are not the place for local taste. If a
project genuinely needs one changed, change it deliberately and note it in
`docs/CHANGELOG.md` — do not quietly trim it because the file is long.

Then write `docs/scenarios/questions.yaml` from the scenarios question — three to five cases, in the
format the shipped template documents. Include at least one `expect_state: unknown`. If the
the user had no answer to the scenarios question, leave the file with `questions: []` and its comments intact,
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
   - **the three declarations**, and for the index, one line on why that name
   - every file you moved, and anything you deliberately did not touch
   - the section you added to the project's root `README.md`
   - **the rules you wrote in `DOCS.md`, quoted** — ask them to read and correct these now,
     while it is cheap
3. Tell them what to do next: put material in the source folder, then run `/scan-docs`. If you
   moved existing files into it, or declared a folder that already had material in it, say that a
   scan will turn them into pages — and roughly how many files that is.
4. **Ask what to call them.** Run `git config user.name`. If it returns something, offer it; if it
   returns nothing, ask plainly. One question, and take the answer as given:

   > Git has you as **Fenton Martin** — shall I call you Fenton, or something else?

   Write the answer into the `Address:` declaration verbatim. If they decline, do not press: leave
   it `(not set)` and use *"you"* from then on. Never take a name from a commit log or an email
   address without asking — a repository's authors are frequently not the person in front of you.

   Ask this here, at the end, rather than opening with it. A form of address is not knowledge
   about the project, and leading with it makes the first minute of a setup about you instead of
   their work.
5. **Offer the one setting.** Commits are `none` — you write files, they review and commit. Put
   the alternatives in two lines and take whatever they say:

   > I've set commits to `none`: I'll write files and leave them for you to review. I'd keep it
   > there for now — reading `git diff` after a scan is how you find where my judgment differs
   > from yours, and it's what turns a generic base into a good one. Once the rules have settled,
   > `per-run` (one commit per command) or `per-file` are a one-line change in `docs/DOCS.md`.

   Change the declaration if they ask for something else, and do not argue past one sentence. It
   is their repository.
6. Say in one line that the rules are theirs to change at any time and that most changes cost
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
| `Part 1 — The contract` present, no declarations block under `## Layers` | `2.0` |
| A declarations block and a `## Commits` section | `2.1` — already current |

**From `2.0` to `2.1` there is nothing to migrate.** Replace the kit files (step 1) and stop.
Their `docs/` is already correct: every declaration defaults to exactly where their base already
keeps things, `docs/README.md` included. Skip steps 2, 4 and 5 — there is no folder to rename and
no lint sweep to run. Then do step 3, which is the part that matters here, and offer these without
doing them unasked:

- add the declarations block to Part 1 — root, source folder, index — set to where their base
  actually keeps things;
- **add the `Commits` section, set to `none`, and put the choice to them once.** It is new, it
  changes what the agent does to their repository, and defaulting it silently is the wrong way to
  introduce that. Same two lines as an init: `none` writes files for them to review, `per-run` and
  `per-file` are a one-line change later;
- **ask what to call them**, as in step 6 of an init, and add the `Address:` declaration;
- add the knowledge-base section to the project's root `README.md` if it has none;
- rename the index to `INDEX.md` if the project has a root README that competes with
  `docs/README.md` — a `git mv`, a declaration change, and a lint run;
- name what is new in case they want it: a fourth preset, subfolder grouping, `/all-docs` and
  `/help-docs`, and *Changing the shape later* in Part 2 for reshaping the base.

### Step 0 — does the base still describe what it is for?

**Every upgrade, including a `2.x` one.** A `DOCS.md` was written from an interview about a project
that has since moved. Read their page types and project-specific rules against what the repository
now looks like, and say what you notice:

- Do the page types name things that still exist? A base whose entity type is *"one page per
  microservice"* in a repo that consolidated to a monolith is describing a project that is gone.
- Does the preset still fit? A codebase base that has filled up with vendor PDFs is really doing
  outside research, and its rules are working against it.
- Do the project-specific rules still bite? A rule nothing has violated in a year is either
  well-internalized or obsolete, and the difference is worth a sentence.
- Is anything in the base's own `CHANGELOG.md` a decision that should have become a rule?

**Report; do not rewrite.** This is the one part of an upgrade that needs the user's judgment, and
they are the only one who knows which drift was deliberate. Offer to make the changes they confirm,
in the same pass as step 3. If nothing has drifted, say that in a line and move on — it is worth
knowing.

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
4. their **declarations**, if any are not the default — and fill in any the old file did not have,
   from where their base actually keeps things rather than from what the shipped file says
5. their **`Commits` setting**, if they had one. If they did not, it is `none` and you say so —
   never carry a shipped default into a repository the user has not agreed to have committed to

Then delete every preset section, as in step 5 of an init.

**Show them the diff before writing it.** Their old `DOCS.md` was written from an interview, and
those three sections are the only part of this kit they authored. Losing them silently is the
worst outcome of an upgrade.

If they customized anything *else* — a changed link style, an edited citation format — port that
too and say you did. Do not assume the shipped text is what they want back.

### Step 4 — run `/lint-docs`

It rewrites the `sources:` values and citations still pointing at `docs/sources/` (check 15),
adds `status: active` and a defaulted `claim_type` to every page that predates those fields
(check 2), and reports what needs the owner.

Expect a large diff and a lot of WARNINGs on a base of any size. That is the upgrade landing, not
breakage.

### Step 5 — scenarios, optional

`docs/scenarios/questions.yaml` is new and empty by default. Offer to write three to five
scenarios from the scenarios question — including at least one `expect_state: unknown` — or leave
the template and say it is theirs to fill in.

### Finish

Append one `docs/CHANGELOG.md` entry recording the upgrade, then report: the version they moved
from, what step 0 found about whether the base still describes the project, the three sections you
ported, the page count lint touched, and anything left open.

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
2. Change the `Root:` declaration in Part 1.
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

**Grouping a layer into subfolders.** Worth proposing unasked once a layer passes roughly thirty
pages, because that is when the index stops being scannable. Group by subject, two levels at most,
and never by date or by type. Slugs do not change, so no wikilink breaks — but check for collisions
first (lint check 17), since the whole risk here is two pages ending up with one slug in different
folders. Summary pages move to mirror their sources' paths, not independently. Rebuild the index
and lint.

**Renaming the index.** `git mv docs/README.md docs/INDEX.md`, change the declaration, lint. Worth
offering when a project's root README makes the second one confusing, which is most projects.

**Moving or repointing the source folder.** If they are moving material, `git mv` it, change the
declaration, and lint — check 15 rewrites the citations. If they are pointing at a folder that
already exists, move nothing: change the declaration and expect check 13 to report every file in
it as unread, because it is. Say how many, and that it is a scan queue rather than damage.
Promoting read-in-place paths to filed sources works the same way, at the same cost.

**Refuse these**, and say why in one sentence: renaming or adding a layer, adding a fourth page
type, changing the front-matter schema, declaring more than one source folder, or pointing the
source declaration at a folder the agent writes to. They are what `lint-docs` and `test-docs` check
against, and a local change to them means every future version of the kit fights this base. If the
user insists after hearing that, do it, note it prominently in `docs/CHANGELOG.md`, and tell them
which checks will now misfire.

Whatever you do here, append one `docs/CHANGELOG.md` entry recording the before and after shape.
A base whose structure changed without a log entry is one a future reader cannot make sense of.

## Rules

- Never delete a file. Move, or leave alone.
- Never overwrite an existing `DOCS.md`, `AGENTS.md`, or `CLAUDE.md` — merge or append.
- Never rearrange a docs folder that a site generator builds from.
- Never invent, rename, or omit a layer. Every base gets the same five folders and the same three
  page types; only the nouns and the rules are project-specific.
- If you scaffold anywhere other than `docs/`, or point at a source folder other than
  `docs/core-sources/`, set the matching declaration in Part 1. An undeclared path is the one
  mistake here that looks like it worked.
- Never write to a folder you declared as the source layer, and never declare a folder the agent
  already writes to.
- If the project already has a `docs/DOCS.md`, the kit is installed. **Do not init — follow
  step 7, Upgrading an existing install.** Never re-interview a project that already has a
  schema, and never overwrite the three sections its owner authored.
