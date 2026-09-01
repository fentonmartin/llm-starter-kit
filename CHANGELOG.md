# Changelog

Release history for `llm-starter-kit` itself. The log of what an agent did *inside* a knowledge
base is `docs/CHANGELOG.md`, which is a different file with a different purpose.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versioning is [semantic](https://semver.org/spec/v2.0.0.html), applied to the schema and the
command surface: a breaking change is one that invalidates an existing knowledge base.

## [2.3.0] — 2026-09-01

One structure for every use case, and a kit you can actually be shown how to use.

`2.0.0` made a page auditable. This release answers the question that follows — *what if my
project isn't a codebase?* — without giving each kind of project its own layout, and then fills
in what was missing for someone meeting the kit for the first time: a tutorial, a way to ask what
state things are in, a single command that does the whole pass, and the reasoning behind every
folder name written down where it can be read.

**Nothing breaks.** No folder, field, command or check was renamed or removed. Every declaration
defaults to what an existing base already uses — `docs/`, `docs/core-sources/`, `docs/README.md`
— so a `2.0` base is current as it stands, and subfolders are optional everywhere.

> `2.1.0` and `2.2.0` were never released. They existed as intermediate numbers while this work
> was in progress and are folded in here, which is why the log goes `2.0.0` → `2.3.0`. Upgrading
> is `2.0` → `2.3`, and there is nothing to migrate.

### Added — shape and setup


- **A fourth preset: *a document library*.** For a project whose documents *are* the project — a
  book or spec set being written, a policy library, a pile of notes used as the knowledge base.
  Distinct from *outside research* in that the material is the user's own and therefore
  authoritative: a claim needs no attribution, and two of your own documents disagreeing is a real
  contradiction rather than a difference of opinion between sources. Topic pages carry the weight,
  and a base that produces summaries and no topics is the failure mode to watch for.

- **A declared root.** Part 1 of the contract now opens by naming the folder the base lives in.
  Every skill reads that one line and resolves its `docs/…` paths against it. `docs/` remains the
  default and the right answer for almost every project; the two that need another are a `docs/`
  a site generator already owns (`docs/kb/`) and a project running two separate bases.

  This makes an existing feature actually work. `init-docs` has always scaffolded to `docs/kb/`
  when it found a `mkdocs.yml` or a `docusaurus.config.js`, but nothing told the rest of the
  contract or the other four skills where the base had gone. It does now.

- **Two knowledge bases in one project**, for the case where a project holds two bodies of
  knowledge with genuinely different authority — one whose facts come from its own code, one whose
  facts come from other people's papers. Each base is self-contained with its own root, contract,
  preset and index; every command takes a root argument; a run never spans two bases and pages
  never link across them. Documented as a pattern rather than a feature, with the warning that
  matters most: do not split because a folder got large.

- **A migration guide — *Changing the shape later*, in Part 2.** Sorted by what it costs. Renaming
  page-type nouns, rewording rules and changing out-of-scope paths are free. Switching preset,
  reorganizing within a layer, and changing link style cost a lint run. Moving the root or
  splitting a base is real work, with the steps in order. Renaming a layer or adding a page type is
  never. `init-docs` §8 is the operational version of the same thing, including which requests to
  push back on.

- **A recipe for writing your own preset**, since four will not cover everything: a preset may set
  the page types table, the out-of-scope paths, and the project-specific rules. Nothing else. A
  preset that wants a fourth page type or a renamed folder is a fork, and lint will not understand
  it.

- **Hard rule 12 in `AGENTS.md`** — never add, rename, or remove a layer, and never add a page
  type. The vocabulary belongs in the page types table, not in the folder names.


### Added — where things live


- **A declared source folder.** Projects that already file their material — `research-papers/`,
  `notes/`, `contracts/` — can point the base at it rather than moving it into a folder the kit
  invented. The folder stays human-owned and immutable to the agent wherever it sits; the
  declaration is what establishes that, not the folder's name or location.

  Three constraints, and they are the reason this is a declaration rather than a free-for-all:
  exactly one folder, because one summary page per source file is the backbone of provenance and
  two folders make that ambiguous the first time a file name repeats; no overlap with a page
  layer, because a source folder containing `topics/` makes the agent's own output look like
  source material; and it must be a folder the agent never authors in.

- **Read-in-place sources.** An explicit home for material that is cited but not filed — a source
  tree that changes every commit, or a published docs folder the base was scaffolded beside. They
  produce no summary pages, are skipped by the unread-source and excluded-material checks, and
  their citations record the commit they were read at. This existed informally as a line in the
  codebase preset (*"treat `src/**` as sources"*); it is now part of the contract, which is where
  the distinction between curated and merely-present material belongs.

- **`lint-docs` check 16, the source declaration.** ERROR, because everything downstream is
  silently wrong when it is broken and nothing else notices: a declared folder that does not
  exist, more than one declared, or one that overlaps a page layer. Never fixed silently — each
  means the contract says something its author did not intend.

- **An interview question about where material lives**, asked only when the survey found material
  already organized somewhere. A new project gets `docs/core-sources/` and no question.


### Added — safety and voice

- **A protocol for a source too large to read.** Previously `scan-docs` said *"read in ranges until
  you reach the end"* and stopped there, while `AGENTS.md` rule 6 forbids silent truncation — so
  the contract banned exactly what the procedure would drift into, in the one command that writes
  the provenance chain. A source past what the agent can hold now produces **no page and a
  report**, with three options: split the file into parts that each become their own source, name
  a page range summarized as an explicit partial, or leave it unread. A partial summary is always
  marked as one — `status: draft`, the range in the first line, the remainder an open question.

- **A protocol for a source that cannot be read at all.** Scanned PDFs with no text layer, formats
  the agent cannot open, `.url` files with no network to fetch them. Each is reported and produces
  no page. Spreadsheets get their shape summarized — columns, row count, what a row is — never
  their contents. The rule underneath: an unread source is a known gap, an invented summary is a
  false one, and nothing downstream can tell the difference.

- **Guidance on keeping the repository usable.** The source layer fills with things git handles
  badly, and unlike pages, sources never get smaller. Prefer text to binary where there is a
  choice; Git LFS is the normal answer at scale and changes no paths; gitignoring the source
  folder is allowed and costs you provenance nobody else can verify. Whatever you choose, the
  paths in `sources:` and citations must stay stable.

- **A form of address, asked once.** `init-docs` reads `git config user.name` and offers it —
  *"Git has you as Fenton Martin — shall I call you Fenton, or something else?"* — recording the
  answer verbatim in an `Address:` declaration. Decline and it is `(not set)`, which means *"you"*
  and no further asking. It never takes a name from a commit log or an email address without
  asking, because a repository's authors are frequently not the person in front of it. Asked at
  the end, not the start: a form of address is not knowledge about the project.

### Added — using it


- **Setup has two modes, and says which one it is in.** `/init-docs` now opens by deciding whether
  this is a **fresh start** (no documentation, nothing filed) or a **merge** (there is already
  material), states it, and lets everything else follow. When in doubt it is a merge — treating an
  existing base as a fresh start is the one mistake in this skill that destroys work, while the
  reverse just means a question with an empty answer.

  This replaces a source-location question that tried to cover both cases at once and read like a
  configuration menu. Now a fresh start gets one clear choice and a merge gets told what is
  happening to its files.

- **On a fresh start, one layout choice: `docs/core-sources/` or a top-level `sources/`.** Same
  behaviour either way — read, never written — and a one-line change later. `core-sources/` keeps
  the base as one copyable folder; `sources/` sits at the top of the project where it is easier to
  drop files into, and gets a one-line README explaining what it is. Someone who will actually fill
  a visible folder beats a tidier tree they never fill, so the second option is offered without
  argument.

- **A seventh command: `/all-docs`** — the sit-down-to-the-docs pass. Scan new material, lint what
  that changed, test whether the base still answers, in that order because each stage changes what
  the next one sees. Stages with nothing to do are skipped and said to be skipped.

  It reports the plan and the cost *before* it starts, and stops to ask when the scan is large —
  more than roughly ten unread sources means a run whose cost nobody agreed to, so it offers a
  batch instead. At the end it produces **one** report, leading with a counted *needs you* list
  rather than three reports in a row.

  It orchestrates rather than reimplements: each stage follows its own `SKILL.md`, and where the
  two disagree the stage wins. Running commands together confers no authority the commands lack —
  contradictions and human-decision fields stay untouchable. It never runs `ask-docs`, because a
  question is a human act with a human's question in it. It stops immediately if a security
  exclusion fires, and skips `test` when lint found errors that make the base unreadable, since
  testing an unreadable base produces noise rather than signal. One changelog entry per pass, not
  three.

- **A commit setting, declared in Part 1: `none`, `per-run`, or `per-file`.** Default `none` —
  the agent writes files and stops, and you review and commit.

  `none` is the default on purpose. Reading `git diff` after a scan is the feedback loop that turns
  a generic base into a good one, and auto-committing does not remove that review, it removes the
  moment that prompts it. The recommendation is to turn it on once the rules in `DOCS.md` have
  stopped changing.

  Rules that hold at every setting: stage only the paths the run wrote — never `git add -A`, which
  would sweep up whatever the user had open, and someone mid-branch is the normal case; never
  commit when a security exclusion fired or a conflict is unresolved; never push, at any setting,
  because committing is local and reversible and pushing is neither; and a commit never replaces
  the report.

  `init-docs` sets it to `none` and puts the choice once at the end rather than in the interview —
  it is a preference, not knowledge about the project. An upgrade adds the section and asks the
  same question, because it changes what the agent does to someone's repository and defaulting
  that silently is the wrong way to introduce it.

- **Hard rule 13 in `AGENTS.md`** — never commit unless the contract says to, and never push.

- **A sixth command: `/help-docs`.** What this base is, what state it is in, and the one thing
  worth doing next — then the command list, in that order, because the question behind *"what can
  I do?"* is almost always *"what should I do now?"*

  It works out the state from the declarations, the index, and directory listings only: it never
  opens a page or a source, because help that costs a full context read is not help. Six states,
  reported first-match — not installed, no material yet, nothing scanned, unread sources, no
  scenarios, healthy — each with one next step and the command for it. It writes nothing, fixes
  nothing it notices, and is explicitly allowed to answer *"nothing is owed"* rather than invent
  work. Asked something narrower (*"how do I add a source?"*) it answers that in two lines instead
  of printing a status report.

  **There is no `/help` alias**, deliberately: every agent this kit runs in already has one, and
  shadowing it would break something people rely on. The long-name rationale that keeps `/lint`
  from colliding with code linters applies here more strongly.

- **The interview branches by use case instead of asking everyone everything.** Two questions for
  everyone — what kind of project this is, and where sources go — then **two questions in that
  kind of project's own vocabulary**, and two at the end. A codebase is asked what units someone
  looks up by name and where the code contradicts the docs; a document library is asked what
  subjects cut across the documents and who rules when two disagree; a knowledge centre is asked
  what it is trying to answer and which sources it does not trust; an operations base is asked
  what people get paged for and which values drift.

  Previously every project got the same seven generic questions, which produced generic answers
  and then a generic `DOCS.md` — the failure this skill says up front is the main way it fails.
  *"What recurring things deserve a page?"* invites a shrug; *"what do you keep re-explaining?"*
  gets a real list. The other blocks are never read out, because a user answering *"what are your
  services?"* when they have no services is being interviewed by a form.

  *"Who reads it"* is no longer asked at all — it is inferred from the repo and stated as an
  assumption the user can correct.

- **A tutorial — *your first hour*.** Six steps ending in a working base: install, add one source,
  read the diff, ask a question, write scenarios, lint. Deliberately one source at step 2, because
  step 3 is reading the agent's judgment line by line and that is tractable for one document and
  not for ten. The step that matters is turning each disagreement into a checkable rule and
  re-running the scan to watch it take effect.

- **Why the structure is shaped this way**, in Part 2 of the contract and condensed in the README.
  What job each folder name is doing and what it is deliberately not called, why the layers are
  separated by write-ownership, why the three page types are separate (they fail differently), and
  how an agent reads a base without reading all of it. Part 1's *Why "core"* is trimmed to a few
  lines and points here, since rationale does not belong in a file read in full every run.

- **Subfolders in the page layers.** Recommended past roughly thirty pages in a layer, grouped by
  subject, two levels at most. The layer folder still decides what a page is, however deep it
  sits, and wikilinks resolve by slug — so regrouping breaks nothing and is free to do later. A
  summary mirrors its source's path, which is what lets lint keep pairing them.

- **`lint-docs` check 17, slug collisions.** ERROR, and only possible once subfolders exist:
  `topics/auth/tokens.md` and `topics/billing/tokens.md` make every `[[tokens]]` resolve to one of
  them arbitrarily, with no broken-link symptom on the other. Reported with inbound link counts,
  never auto-renamed — two pages wanting one slug are usually one page written twice.

- **A declared index, and `INDEX.md`.** The index is an index, not a readme, and in a project with
  its own root `README.md` a second one a level down competes with it. `docs/INDEX.md` is now the
  common outcome of setup; `docs/README.md` stays valid and is still the default. Either name
  works and the contract treats them identically.

- **The project's root `README.md` gets a section.** One short block: what the base covers, where
  it lives, which commands read it. If the project has no root README, `init-docs` writes a
  minimal one. Nothing else in that file is ever touched — it is the most-read file in the
  repository and it is not the agent's.

- **Step 0 of an upgrade: does the base still describe what it is for?** Every upgrade now re-reads
  the page types and project rules against what the repository has become — page types naming
  things that no longer exist, a preset that stopped fitting, rules nothing has violated in a
  year. Reported, never rewritten: this is the part of an upgrade that needs the owner's judgment,
  since only they know which drift was deliberate.


### Changed


- **`lint-docs` check 15** was *Legacy layout*; it is now *Stale layout paths*. As well as
  detecting a `1.x` `docs/sources/` folder, it now catches `sources:` values and citations that
  point outside the declared root — which is what a root move leaves behind, and which is
  mechanically fixable in exactly the same way. The declared root wins; lint rewrites the paths and
  never moves a folder to make them true.

- **`init-docs` states what the structure does not vary.** The interview says out loud that all
  four presets share the same folders, page types, front matter and commands, and that the choice
  is reversible — because a user who believes they are choosing a layout treats a cheap decision as
  an expensive one, and a user who thinks the schema is fixed abandons the kit rather than editing
  it. The scaffold step says the tree is the same for every project: no folder added because the
  project seems to want one, no layer renamed to match local vocabulary, `scenarios/` kept even
  when question 6 got no answer.


- **The `docs/kb/` case now declares its sources too.** Scaffolding beside a published docs site
  sets the root to `docs/kb/` and the source folder to `docs/`. The published pages are the
  material: they stay where the generator expects them and are cited at their real paths, and
  `docs/kb/core-sources/` is not created at all.

- **`lint-docs` check 15** also catches citations left behind by a moved source folder, alongside
  the moved-root and `1.x` cases it already covered.

- **Hard rule 1 in `AGENTS.md`** and the never-write rules in `scan-docs` and `lint-docs` now name
  the declared source folder rather than the literal path, and cover read-in-place paths too.

- **"Why core" says the name is a default and the role is not.** A project calling it `sources/`
  or `notes/` loses nothing, so long as one folder holds the curated material and the agent never
  writes there. Keeping `core-sources/` is still the recommendation: a reader who has seen the kit
  before knows what it is on sight.


- **The source declaration is always a folder.** There is no glob form, and no way to declare "the
  project root". A declaration that can sweep a repository puts
  `package.json` and every stray file into the provenance chain and extends the immutability rule
  over half the project, which is a lot of hazard for a case a plain `sources/` folder covers.

- **Setup asks less.** One question shapes the base — the preset, plus where sources go on a fresh
  start. The root is no longer a
  question at all: it is `docs/`, and the one exception (a `docs/` a site generator owns →
  `docs/kb/`) is decided from what the survey found and announced rather than offered. Asking
  invited a decision with no upside and a migration if it went wrong. The source location is asked
  only when there is something to point at; a new project gets the default and no question.

- **The README leads with what setup decides**, in a table separating what you choose from what is
  fixed and what is inferred — after which the folder-naming rationale and subfolder rules follow
  in the structure section.

- **The slash-command descriptions no longer hardcode `docs/core-sources/`.** `/scan-docs` reads
  the source folder, which is now a declaration and not necessarily that path.

- **`lint-docs` check 16 covers the index too**: a declared index that does not exist, or both
  `README.md` and `INDEX.md` present and both serving as page lists. Two indexes drift, and an
  agent reading the wrong one silently cannot see half the base.

- **Check 7** accepts subfolders at any depth and now also checks subfolder naming; **check 9**
  reads the index at whatever name it is declared under.

- **The word "human" is gone from everything the kit says.** The rules now name **the owner** where
  authority matters — who may write to a layer, who rules on a contradiction, whose decision
  `claim_type: decision` records — and address you as *you* everywhere else. *Human review* is now
  *Review and ruling*; *"needs a human"* is *"needs you"*. The authority model is unchanged; it
  reads like an assistant talking to the person it works for rather than a specification
  distinguishing two species.


## [2.0.0] — 2026-08-30

An architectural maturity release. `1.0.0` established the loop — sources in, pages out, with
citations. `2.0.0` makes the result auditable: what kind of claim a page carries, where it came
from, whether it is still current, and who is allowed to decide.

**One breaking change** — `docs/sources/` is now `docs/core-sources/`. Everything else is
additive. See *Breaking changes* and *Migration* below.

### Breaking changes

- **`docs/sources/` → `docs/core-sources/`.** The folder holding raw material was renamed, and
  every `sources:` value and citation now points at the new path.

  The prefix names a **role**, not a file type: `core-sources/` is the root of the provenance
  chain, the one layer every page is reproducible from. It also removes a real ambiguity — a
  knowledge base documenting a codebase has two kinds of source, the curated immutable material
  here and `src/**` which the codebase preset reads in place and which changes every commit.
  Calling both "sources" blurred the immutability rule exactly where it mattered most.

  The `sources:` front-matter field keeps its name. It lists source paths; only the paths change.

  `lint-docs` check 15 detects a `1.x` layout and offers the migration, and `scan-docs` checks
  for it before reporting an empty work set — an empty work set on a base full of material is
  the one failure here that looks like success.

### Added

- **Claim types.** `claim_type` distinguishes `fact`, `decision`, `assumption`, `open-question`,
  and `contradiction`. *"Authentication uses Redis"* and *"the team chose Redis"* are different
  claims, and storing them identically is how a knowledge base starts lying. Individual claims
  inside a page are marked with Obsidian callouts.

  Five values, deliberately. `hypothesis` was folded into `assumption` — a distinction nobody
  applies consistently makes the field unreliable rather than merely coarse. `instruction` was
  dropped as a category error: a rule for how work is done here belongs in `DOCS.md`, and a page
  carrying one is a second, unenforced home for it.
- **Lifecycle.** `status` tracks `draft`, `active`, `stale`, and `superseded`, with
  `superseded_by` naming the replacement. Obsolete knowledge no longer looks identical to
  current knowledge.

  Four values, deliberately. `deprecated` and `archived` were dropped: *"no longer in use"* is a
  fact about the world that belongs in the page body with a citation, and retiring a page from
  view is a filesystem decision. Both were easy to confuse with `superseded`.
- **A two-value authority rule.** `claim_type: decision` and `status: superseded` are the only
  values an agent may never set. Everything else it can reach on its own evidence.
- **An authority model.** An explicit source-of-truth hierarchy, and the rule it enforces: the
  agent discovers, summarizes, classifies, and proposes; the human decides. Agents may not set
  `claim_type: decision` nor `status: superseded`. `confidence: high` is not authority.
- **A table-cell escaping rule.** A wikilink's pipe must be written `\|` inside a table cell;
  unescaped it splits the cell and silently breaks the table. Found by running the scan loop
  against the example sources and comparing the result against the shipped pages, which had the
  bug.
- **Provenance anchors.** A per-format ladder — page for PDFs, section for specs, line range for
  code, timestamp for transcripts — with a graceful fallback to the bare file name and a
  standing prohibition on inventing an anchor that was not seen.
- **A retrieval and context contract.** Selection by index and links, capped at two hops, with
  scope flags `--topic`, `--entity`, and `--source`. A context budget is estimated before
  reading, and exceeding it is a graceful failure that asks for a narrower scope. Never a silent
  truncation.
- **Answer states.** Answers separate `known`, `inferred`, `contradicted`, and `unknown`.
  *"I don't know based on the available knowledge"* is a correct answer.
- **A documented human-review workflow.** The path out of `contradiction` and `open-question`,
  in a Decision / Rationale / Evidence shape that keeps the losing evidence on the page. A
  ruling whose reasons have been deleted is not auditable.
- **Git conflict handling.** Agents never auto-resolve a semantic conflict in a page. Git knows
  that two people wrote different text; it does not know which is true. `docs/CHANGELOG.md` and
  `docs/README.md` are the two exceptions, because neither carries meaning.
- **Security exclusions.** A default deny-list — `.env*`, `*.pem`, `*.key`, `secrets/`,
  `credentials/`, `private/`, `.ssh/`, `.aws/` — that `scan-docs` checks before reading and
  `lint-docs` checks after. A source found to contain credentials halts the scan rather than
  being redacted, and an optional `sensitivity` marker documents what a page holds.
- **`test-docs`, a fifth operation** (alias `/test`). `lint-docs` checks that the base is
  well-formed; `test-docs` checks that it is still right. Individual cases are called
  **scenarios**, they live in `docs/scenarios/questions.yaml`, and they score
  on citation validity, source coverage, required facts, forbidden claims, and answer state.
  Structural checks and semantic ones are reported separately.
- **Three domain presets** — codebase, outside research, operations and runbooks. Interview
  question 1 already asked what the knowledge base was for and did nothing structural with the
  answer; now it selects a starting shape for page types and project rules. The contract is
  identical across all three: same schema, same claim types, same lifecycle. They differ only in
  what a page is about and which rules come pre-written, and `init-docs` deletes the unused ones
  so a project's contract carries exactly one page-type table.
- **`examples/example-project/`.** Two sources that disagree, four generated pages, three
  scenarios, and suggested edits that should each produce a specific finding.
- **`examples/example-project/TRANSCRIPT.md`.** What `ask-docs`, `lint-docs`, and `test-docs`
  actually output against that base — the contradicted answer, a clean lint holding the
  contradiction open as INFO, and three passing scenarios. The generated pages already showed
  what `scan-docs` produces; the other three operations were documented but never demonstrated,
  which is a poor look for a kit arguing evidence over assertion.
- **`CHANGELOG.md`** — this file.

### Changed

- **`docs/DOCS.md` is now the knowledge governance contract** rather than a formatting schema.
  It defines claim types, lifecycle, authority, provenance, freshness, retrieval limits, answer
  states, human review, git conflicts, and security, alongside the page and naming rules it
  already carried.
- **`lint-docs` gained severity levels and grew from 8 checks to 15.** ERROR is reserved for
  mechanically certain faults, so a merge gate can depend on it. An open contradiction is INFO,
  never an ERROR — it is the system working. New checks cover malformed front matter, invalid
  field values, agent-set human-only values, broken citations, and excluded material.
- **`lint-docs` handles malformed YAML as a finding, not a failure.** One unparseable file is
  one ERROR; the sweep continues. Partial metadata from a block that failed to parse is never
  ingested, and a block that could not be read is never silently rewritten.
- **`ask-docs` no longer reads broadly by default.** Selection is explicitly capped, budgeted,
  and reported: answers close with what was and was not consulted.
- **`scan-docs` types claims and anchors provenance**, records anchors while reading, and leads
  its report with what only a human can settle.
- **`init-docs` scaffolds `docs/scenarios/questions.yaml`**, asks what must never be read separately
  from what is out of scope, and lists the governance sections it must copy through unchanged.
- **`AGENTS.md` grew from 6 hard rules to 11**, and gained a repository map so an agent does not
  rediscover the architecture each session.
- **`docs/CHANGELOG.md`** now logs `test-docs` runs alongside the other three.

### Migration

`/init-docs` detects an existing install and runs this as an upgrade rather than an interview.
By hand, in order:

**1. Update the kit files.** `.claude/` and the root `AGENTS.md` are kit-owned — replace them
wholesale, there is nothing of yours in them. A plugin update does this; a copied install means
re-copying. **Do not touch `docs/` in this step.**

**2. Rename the sources folder:**

```bash
git mv docs/sources docs/core-sources
```

**3. Upgrade `docs/DOCS.md` — start from the new file, not yours.** Your `DOCS.md` was written
from an `init-docs` interview and holds three sections you authored: **page types**,
**project-specific rules**, and **out of scope**. Copy the shipped 2.0 file over yours and port
those three back verbatim, appending your out-of-scope paths to the security exclusions rather
than replacing them. Then delete the preset sections.

Not the reverse. 2.0 adds roughly a dozen governance sections, and merging those into a 1.x file
by hand is where the mistakes live; carrying three sections the other way is mechanical.

**4. Run `/lint-docs`.** Check 15 rewrites the `sources:` values and citations still pointing at
`docs/sources/`, and check 2 adds `status: active` plus a defaulted `claim_type` to every page
written before those fields existed — `fact` on a summary, `open-question` on a topic or entity
it cannot classify, never `decision`.

Order matters: rewriting paths *before* the folder moves breaks every citation in the base. If
both folders somehow exist, lint stops and asks — a half-finished migration needs a human to say
which file wins.

Expect a large diff and plenty of WARNINGs on a base of any size. That is the upgrade landing,
not breakage. **Commit before you start.**

**5. Optional — `docs/scenarios/questions.yaml`.** New and empty by default. Without it,
`test-docs` has nothing to run and says so.

**Your pages need no editing.** No command was renamed for `1.x` users and no field was removed;
`test-docs` and `docs/scenarios/` are new here, never having shipped under another name. Pages
missing `status` or `claim_type` stay valid either way — if you skip step 4 entirely, they are
upgraded as they are next touched.

### Known limitations

Stated plainly, because documentation that claims more than it does is the failure this kit
exists to prevent.

- **Nothing here executes.** All five operations are Markdown instructions an agent follows.
  There is no CLI, no parser, no test suite, and **nothing that can run in CI.**
- **`lint-docs` and `test-docs` are therefore not deterministic.** Two runs may word the same
  finding differently or disagree at the margin. The structural checks are far more stable than
  the semantic ones, because they compare paths and strings rather than meaning — but "more
  stable" is not "reproducible," and neither can gate a merge on its own.
- **The context budget is an estimate**, at roughly 4 characters per token. It is deliberately
  conservative and it will sometimes refuse a question that would have fit.
- **Retrieval is index-and-links.** It scales far past reading everything, and it depends on
  `docs/README.md` being accurate and pages being linked. An unlinked page not in the index is
  invisible to `ask-docs` — which is why `lint-docs` reports orphans.
- **The security exclusions are instructions, not enforcement.** Nothing in a Markdown file can
  stop an agent from reading a path. Use filesystem permissions and `.gitignore` for anything
  that actually matters.

## [1.0.0] — 2026-08-30

Initial release.

- Four operations — `init-docs`, `scan-docs`, `ask-docs`, `lint-docs` — as agent-agnostic
  Markdown skills, with slash-command aliases.
- `docs/` structure: `core-sources/` (human-owned, immutable to agents), `summaries/`, `topics/`,
  `entities/`, plus an index and an append-only changelog.
- `docs/DOCS.md` as a per-project schema, written by `init-docs` from an interview.
- Front matter: `type`, `title`, `created`, `updated`, `sources`, `confidence`.
- Obsidian-compatible wikilinks, so `docs/` opens as a vault without conversion.
- Contradiction callouts, and the rule that contradictions are never silently resolved.
- `AGENTS.md` as the entry point for any filesystem-capable agent.

[2.3.0]: https://github.com/fentonmartin/llm-starter-kit/releases/tag/v2.3.0
[2.0.0]: https://github.com/fentonmartin/llm-starter-kit/releases/tag/v2.0.0
[1.0.0]: https://github.com/fentonmartin/llm-starter-kit/releases/tag/v1.0.0
