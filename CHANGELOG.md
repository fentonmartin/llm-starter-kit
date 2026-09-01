# Changelog

Release history for `llm-starter-kit` itself. The log of what an agent did *inside* a knowledge
base is `docs/CHANGELOG.md`, which is a different file with a different purpose.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versioning is [semantic](https://semver.org/spec/v2.0.0.html), applied to the schema and the
command surface: a breaking change is one that invalidates an existing knowledge base.

## [2.1.0] — 2026-09-01

One structure, more use cases. `2.0.0` fixed what a page must contain; `2.1.0` answers the
question that follows — *what if my project isn't a codebase?* — without giving each use case its
own layout.

**Nothing breaks.** No folder, field, command, or check was renamed or removed. Existing bases are
current as they stand: the root defaults to `docs/`, and a `DOCS.md` with no root line behaves
exactly as before.

### Added

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

[2.1.0]: https://github.com/fentonmartin/llm-starter-kit/releases/tag/v2.1.0
[2.0.0]: https://github.com/fentonmartin/llm-starter-kit/releases/tag/v2.0.0
[1.0.0]: https://github.com/fentonmartin/llm-starter-kit/releases/tag/v1.0.0
