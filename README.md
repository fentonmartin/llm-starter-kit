<div align="center">

# 📚 llm-starter-kit

**A persistent, governed knowledge layer for LLMs and AI agents.**

Drop it into any repo. It builds a `docs/` folder that every agent reads, maintains, and
answers from — where every claim traces to a source, disagreements stay visible, and obsolete
knowledge is marked as obsolete.

`init-docs` · `scan-docs` · `ask-docs` · `lint-docs` · `test-docs` · `help-docs` · `all-docs`

[![version](https://img.shields.io/badge/version-2.1.0-blue?style=flat-square)](CHANGELOG.md)
[![license](https://img.shields.io/badge/license-MIT-green?style=flat-square)](LICENSE)

**Latest: `2.1.0`** — one structure for every use case, a tutorial, `/all-docs` and `/help-docs`,
a commit setting, and the reasoning behind every folder name.
[What changed](CHANGELOG.md#210--2026-09-01) · [Tutorial](#tutorial-your-first-hour) · [Upgrading](#already-using-an-older-version)

[**Quick start**](#quick-start) · [**Tutorial**](#tutorial-your-first-hour) · [Knowledge model](#knowledge-model) · [Retrieval](#retrieval-and-context-limits) · [Governance](#contradictions-and-review) · [Any AI agent](#works-with-any-ai-agent) · [Limitations](#limitations)

</div>

---

## Overview

Every agent that opens your repo starts blind. It greps around, half-understands the
architecture, and gives you a confident answer built on a partial reading. Next session, it
does the whole thing again.

This kit makes the repository the durable knowledge layer instead. The thinking happens *once*,
when material comes in, and the result is Markdown you can read, diff, and correct.

**The core principle: knowledge should survive changing models and agents.** The agent is not
the owner of what the project knows. The repository is.

```
Sources  →  Scan  →  Knowledge  →  Governance  →  Retrieve  →  Ask  →  Evidence-backed answer
```

### What it is not

Not a RAG framework, not a LangChain alternative, not a vector database wrapper, not an agent
orchestrator, not a hosted service. There is no database, no index to rebuild, no server, and no
runtime dependency. Markdown, YAML front matter, git, and the filesystem are the entire stack.

---

## Why

You could point an agent at a vector database. Retrieval is the easy half of the problem, and it
leaves the hard half untouched:

| Retrieval alone gives you | It does not give you |
|---|---|
| Relevant chunks | Whether the chunk is a fact, a proposal, or somebody's assumption |
| A similarity score | Whether you ever approved it |
| The nearest match | Whether two sources disagree, or which one you got |
| An answer | Whether the source changed after the answer was written |
| Embeddings | Anything you can read, review in a pull request, or fix by hand |

A retriever asked *"what is the session TTL?"* returns the more relevant of two conflicting
documents and answers confidently. **The confidence is the bug.** This kit records both values,
marks the page `claim_type: contradiction`, and says nobody has ruled.

The trade-off is honest: scanning costs time and tokens up front, and there is more structure to
learn. What you get back survives. A vector index is not something you can read.

See [`examples/example-project/`](examples/example-project/) — that exact contradiction, worked
end to end, in about five minutes of reading.

---

## Quick start

Commit first. A scan touches many files, and the diff is how you review what the agent did:

```bash
cd your-project
git commit -am "checkpoint"
```

### 1. Paste this into your agent

**Works everywhere** — Claude Code, opencode, Codex, Cursor, Windsurf, or anything else with
file access. Open the project you want documented and paste this into the chat box:

```
Set up the llm-starter-kit documentation system in this project.

1. Clone https://github.com/fentonmartin/llm-starter-kit into a temp folder
   outside this repo.
2. Copy its .claude/ and AGENTS.md into this project, overwriting neither if
   they already exist — append instead, and tell me what you appended.
3. Copy its docs/ ONLY if this project has no docs/ folder. If I already have
   one, leave it completely alone: init-docs reorganizes it in step 4, and copying
   a template over it is the one thing that would lose my work.
4. Read .claude/skills/init-docs/SKILL.md and follow it exactly: survey this
   project first, tell me whether this is a fresh start or a merge, then
   interview me, then write docs/DOCS.md from my answers.
5. Delete the temp folder.

Do not skip the interview and do not answer its questions on my behalf.
Show me the docs/DOCS.md rules you wrote before we go any further.
```

That's the whole setup. Afterwards, **`/help-docs`** tells you where you are and what to do next
at any point — it's the one command you don't need to remember anything to use.

<details>
<summary><b>Copy the files yourself instead</b>, and day-to-day prompts</summary>

<br>

```bash
git clone https://github.com/fentonmartin/llm-starter-kit /tmp/lsk
cp -r /tmp/lsk/.claude /tmp/lsk/AGENTS.md your-project/

# the template docs/ folder ONLY if you don't have one —
# cp would merge into an existing docs/ and overwrite files
[ -d your-project/docs ] || cp -r /tmp/lsk/docs your-project/

rm -rf /tmp/lsk
```

Already have a `docs/`? Skip that line entirely and let `/init-docs` merge it — that's a
[supported path](#fresh-start-or-merge), and copying the template over it is the one move that
would cost you work.

Then: *"Read `.claude/skills/init-docs/SKILL.md` and follow it exactly. Survey this project
first, then interview me before writing anything."*

Each skill is a plain Markdown file, so pointing an agent at one is the same as running the
command — `Read .claude/skills/scan-docs/SKILL.md and follow it.` Most agents don't need this
much hand-holding once `AGENTS.md` is in the repo; "scan my sources" or "what do the docs say
about X" usually routes correctly on its own.

</details>

### 2. Or install as a Claude Code plugin

**Install** once — it is then available in every repo:

```
/plugin marketplace add fentonmartin/llm-starter-kit
/plugin install llm-starter-kit
```

**Set up** from inside the project you want documented:

```
/init-docs
```

It reads your README, your manifest, and your layout, tells you whether this is a
[fresh start or a merge](#fresh-start-or-merge), then asks **five or six questions — and most of
them depend on your first answer.**

Two for everyone:

> - **What kind of project is this?** A codebase · documents you write · a knowledge centre of
>   material from elsewhere · a system you operate
> - **Where should sources go** — `docs/core-sources/` or a top-level `sources/`? *(fresh start
>   only; a merge points at what you already have)*

Then **two questions in your own vocabulary**, not a generic checklist. You're only asked the pair
that fits:

| If you said… | You're asked |
|:--|:--|
| **A codebase** | What units would someone look up by name — services, tables, endpoints, jobs? · What do you keep re-explaining, and where does the code contradict the docs? |
| **Documents you write** | What are the documents, and what subjects cut across them? · Which terms mean something specific here, and who rules when two documents disagree? |
| **A knowledge centre** | What are you actually trying to answer, and whose material is this? · Which sources don't you trust, and what gets quoted at you as fact? |
| **A system you operate** | Which systems, and what do people get paged for? · Which values drift, and where does the spec differ from production? |

And two at the end:

> - **What must never be read?** *(beyond the built-in `.env*`, `*.pem`, `secrets/`)*
> - **What would you be most embarrassed to get wrong?** *(these become your scenarios)*

It doesn't ask who reads it — that's inferred from your repo, and it'll tell you what it assumed.
Two more questions come *after* setup is written, where they can't crowd out one that shapes the
base: [what to call you, and whether it may commit](#what-it-never-asks).

The first answer picks a **preset** — the starting shape for your page types and rules:

| If you said… | Preset | Pages are one per… |
|:--|:--|:--|
| **a codebase** | **codebase** | module · decision or flow · service, table, endpoint, env var |
| **documents you write** | **document library** | document · subject or theme · person, org, product, term |
| **a knowledge centre** — papers, articles, vendor docs | **outside research** | paper · question or debate · person, lab, model, dataset |
| **a system you operate** — incidents, on-call | **operations** | incident or runbook · procedure or failure mode · service, alert, config key |

**Same structure for all four.** Same folders, same three page types, same front matter, same
claim types, same lifecycle, same commands. A preset is not a layout — it supplies the
starting nouns and a handful of pre-written rules, and nothing else. `/init-docs` takes one, swaps
in your actual nouns, and deletes the rest.

That means the choice is cheap and reversible. Picking wrong costs you a `DOCS.md` edit, not a
migration — see [As the project grows](#as-the-project-grows).

### Fresh start or merge?

`/init-docs` starts by looking at your repo and telling you which of two situations it's in.
Everything else follows from that.

|  | **Fresh start** | **Merge** |
|:--|:--|:--|
| **When** | No documentation, nothing filed yet | You already have docs, notes, or a pile of PDFs |
| **Sources** | **You choose** — see below | Points at wherever the material already is. **Nothing moves.** |
| **Your files** | None to worry about | Classified, then pointed at or moved — never overwritten, never deleted, and every move shown as `old → new` first |

When in doubt it treats your project as a merge, because assuming a fresh start is the one mistake
here that costs real work.

**On a fresh start you pick where sources go.** Two options, and this is the only layout decision
you make:

```
A.  docs/core-sources/     everything under docs/ — one folder to copy, or open as an Obsidian vault
B.  sources/               top level, beside docs/ — more visible, easier to drop files into
```

Identical behaviour either way: the agent reads it and never writes to it. It's a one-line change
later if you pick wrong. Take **A** if you have no preference — but take **B** without guilt if
you'll actually use it, because a visible folder you fill beats a tidy one you don't.

**On a merge, there's usually nothing to decide.** If your material is in `research-papers/`, the
base points at `research-papers/`. You keep filing where you file today. You're only asked when
material is scattered across two or more places, because there's exactly one source location and
someone has to pick.

### What it never asks

| | What happens |
|:--|:--|
| **The root folder** | Always `docs/`. One exception, decided for you: if your `docs/` is built by a site generator, the base goes to `docs/kb/` and your published docs become the sources. You're told, not asked. |
| **The index filename** | `docs/INDEX.md` — it's an index, not a readme, and your project already has a root `README.md`. `docs/README.md` only when your project has none. |
| **Your root `README.md`** | Gains one section saying the base exists and which commands read it. Nothing else in it is ever touched. **No README at all?** It asks before creating one, shows you the draft, and the project description is a single line marked as its guess — it won't invent a purpose, a usage section, or a feature list for a project it met ten minutes ago. |
| **The folder layout** | Fixed. Same five folders in every project — see [why](#why-these-names). |

Two things *are* asked at the end, once setup is done and neither can crowd out a question that
shapes the base.

**What to call you.** It reads `git config user.name` and offers it — *"Git has you as Fenton
Martin — shall I call you Fenton, or something else?"* Whatever you answer is used from then on;
decline and it says *"you"* and never asks again. It won't take a name from a commit log without
asking, since a repo's authors often aren't the person sitting there.

**Commits:**

```
Commits: none        the default. files are written, you review and commit
         per-run     one commit per command run
         per-file    one commit per page. noisy, individually revertable
```

`none` isn't timidity. Reading `git diff` after a scan is how you find where the agent's judgment
differs from yours, and auto-committing doesn't remove that review — it removes the *moment* that
prompts it. Turn it on once your rules have stopped changing. At any setting the agent stages only
what the run wrote (never `git add -A`, which would sweep up your work in progress) and **never
pushes**.

So the interview is really one question — the preset — plus *"where should sources go?"* on a
fresh start.

One more thing worth knowing: material that should be **cited but not filed** — a source tree,
docs published by a generator — goes in **read-in-place sources**. No summary pages, and citations
carry a commit hash.

Out comes a `docs/` folder and a `docs/DOCS.md` **written for your project**, in your
vocabulary.

**Use it:**

```
/scan-docs                      # read new material, write the pages
/ask-docs how does auth work?   # answer from the pages, with citations
/lint-docs                      # gaps, contradictions, stale pages, schema errors
/test-docs                      # check the base still answers its known questions
/all-docs                       # scan, lint and test in one pass
/help-docs                      # what state is this in, and what should I do next?
```

> [!TIP]
> After your first `/scan-docs`, run `git diff`. It's the most useful fifteen minutes you'll
> spend with this kit — you see exactly where the agent's judgment differs from yours, and
> that's what the rules in `docs/DOCS.md` exist to correct.

### What a merge actually does

Never overwrites, never deletes — and which of two things happens depends on where your material
lives:

- **Material in its own folder** — `notes/`, `research-papers/`, `contracts/`. Nothing moves at
  all. The base points at that folder and reads it there. This is most projects.
- **A `docs/` folder with a mix of things in it.** That's the case below: classified one file at a
  time, every move shown as `old → new` first.

<details>
<summary>What the merge does</summary>

<br>

```
BEFORE                          AFTER
docs/                           docs/
├── architecture.md             ├── DOCS.md          ← the schema
├── setup-guide.md              ├── INDEX.md         ← the index
├── vendor-api-spec.pdf         ├── topics/
└── old-notes.md                │   ├── architecture.md      ← kept, front matter added
                                │   └── setup-guide.md
                                ├── core-sources/
                                │   ├── vendor-api-spec.pdf  ← raw material
                                │   └── old-notes.md         ← unclassifiable → safe here
                                ├── summaries/
                                ├── entities/
                                └── scenarios/
```

Anything it can't confidently classify goes to `core-sources/`, where the next scan turns it into
proper pages — **nothing is lost by that default**. It shows you every move as `old → new`
before touching a thing, then fixes inbound links across the repo.

If your `docs/` is built by a site generator (`mkdocs.yml`, `docusaurus.config.js`,
`book.toml`), it won't rearrange anything — it scaffolds at `docs/kb/`, points the schema's root
line there, and treats your published docs as sources.

</details>

### Already using an older version?

Commit first, then paste this into your agent:

```
Upgrade the llm-starter-kit installation in this project to 2.1.

1. Clone https://github.com/fentonmartin/llm-starter-kit into a temp folder
   outside this repo.
2. Read its .claude/skills/init-docs/SKILL.md, section "7. Upgrading an
   existing install", and follow it exactly.
3. Delete the temp folder when you are done.

Rules for this upgrade:
- .claude/ and AGENTS.md are kit files. Replace them wholesale.
- docs/ is MY knowledge base. The only file you may change there is
  docs/DOCS.md, plus whatever /lint-docs fixes at the end.
- docs/DOCS.md holds three sections I wrote: page types, project-specific
  rules, and out of scope. Start from the new DOCS.md and port those three
  into it — do not merge the new sections into my old file. Show me the
  diff before you write it.
- If I am on 1.x: rename docs/sources to docs/core-sources BEFORE rewriting
  any paths. Skip this if the folder is already named core-sources.
- Then run /lint-docs and show me what it changed.
- Two things are new and you should ask me, not assume: whether you may
  commit your own work (default: no), and what to call me.

Tell me what version I was on before you start, and stop if anything is
ambiguous rather than guessing.
```

It also re-reads your `DOCS.md` against what your repo has become and tells you if the base has
drifted from what it says it is for — page types naming things that no longer exist, a preset that
stopped fitting. It reports that; it never rewrites it.

**Installed as a plugin?** Run `/plugin update llm-starter-kit` first, then paste the same prompt
with steps 1 and 3 removed — the plugin has already replaced the kit files for you.

**Already on 2.0?** Nothing in your `docs/` has to change — every declaration defaults to where
your base already keeps things. Replace the kit files, then let it add the four new lines to your
`DOCS.md` (root, source folder, index, commits) and ask you the two questions above. What 2.1 adds
— a fourth preset, `/all-docs`, `/help-docs`, subfolder grouping, the migration guide — is then
there when you want it.

**Why the prompt is that specific about `docs/DOCS.md`:** it's the only file in this kit you
authored — written from your `init-docs` interview. 2.0 adds about a dozen governance sections,
and merging those into your old file is where mistakes happen; carrying your three sections into
the new file is mechanical and you can check it at a glance.

Everything else is automatic. `/lint-docs` rewrites the `sources:` paths and adds `status` and
`claim_type` to pages written before those fields existed. **Your pages need no editing.** Expect
a large diff and plenty of WARNINGs on a base of any size — that's the upgrade landing, not
breakage.

Prefer to drive it yourself? The manual steps are in [CHANGELOG.md](CHANGELOG.md#migration).

---

## Commands

| Command | What it does | When |
|:--|:--|:--|
| **`/init-docs`** | Surveys the project, interviews you, scaffolds `docs/`, writes a schema for *this* codebase | Once, at the start |
| **`/scan-docs`** | Reads new sources in full, writes a summary each, updates every topic and entity they touch, types the claims, flags contradictions | When material piles up |
| **`/ask-docs`** | Answers from the pages within a context budget, with citations and an explicit known / inferred / contradicted / unknown split | Instead of digging |
| **`/lint-docs`** | 17 checks at three severities — schema, citations, staleness, orphans, gaps, contradictions. Fixes the mechanical, escalates the rest | Every few scans |
| **`/test-docs`** | Runs `docs/scenarios/questions.yaml` and reports what the base no longer answers correctly | After big changes |
| **`/help-docs`** | What this base is, what state it's in, and the one thing worth doing next | When you're not sure |
| **`/all-docs`** | Scan, lint, test in order — skips what has nothing to do, reports once | Sitting down to the docs |

`/init` `/scan` `/ask` `/lint` `/test` `/all` work as short aliases. The long names are canonical
because this installs globally — a bare `/lint` would collide with code linters and misfire on
*"lint my code."* There's deliberately **no `/help` alias**, for the same reason: your agent almost
certainly has its own.

**Not sure where to start?** `/help-docs` reads your declarations and counts your files — it never
opens a page, so it's cheap — then tells you what state the base is in and the single next step:

```
Knowledge base: docs/          (codebase — one page per service, decision, endpoint)
Sources:        docs/core-sources/       14 files · 3 unread
Pages:          9 summaries · 6 topics · 4 entities
Scenarios:      none yet

Next: 3 sources are waiting. /scan-docs reads them, or /all-docs does the whole pass.
```

It writes nothing, and it won't invent work — *"nothing is owed"* is an answer it's allowed to
give.

**Been away a while?** `/all-docs` does the whole pass in the order the stages depend on each
other — scan new material, lint what that changed, test whether the base still answers — skipping
any stage with nothing to do:

```
14 files in docs/core-sources/, 3 with no summary.
Plan: scan 3 → lint → test 5 scenarios.
Scanning reads sources in full, so this is the expensive part.
```

It tells you the cost before it starts, and **stops to ask if the scan is large** — more than about
ten unread sources and it offers to do a batch instead. At the end you get *one* report, leading
with what needs you:

```
Needs you (3):
  · Contradiction — session TTL. 30 minutes vs 24 hours. Both sides recorded.
  · Scenario failed — "session-revocation" expected unknown, got known.
  · Gap — [[rate-limiting]] has 5 inbound links and no page.

Ran:  scan 3 sources · lint 0 errors 4 warnings · test 4/5 pass
```

It orchestrates the real commands rather than reimplementing them, and it gains no authority by
running them together: contradictions and your-decision-only fields are as untouchable in a pass as
anywhere. It never runs `/ask-docs` — a question is yours to ask.

These are [Agent Skills](.claude/skills/), so plain description works too: *"document this
repo"* runs `init-docs`.

**Scope flags** for `ask-docs`, when a question would otherwise pull in too much:

```
/ask-docs what is the TTL? --topic session-management
/ask-docs what is the TTL? --source docs/core-sources/260710-ops-runbook.md
/ask-docs where is Redis used? --entity redis
```

Every command also takes a **root**, for the rare project with more than one knowledge base —
`/scan-docs research/`. With no root, it's `docs/`. See [As the project grows](#as-the-project-grows).

---

## Tutorial: your first hour

A walkthrough with a real base at the end of it. Use your own repo, or an empty folder and three
PDFs you've been meaning to read — the shape is the same either way.

### 1. Install and set up · ~10 min

Paste the [quick start prompt](#1-paste-this-into-your-agent), answer the interview, and read the
`DOCS.md` it writes you. **Read the page types table and the project-specific rules properly** —
those two sections are the whole product of the interview, and correcting them now costs a
sentence. Correcting them after fifty pages costs fifty pages.

```
docs/
├── DOCS.md          ← read this. it's yours, not the kit's
├── INDEX.md         ← empty
├── core-sources/    ← empty
└── summaries/  topics/  entities/  scenarios/
```

### 2. Add one source, not ten · ~5 min

Drop **one** document into `docs/core-sources/`. One, deliberately: you're about to review the
agent's judgment line by line, and that's tractable for one source and not for ten.

Name it with a date prefix if it has a meaningful date — `260415-auth-spec.pdf`. The kit uses
`YYMMDD` so files sort chronologically in any file browser.

Two things it will refuse rather than fake, and both are the system working: a source **too large
to hold** gets reported with options — split it, name a page range, or leave it — instead of a
summary of the part it managed to read; and a source it **can't read at all**, like a scanned PDF
with no text layer, produces no page and stays flagged as unread. An unread source is a known gap.
An invented summary is a false one nothing downstream can catch.

```
/scan-docs
```

### 3. Read the diff · ~15 min

```
git diff
```

**This is the most valuable quarter-hour you'll spend with this kit.** You're looking for the gap
between the agent's judgment and yours:

- Did it split the source into the *topics you'd have chosen*? Wrong topic boundaries are the most
  expensive thing to fix later.
- Are the `claim_type` values right? A *"we chose Redis"* recorded as `fact` rather than `decision`
  is the single most common error, and it's how a base starts quietly lying.
- Did it wikilink concepts that deserve their own page — including pages that don't exist yet?
  Those links aren't broken, they're your to-do list.
- Anything asserted without a citation?

Every disagreement you find is a rule waiting to be written. Add it to the
**project-specific rules** in `DOCS.md`, in language that's checkable — *"figures carry their unit
and as-of date"*, not *"be precise"*. Then re-run `/scan-docs` on the same source and watch the
rule take effect.

### 4. Ask it something · ~5 min

```
/ask-docs what does the spec say about session expiry?
```

Read the answer's shape, not just its content. Every answer is labelled **known / inferred /
contradicted / unknown**, and cites the pages it used. An `unknown` is the feature working: it
means the base doesn't know, and it told you instead of guessing.

Now ask something the source *doesn't* cover. You should get `unknown`, not a plausible paragraph.
If you get the paragraph, that's a finding worth reporting.

### 5. Write down what must not break · ~10 min

Open `docs/scenarios/questions.yaml` and write three cases — questions the base must keep answering
correctly. Include one whose honest answer is *"nobody knows yet"*:

```yaml
questions:
  - id: session-ttl
    question: What is the session TTL?
    expect_sources:
      - docs/core-sources/260415-auth-spec.pdf
    require_facts:
      - "24 hours"
    expect_state: known

  - id: revocation
    question: How are sessions revoked on password change?
    expect_state: unknown        # the sources don't say. that's the point.
```

```
/test-docs
```

These are the regression tests for your knowledge. They're worth more than they look: the failure
this kit exists to prevent is a base that still passes every schema check while quietly answering
wrongly.

### 6. Add the rest, then lint · ~15 min

Now add the other documents and scan them. With several sources in, run:

```
/lint-docs
```

Expect findings — a new base always has gaps, and gaps are information. Read them by severity:
**ERROR** is mechanically wrong and worth fixing now; **WARNING** is tolerable; **INFO** wants your
judgment and includes the contradictions, which are the most interesting output this kit produces.

### Then what

A rhythm, not a project: sources in as they arrive, `/scan-docs` when a few have piled up,
`/lint-docs` every few scans, `/test-docs` after anything structural. See
[A rhythm that works](#a-rhythm-that-works).

Come back in three weeks having forgotten all of it? **`/help-docs`** tells you what state the base
is in and the one thing worth doing next, and **`/all-docs`** just does the pass.

Two things to expect as it grows. Layers pass thirty pages and want
[subfolders](#directory-structure). And your `DOCS.md` rules keep accumulating — that's the base
learning, and it's the point.

---

## Directory structure

```
README.md                you are here
AGENTS.md                agent entry point — 13 hard rules, and a pointer to the schema
CHANGELOG.md             this kit's release history
LICENSE
docs/
├── DOCS.md              ⚖️  the governance contract — read below
├── INDEX.md             🗂️  the index. every generated page, once. (or README.md)
├── CHANGELOG.md         📜  append-only log of every run
├── core-sources/        📥  raw material. yours. read-only to the agent.
├── summaries/           📄  one page per source
├── topics/              💡  one page per idea, decision, or flow
├── entities/            🏷️  one page per service, table, person, product…
└── scenarios/           ✅  questions.yaml — what this base must answer correctly
examples/example-project/  a complete worked example, five minutes to read
.claude/skills/          the seven skills, as plain Markdown
.claude/commands/        the seven, plus short aliases
.claude-plugin/          install this repo as a Claude Code plugin
```

The `docs/` folder here is the template. `/init-docs` reproduces it inside your project.

**Every project gets this same tree**, whichever preset it picked — a codebase, a reading pile, a
runbook set and a library of documents are laid out identically. One shape means one thing to
learn, one set of habits, and a base you can lift into another project intact. What varies between
use cases is what a page is *about*, and that lives in the page types table in your `DOCS.md`.

Everything sits under `docs/` so the knowledge base is one self-contained folder you can copy
anywhere. `docs/core-sources/` is nested for filing only — it stays read-only to the agent and
exempt from every page rule.

`docs/DOCS.md` Part 1 opens by declaring three paths, and every skill resolves against them rather
than assuming:

```
> Root: docs/
> Source folder: docs/core-sources/
> Index: docs/INDEX.md
> Commits: none
> Address: (not set)
```

The last two aren't paths: **Commits** is whether the agent commits its own work, and **Address**
is what it calls you. Both are asked once at setup and are a one-line change afterwards.

The root is not a choice and the index name follows a rule — see
[what it never asks](#what-it-never-asks). The source location is the one you pick, on a
[fresh start](#fresh-start-or-merge).

### Why these names

Every folder name is doing a job, and the jobs are what the rules protect:

| | Why not something else |
|:--|:--|
| `core-sources/` | Not `sources/` — "source" is already the `sources:` field, the citation, and source code. The `core-` prefix names a **role**: the root of the provenance chain, the one layer everything else is reproducible from. It also separates curated material from read-in-place paths, which is exactly where immutability has to be sharp. |
| `summaries/` | Not `notes/`. One source, one job, and the name says both. The only page type allowed to paraphrase at length, because faithfulness to one document is what it's *for*. |
| `topics/` | Not `concepts/`. A topic outlives any one source and is where sources are compared — so it's the only layer where a contradiction can live. |
| `entities/` | Not `glossary/`. The point is the backlinks: one thing, stable facts, and every place it appears. A glossary implies definitions only. |
| `scenarios/` | Not `tests/`. Nothing executes here, and calling it tests promises a green tick this kit can't give. |
| the index | Not generated. It's the entry point an agent reads before anything else, maintained in the same pass that changes a page. |

**And why they're separate at all:** who may write to a file is the only durable way to keep
provenance honest. Sources are yours and immutable, pages are agent-maintained and derived,
scenarios are yours, bookkeeping is agent-owned. Merge any two and *"where did this claim come
from?"* stops having an answer. The three page types are separate for a different reason — they fail
differently, and keeping them apart is what makes a lint finding specific enough to act on.

Full version in [`docs/DOCS.md`](docs/DOCS.md) Part 2, *Why the structure is shaped this way*,
including how an agent reads a base without reading all of it.

### Subfolders

**Group a layer once it passes ~30 pages.** Below that, flat is easier to scan.

```
docs/topics/
├── auth/
│   ├── session-management.md
│   └── token-rotation.md
└── billing/
    └── invoice-generation.md
```

- **By subject, never by date or type.** `topics/auth/`, not `topics/2026/`.
- **The layer folder still decides what a page is.** Everything under `topics/` is a topic page,
  however deep. Never nest one layer inside another.
- **Two levels is the limit.** Deeper and the path becomes a taxonomy nobody maintains.
- **Slugs stay unique across the whole layer.** `auth/tokens.md` and `billing/tokens.md` make
  `[[tokens]]` ambiguous — lint catches it as an ERROR. Two pages wanting one slug are usually one
  page.
- **A summary mirrors its source's path.** `core-sources/vendor/acme.pdf` →
  `summaries/vendor/acme.md`. That's how lint pairs them.
- **Moving a page between subfolders breaks nothing** — wikilinks resolve by slug. Regroup whenever
  the shape stops fitting.

---

## As the project grows

The layout is fixed so that adjusting it stays cheap. Almost everything you'll want to change is a
`DOCS.md` edit, not a migration.

**Free — edit `docs/DOCS.md`, carry on.** No migration, no lint run, existing pages stay valid.

- Rename the nouns in your page types table: *one page per module* → *one page per service*.
- Add, remove, or reword project-specific rules.
- Add or remove out-of-scope paths.

**Cheap — edit `DOCS.md`, run `/lint-docs`, commit the diff.**

- **Switched preset**, because the base turned out to be about something else. Existing pages keep
  their front matter and provenance and get reshaped as they're next touched — nothing is
  bulk-rewritten, because those pages were true when they were written.
- **Split an overgrown topic page** into three, or promote a section to its own entity page. Lint
  finds the inbound wikilinks that need repointing.
- **Switched link style** between wikilinks and relative Markdown.
- **Repointed the source folder** at material you already file elsewhere. Nothing moves — lint
  then reports every file in it as unread, which is a scan queue, not breakage.

**Real work — one commit, lint straight after.** Moving the root (`docs/` → `docs/kb/`): move the
folder whole, change the `Root:` declaration in Part 1, run `/lint-docs` — check 15 rewrites every citation
still pointing at the old root — then grep the repo for the old path, since `CLAUDE.md` and CI
config don't fix themselves. The exact commands are in
[Part 2](docs/DOCS.md#changing-the-shape-later), including the staging step git needs to move a
folder into a subfolder of itself. Splitting one base into two, or merging two into one, is the
same idea: move the folders, port the rules, lint both.

**Never.** Renaming or adding a folder, adding a fourth page type, changing the front-matter
schema. Those are what `lint-docs` and `test-docs` check against — change them locally and every
future version of this kit fights your base.

<details>
<summary>Two knowledge bases in one project</summary>

<br>

Worth it when a project has two bodies of knowledge with **different authority** — one whose facts
come from your own code, one whose facts come from other people's papers. Keeping them apart is the
point: a vendor's claim must never settle a question about your own system.

```
docs/         ← the codebase base.  preset: codebase
research/     ← the reading pile.   preset: outside research
```

Each is fully self-contained: its own `DOCS.md` with its own declarations and its own preset, its own
five folders, its own index and changelog. Run `/init-docs` once per base — the interview runs
again, which is the whole point, since the second base gets its own page types and rules.

Every command takes the root:

```
/scan-docs research/
/lint-docs research/
/ask-docs --root research/ what does the field think about X?
```

With no argument, the root is `docs/`. A run never spans two bases, and pages never wikilink across
roots — if you keep asking questions that need both, they were one base. Merge them.

**Don't split because a folder got large.** A big base is what success looks like, and `ask-docs`
selects candidates by index rather than reading everything.

</details>

---

## Knowledge model

> **`docs/DOCS.md` Part 1 is the authoritative definition of everything in this section.** What
> follows explains the model; that file governs it. Where they differ, `DOCS.md` is right.

Three page types — **summary** (one per source, the only page allowed to paraphrase at length),
**topic** (one per idea or question, where sources are compared), and **entity** (one per person,
service, product, or dataset).

Then one distinction that does most of the work.

### Claim types

**Sources say different kinds of things, and storing them identically is how a knowledge base
starts lying.** These are not interchangeable:

| Sentence | `claim_type` | Who can establish it |
|:--|:--|:--|
| Authentication uses Redis. | `fact` | Evidence |
| The team chose Redis for sessions. | `decision` | **Yours only** |
| We believe Redis is required here. | `assumption` | Either, if labelled |

Plus `open-question` (nobody knows yet) and `contradiction` (sources disagree).

**Five values, deliberately.** A sixth distinction you can't apply consistently at 2am is worse
than no distinction — it makes the field unreliable rather than merely coarse.

Individual claims inside a page are marked with Obsidian callouts, which render in Obsidian and
grep cleanly:

```markdown
> [!check] Decision
> Session storage moved to Redis on 2026-03-11. Rejected: in-process cache (loses state on
> deploy), Postgres (write amplification). — approved by @fenton

> [!question] Open question
> Nothing in the sources describes session revocation on password change.
```

### Authority

```
Your decision, recorded in a page    ← highest
Source material in docs/core-sources/
Agent-generated page content
The agent's background knowledge    ← none; must be labelled if used at all
```

**The agent discovers, summarizes, classifies, links, flags, and proposes. It does not decide.**
It may never set `claim_type: decision` or `status: superseded` — those two values are yours
acts, and they are the only two. An agent that thinks a decision was made writes `open-question`
and asks.

`confidence: high` is not authority. Confidence describes how well evidence supports a claim; it
never promotes an interpretation into a project decision.

### Metadata

Front matter identifies and governs the document. **The knowledge lives in the Markdown body,
never in YAML** — agents corrupt nested YAML far more readily than they corrupt prose, so the
schema is kept small enough to be hard to break. Flat scalars and flat lists only.

Six required fields — `type`, `title`, `status`, `claim_type`, `updated`, `sources` — and four
optional ones: `created`, `confidence`, `superseded_by`, `sensitivity`.

```yaml
---
type: topic
title: Session management
status: active
claim_type: contradiction
updated: 2026-08-30
sources: [docs/core-sources/260415-auth-spec.md, docs/core-sources/260710-ops-runbook.md]
---
```

Pages written before `status` and `claim_type` existed stay valid — `lint-docs` reports them as
WARNING with safe defaults, never as errors. **No migration is required.**

### Lifecycle

Obsolete knowledge must not look identical to current knowledge.

```
draft ──→ active ──→ stale ──→ active     (re-scanned against current sources)
                        │
                        └────→ superseded
```

**Four values, deliberately.** *"No longer in use"* is a fact about the world and belongs in the
page body with a citation; retiring a page from view is a filesystem decision. Neither needs its
own lifecycle state, and both were easy to confuse with `superseded`.

A page goes `stale` when a source it cites changed after its `updated` date. **Stale means
unverified, not wrong** — an agent answering from a stale page says so. Re-scanning clears it. A
`superseded` page keeps its content and gains a `superseded_by` pointer; it is never emptied or
deleted.

---

## Provenance and citations

**A citation is a claim that a document was read.** An agent may cite only what it actually
opened in that run. Citing a plausible-looking path it did not open is fabrication, and
`test-docs` scores it as a hard failure.

Claims carry the most precise anchor the source format supports — `p.4` for a PDF, `§4.2` for a
spec, `src/auth/session.php:112-140` for code, `14:20` for a transcript:

```markdown
... throughput dropped 40% ([[docs/core-sources/260415-bench|260415-bench]], p.4).
```

**Falling back to the bare file name is always correct. Inventing an anchor never is.** A page
number the agent did not see is worse than no page number, because it survives review. The same
applies to line ranges and commit hashes.

Every answer closes with what was consulted, so you can audit the retrieval rather than trust it:

```
Consulted: docs/topics/batching.md, docs/entities/vllm.md, docs/summaries/260415-bench.md
Not consulted: docs/topics/throughput.md (outside scope --topic batching)
```

---

## Retrieval and context limits

`ask-docs` never loads the whole knowledge base. At a thousand pages that is impossible; at a
hundred it already buries the relevant page in noise.

```
question → select candidates → read → estimate budget
                                            │
                                ┌───────────┴───────────┐
                           within budget           over budget
                                │                       │
                              answer          ask for narrower scope
```

**Selection is by index and links**, not by reading everything: match the question against
the index, read those entry points, follow their wikilinks one hop out — two if the
question is broad. Then stop. If the answer is not in reach after two hops, that is a finding
about the knowledge base, not a reason to widen the sweep.

The selection step is deliberately isolated. **It is the retriever, and it is replaceable** — a
project that later indexes `docs/` differently changes only that step, and nothing about the page
format changes with it. Scale comes from the index and the link graph rather than an embedding
store, which is what keeps the whole thing readable and greppable. The cost is that an unlinked
page missing from the index is invisible — which is exactly why `lint-docs` reports orphans.

**The budget is roughly half the context window**, estimated before reading at ~4 characters per
token. Exceeding it is a graceful failure, not a truncation:

```
The evidence for this question exceeds the context budget:
34 pages across 6 topics, roughly 180k tokens.

Narrow the scope and I can answer precisely:
  --topic authentication      7 pages
  --source docs/core-sources/260415-auth-spec.pdf
```

Reading the largest few and answering anyway is the worst available outcome. An answer built on
silently dropped evidence is indistinguishable from a good one, and unauditable.

Answers always separate four states — **known**, **inferred** (labelled as such, every time),
**contradicted**, and **unknown**. *"I don't know based on the available knowledge"* is a correct
answer and often the most useful one. An unsupported inference presented as a fact is a defect
regardless of whether it is right.

---

## Contradictions and review

**Contradictions are never silently resolved.** Two sources disagreeing is a finding, not a bug —
and surfacing it is the main thing this kit offers over plain retrieval.

```markdown
> [!warning] Contradiction
> [[260415-auth-spec]] (§4.2) gives session TTL as 30 minutes.
> [[260710-ops-runbook]] (SESSION_TTL) gives 24 hours, raised from 1800 on 2026-06-02.
> Unresolved as of 2026-08-30.
```

The agent must not close this by reasoning that one source looks newer, more official, or more
detailed. Recency decides only when `docs/DOCS.md` says so — for example, a project rule reading
*"the runbook supersedes the spec for operational values."* Absent such a rule, it stays open
until you rule. `lint-docs` reports open contradictions as **INFO, never ERROR**: an open
contradiction is the system working.

**Your ruling is the documented path out**, and the evidence is preserved rather than
overwritten. The page is rewritten with the ruling on top and the history intact underneath —
`## Decision`, then `## Rationale`, then `## Evidence` listing both sources including the one
that lost. `claim_type` becomes `decision`, `status` becomes `active`, and the commit gives the
ruling an author and a date.

What makes this auditable is that the disagreement is still on the page after it is settled.
Deleting the losing side destroys the reason the decision exists.

### Git conflicts

A merge conflict inside `docs/` is a **semantic** conflict. Git can tell you someone wrote X and
an agent wrote Y in the same place. It cannot tell you which is true.

**Agents never auto-resolve a conflict in `summaries/`, `topics/`, or `entities/`.** They surface
both sides and stop:

```
conflict → you read both sides → check the sources → resolve the Markdown
        → /lint-docs → commit
```

`docs/CHANGELOG.md` (keep both, date order) and the index (keep the union, re-sort) are
the only exceptions, because neither carries meaning.

---

## Linting

`/lint-docs` runs 17 checks at three severities.

| Severity | Reserved for | Examples |
|:--|:--|:--|
| **ERROR** | Mechanically certain and factually wrong | Unparseable front matter, a citation to a file that does not exist, `superseded` with no `superseded_by`, a secret in `core-sources/` |
| **WARNING** | Certain but tolerable | Missing `status`, an orphan, a stale page, a malformed slug |
| **INFO** | Needs you | An open contradiction, a gap with five inbound links, a probable duplicate |

If you gate a merge on lint, gate on ERROR.

It **fixes silently** what is mechanical and reversible — index omissions, defaulted `status` and
`claim_type` on pages that predate those fields, miscased values, `status: stale` where a source
outran the page, and wikilinks broken by a rename confirmable from the changelog.

It **reports and never fixes** anything needing judgment: contradictions, duplicates worth
merging, gaps, uncited claims, broken citations, unparseable front matter, and anything claiming
authority it may not have.

**Malformed YAML is handled as a finding, not a failure.** One unparseable file is one ERROR; the
sweep continues over every other page. Partial metadata from a block that failed to parse is
never ingested, and a block that could not be read is never silently rewritten — lint reports the
corrected block and lets you write it.

---

## Testing

`lint-docs` checks that the knowledge base is well-formed. **`test-docs` checks that it is still
right.** A base can pass every schema check and quietly stop answering the questions it was built
to answer.

Cases live in `docs/scenarios/questions.yaml`. You write it; the agent never edits it:

```yaml
questions:
  - id: session-ttl
    question: What is the absolute session TTL?
    expect_sources: [docs/core-sources/260710-ops-runbook.md]
    require_facts: ["24 hours", "30 minutes"]
    expect_state: contradicted

  - id: session-revocation
    question: What happens to existing sessions when a user changes their password?
    expect_state: unknown
```

**`expect_state: unknown` is the most valuable case you can write.** It asserts that the base
admits a gap instead of inventing an answer — the failure this kit exists to prevent, and the one
no other check catches.

Two passes, reported separately: **structural** (citation validity, source coverage, required
facts, forbidden claims, answer state — path and string comparison, trust these) and **semantic**
via `--semantic` (unsupported claims, overstated certainty, evidence sufficiency — judgment,
opt-in, and the only way to catch the interesting failures).

`test-docs` never edits pages to make a case pass.

---

## Security

**Not every file in a repository is safe to read into an LLM.** The agent never reads, ingests,
summarizes, or quotes from:

```
.env, .env.*                       secrets/**, credentials/**, private/**
*.pem, *.key, *.p12, *.pfx         **/*.secret, **/*secrets*.y*ml
id_rsa, id_dsa, id_ecdsa           .aws/**, .ssh/**, .netrc
```

`scan-docs` checks this before reading; `lint-docs` checks it after, as an ERROR. Add your own
paths under *Out of scope* in `docs/DOCS.md`.

If a source turns out to contain credentials, **the scan stops.** It does not redact and
continue — the secret is already in the agent's context, and a partial summary normalizes the
leak.

Pages may carry an optional `sensitivity: public | internal | confidential` marker. It is
documentation, not enforcement: it tells you what you are about to share.

> [!IMPORTANT]
> These exclusions are **instructions, not enforcement.** Nothing written in a Markdown file can
> stop an agent from reading a path. Use filesystem permissions, `.gitignore`, and secret
> scanning for anything that actually matters.

---

## Works with any AI agent

Nothing here is Claude-specific. The kit is Markdown and a folder convention — any agent that
can read a file can follow it. **The repository is the interface.**

| Tool | Setup |
|:--|:--|
| **Claude Code** | Works as-is — plugin or copied folder |
| **Codex · Cursor · opencode · Windsurf** | Nothing to do. They read the root [`AGENTS.md`](AGENTS.md) automatically |
| **GitHub Copilot** | `cp AGENTS.md .github/copilot-instructions.md` |
| **Gemini CLI** | `cp AGENTS.md GEMINI.md` |
| **ChatGPT / any chat window** | Paste the bootstrap prompt ↓ |

<details>
<summary><b>Bootstrap prompt</b> — for an agent with no repo awareness</summary>

<br>

```
This project is a knowledge base. Its layout, and the paths below, are declared at
the top of Part 1 of docs/DOCS.md — read them there rather than assuming these
defaults, since the root, the source folder and the index can each be elsewhere.

- docs/core-sources/ — raw material. Read it. NEVER create, edit, rename, or delete anything
                  in it. It is exempt from all page rules.
- docs/         — everything else here is markdown pages you write and maintain. Each must
                  trace back to a source or to something I explicitly told you.
- docs/DOCS.md  — the governance contract. Read it fully before writing anything. It
                  overrides these instructions.

Seven operations, fully specified in .claude/skills/<name>/SKILL.md — read the relevant file
before you begin:

- init-docs → .claude/skills/init-docs/SKILL.md   (set up, once per project)
- scan-docs → .claude/skills/scan-docs/SKILL.md
- ask-docs  → .claude/skills/ask-docs/SKILL.md
- lint-docs → .claude/skills/lint-docs/SKILL.md
- test-docs → .claude/skills/test-docs/SKILL.md
- help-docs → .claude/skills/help-docs/SKILL.md   (what state is this in, what next)
- all-docs  → .claude/skills/all-docs/SKILL.md    (scan, lint, test in one pass)

Non-negotiable:
- You propose; I decide. Never set claim_type: decision and never set status: superseded.
  Those are the only two values reserved to me.
- Never resolve a contradiction by overwriting. Record both claims.
- Never cite a document you did not actually open, and never invent a page number or
  line range.
- If the evidence a question needs exceeds your context, say so and ask me to narrow the
  scope. Never answer from a silently truncated set.
- Never read .env*, *.pem, *.key, secrets/, credentials/, private/, .ssh/, .aws/.
- docs/CHANGELOG.md is append-only, newest first.
- The index lists every generated page exactly once. Keep it current.
- Dates in file names are YYMMDD. Dates inside files are YYYY-MM-DD.

Start with: help-docs if you are not sure, or init-docs, scan-docs,
ask-docs QUESTION, lint-docs, test-docs, all-docs.
```

In a chat window with no file access, upload `docs/DOCS.md` and the one skill file you need,
then paste the results back into the repo yourself. Clumsy, but the format holds.

</details>

---

## Obsidian

The page format isn't invented here — it's [Obsidian Flavored
Markdown](https://help.obsidian.md/syntax). Open `docs/` as a vault and everything already
works, no conversion and no plugins:

```
File → Open folder as vault → your-project/docs
```

| What you get | Because |
|:--|:--|
| **Graph view of your knowledge base** | Pages link with `[[wikilinks]]`, so the graph is the real structure — not an approximation |
| **Backlinks and unlinked mentions** | Every citation is a link, so any page shows what references it |
| **Callouts render properly** | Contradictions, decisions, assumptions and open questions all use Obsidian's own `> [!type]` syntax |
| **Front matter in the Properties panel** | `type`, `status`, `claim_type`, `updated`, `sources` are editable fields, not raw YAML |
| **Unresolved links are your to-do list** | The gaps `lint-docs` reports are the same ones Obsidian greys out |

Everything stays plain Markdown in git, so editing a page by hand in Obsidian and having an
agent scan it later are the same workflow. `.obsidian/` is gitignored — your vault settings stay
yours.

**Obsidian is optional.** Nothing in the core depends on it: wikilinks are just text, and
`docs/DOCS.md` documents how to switch the whole knowledge base to plain relative Markdown links
if you prefer.

<details>
<summary><b>Using kepano/obsidian-skills alongside</b></summary>

<br>

**This kit does not bundle Obsidian skills.** It ships five: `init-docs`, `scan-docs`,
`ask-docs`, `lint-docs`, `test-docs`. What it borrows from
[kepano/obsidian-skills](https://github.com/kepano/obsidian-skills) is the packaging model and
the Markdown conventions — not the skills themselves.

If you want an agent that can also drive Obsidian directly — Bases, JSON Canvas, the CLI, clean
web-page extraction — install that plugin alongside this one. They compose: his skills handle
Obsidian's formats and tooling, these handle what goes in the pages and why.

```
/plugin marketplace add kepano/obsidian-skills
```

</details>

---

## Continuous integration

**There is no CI here, and nothing in this kit can run in CI.** All seven operations are Markdown
instructions an agent follows — no program, no exit code, no deterministic pass or fail. That is
a design choice: a linter binary would mean a language runtime, a dependency tree, and a release
process, in a project whose value is that it is Markdown in a folder.

What works today: **review the diff** (a scan touches many files; the pull request *is* the
review), **run `/lint-docs` before merging** anything that touched `docs/` and treat ERROR as
blocking, **run `/test-docs`** after a large scan, and **use ordinary secret scanning** — the
exclusion list is guidance to an agent, not a control.

A deterministic linter that could gate a merge is the most valuable thing this project does not
have. See [Roadmap](#roadmap).

---

## A rhythm that works

```
   add sources ─────► /scan-docs ─────► read the git diff
   as you find them   when a few          every time
        │             have piled up            │
        │                                      │
        └──────────── /ask-docs ◄──────────────┘
                   instead of digging
                          │
                   /lint-docs every fifth scan
                   /test-docs after big changes
```

- **Commit after each scan.** A scan touches many files; the diff is your review, and your undo.
- **Revisit `docs/DOCS.md` monthly.** It's the highest-leverage file in the repo.
- **Watch the gaps.** Dangling wikilinks ranked by inbound count are a reading list, generated
  for free.
- **Write scenarios for what you'd hate to get wrong**, especially the ones whose honest answer
  is "nobody knows yet."

Month one it's a filing cabinet. Month six it answers things you'd never have found by
searching, because the connection was made when the material came in.

<details>
<summary><b>When things go wrong</b></summary>

<br>

| Symptom | Cause | Fix |
|:--|:--|:--|
| Pages are generic and say little | `DOCS.md` still has the shipped defaults, or sources were too few and unrelated | Rewrite the rules; scan three to five files on **one** subject — synthesis needs overlap |
| Same concept split across two pages | Slugs differed on first mention | Lint reports duplicates; merge, then add a naming rule |
| Agent asserts things no source supports | Rules too loose | Add an explicit rule; re-scan that source |
| Pages drifted from their sources | Sources edited after scanning | Lint flags them `stale`; re-scan |
| Summaries are shallow | Agent skimmed a long PDF | Scan that source alone, and say *"read it in full"* |
| Too many dangling links | Normal and healthy | That's your reading list, not a bug |
| `ask-docs` keeps refusing on budget | Question too broad, or the index is thin | Use `--topic` / `--source`; check the index descriptions are specific |
| A contradiction won't go away | It is genuinely unresolved | That's the point. Rule on it — see [Contradictions and review](#contradictions-and-review) |

</details>

---

## Limitations

Stated plainly, because documentation that claims more than it does is the failure this kit
exists to prevent.

- **Nothing here executes.** All seven operations are Markdown instructions. There is no CLI, no
  parser, no test suite, and nothing that can run in CI.
- **`lint-docs` and `test-docs` are therefore not deterministic.** Two runs may word the same
  finding differently or disagree at the margin. Structural checks are far more stable than
  semantic ones because they compare paths and strings rather than meaning — but "more stable"
  is not "reproducible."
- **The context budget is an estimate** at ~4 characters per token. It is deliberately
  conservative and will sometimes refuse a question that would have fit.
- **Retrieval depends on the index and the link graph.** An unlinked page missing from
  the index is invisible to `ask-docs`. That is why orphans are a lint check, and it is
  the real cost of not having an embedding index.
- **Security exclusions are instructions, not enforcement.**
- **Scanning is slow and token-expensive.** Reading a long PDF in full is the point, and it is
  not cheap. The cost is paid once per source rather than once per question.
- **Quality tracks `docs/DOCS.md`.** A generic schema produces generic pages, and no amount of
  scanning fixes that.

---

## Roadmap

Genuinely useful, roughly in order. Nothing here is promised.

- **A deterministic linter.** A small zero-dependency script for the mechanical subset — front
  matter parsing, required fields, valid enum values, broken source paths, orphans, dangling
  links. Real exit codes, CI-gateable, and no LLM. The judgment checks stay in the skill. This
  is the single biggest gap.
- **Fixtures for the linter**, including the malformed-YAML case, so its behavior is testable.
- **Better staleness for code sources** — comparing a cited line range against the current file
  rather than trusting mtime.
- **A test report format** that can be diffed between runs, so drift is visible over time.

Deliberately **not** planned: a vector database, an embeddings pipeline, a web UI, an API
server, a hosted service, or multi-agent orchestration. If retrieval needs to scale further, the
answer is a better index behind the same replaceable selection step — not a new dependency.

---

## Design notes

<details>
<summary><b>Why "docs" and not "wiki"</b></summary>

A wiki is a destination you visit. Docs sit in the repo next to the thing they describe, get
reviewed in pull requests, and are the natural target when you tell an agent *"build docs for
this project."*

</details>

<details>
<summary><b>Why the agent never writes to <code>docs/core-sources/</code></b></summary>

One-way flow means you can always regenerate the pages from scratch and compare. If the agent
could edit its own inputs, that guarantee is gone — and a subtly rewritten source is nearly
impossible to spot later.

</details>

<details>
<summary><b>Why it's called <code>core-sources</code> and not <code>sources</code></b></summary>

The prefix names a **role**, not a file type. `docs/core-sources/` is the root of the provenance
chain: every page is reproducible from it, and nothing else is. Deleting a page loses work;
deleting from here loses the truth. "Core" says trunk, not folder.

It also removes a real ambiguity. A knowledge base documenting a codebase has two kinds of
source: the curated, immutable material in `core-sources/`, and `src/**`, which the codebase
preset reads in place and which changes every commit. Calling both "sources" blurred the
immutability rule exactly where it mattered most — the one rule an agent must never get wrong.

And "source" was already carrying three jobs in this kit: the `sources:` front-matter field, the
citation format, and source code. The folder now means one thing.

The practical payoff is small but real: *never write to `core-sources/`* reads as a boundary,
where *never write to `sources/`* read like a category, and categories invite exceptions.

</details>

<details>
<summary><b>Why contradictions are recorded, not resolved</b></summary>

Overwriting a conflicting claim destroys the most valuable signal the knowledge base produces.
Two sources disagreeing is a finding — often the finding that mattered most.

</details>

<details>
<summary><b>Why the schema is only six required fields</b></summary>

Every field is one a future agent will forget, reorder, or corrupt. Nested YAML fails far more
often than prose does, so the rule is that front matter *identifies and governs* the document
while the Markdown body *carries the knowledge*. Richer relationships go in sections, not in
deeper YAML.

</details>

<details>
<summary><b>Why <code>confidence</code> is not authority</b></summary>

Confidence describes how well evidence supports a claim. Authority is about who decided. An
agent can be highly confident about something nobody ever approved, which is precisely the
case the two fields have to keep apart.

</details>

<details>
<summary><b>Why the changelog is append-only</b></summary>

It's the audit trail for everything an agent did, and what `lint-docs` reads to tell a rename
from a deletion.

</details>

<details>
<summary><b>Why linking to pages that don't exist is encouraged</b></summary>

A dangling wikilink is a cheap, in-context to-do written at the exact moment the gap was
noticed. `lint-docs` ranks them by inbound link count, so the gaps that matter most surface
first.

</details>

<details>
<summary><b>Why one schema file instead of per-agent config</b></summary>

`docs/DOCS.md` survives switching agents and can be reviewed like any other spec. `AGENTS.md`
points at it rather than restating it — two files of rules would drift, and drift between them
is exactly the failure this kit exists to prevent.

</details>

---

## Contributing

Issues and pull requests welcome. Two things worth knowing first.

**The constraints are the product.** A change that adds a runtime dependency, a database, a
service, or an agent-specific code path is very unlikely to be merged regardless of what it
enables. See *Changing this kit* in [`AGENTS.md`](AGENTS.md).

**Changes must land in every file that describes the behavior.** A change to a skill usually
means a change to `docs/DOCS.md`, `AGENTS.md`, this README, and `CHANGELOG.md`. Documentation
claiming a capability the skills don't implement is the exact failure mode this project exists
to prevent, so a PR that updates one and not the others isn't finished.

Schema changes stay backward compatible unless there is genuinely no alternative — someone's
knowledge base is already using the old shape. If a field must change, `CHANGELOG.md` carries
the migration.

---

## Sources & inspiration

- **[Andrej Karpathy — *LLM Wiki*](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)**
  — the three-layer pattern (raw sources / wiki / schema) and the ingest–query–lint loop this
  kit is built on, here renamed scan–ask–lint. His `index.md` and `log.md` became
  the index and `docs/CHANGELOG.md`.
- **[kepano/obsidian-skills](https://github.com/kepano/obsidian-skills)** — the Agent Skills
  packaging model (`skills/<name>/SKILL.md`, `.claude-plugin/`) and the Obsidian Flavored
  Markdown conventions: wikilinks, callouts, YAML properties. **None of his skills are vendored
  here.**
- **[Agent Skills specification](https://code.claude.com/docs/en/skills)** — the skill format.
- **[Obsidian Flavored Markdown](https://help.obsidian.md/syntax)** — link, embed, and callout
  syntax.

### What's different here

- Built to be **installed into an existing project**, with `/init-docs` writing a schema from an
  interview rather than shipping a template you're expected to edit later.
- **Merges** with documentation that already exists instead of demanding a clean slate.
- **An epistemic model, not just a format.** Facts, decisions, assumptions, hypotheses, open
  questions and contradictions are different things, and the schema keeps them apart.
- **An explicit authority model.** The agent proposes; you decide. Six metadata values
  are reserved to you.
- **Retrieval with a stated budget and a graceful failure**, rather than an unbounded read.
- **Provenance with real anchors** and a standing prohibition on inventing them.
- Agent-agnostic by construction: plain-Markdown skills and one root `AGENTS.md`.
- **Obsidian-compatible out of the box**, with nothing depending on Obsidian.

---

## License

Released under the [MIT License](LICENSE). © 2026 Fenton Martin.

Your knowledge base is Markdown in a folder — no database, no lock-in, no service to depend on.
Remove this kit tomorrow and every page it wrote still opens in any text editor.
