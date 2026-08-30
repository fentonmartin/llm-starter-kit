# Changelog

Release history for `llm-starter-kit` itself. The log of what an agent did *inside* a knowledge
base is `docs/CHANGELOG.md`, which is a different file with a different purpose.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versioning is [semantic](https://semver.org/spec/v2.0.0.html), applied to the schema and the
command surface: a breaking change is one that invalidates an existing knowledge base.

## [1.1.0] — 2026-08-30

An architectural maturity release. `1.0.0` established the loop — sources in, pages out, with
citations. `1.1.0` makes the result auditable: what kind of claim a page carries, where it came
from, whether it is still current, and who is allowed to decide.

**No breaking changes. No migration required.** Every knowledge base built on `1.0.0` remains
valid; see *Migration* below.

### Added

- **Claim types.** `claim_type` distinguishes `fact`, `decision`, `assumption`, `hypothesis`,
  `open-question`, `contradiction`, and `instruction`. *"Authentication uses Redis"* and *"the
  team chose Redis"* are different claims, and storing them identically is how a knowledge base
  starts lying. Individual claims inside a page are marked with Obsidian callouts.
- **Lifecycle.** `status` tracks `draft`, `active`, `stale`, `superseded`, `deprecated`, and
  `archived`, with `superseded_by` naming the replacement. Obsolete knowledge no longer looks
  identical to current knowledge.
- **An authority model.** An explicit source-of-truth hierarchy, and the rule it enforces: the
  agent discovers, summarizes, classifies, and proposes; the human decides. Agents may not set
  `claim_type: decision` or `instruction`, nor `status: superseded`, `deprecated`, or
  `archived`. `confidence: high` is not authority.
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
- **`examples/example-project/`.** Two sources that disagree, four generated pages, three
  scenarios, and three suggested edits that should each produce a specific finding.
- **`CHANGELOG.md`** — this file.

### Changed

- **`docs/DOCS.md` is now the knowledge governance contract** rather than a formatting schema.
  It defines claim types, lifecycle, authority, provenance, freshness, retrieval limits, answer
  states, human review, git conflicts, and security, alongside the page and naming rules it
  already carried.
- **`lint-docs` gained severity levels and grew from 8 checks to 14.** ERROR is reserved for
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

None required. `1.1.0` is additive.

- Pages without `status` or `claim_type` remain valid. `lint-docs` reports them as **WARNING**,
  not ERROR, and offers safe defaults: `status: active`, and `claim_type: fact` on a summary or
  `open-question` on a topic or entity it cannot classify. It never defaults to `decision`.
- Run `/lint-docs` once and accept the mechanical fixes to upgrade a base in place. Or do
  nothing: pages are upgraded as they are next touched.
- `docs/scenarios/` is optional. Without it, `test-docs` has nothing to run and says so.
- No file was renamed, no command was renamed, no field was removed, and no directory moved.

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
- `docs/` structure: `sources/` (human-owned, immutable to agents), `summaries/`, `topics/`,
  `entities/`, plus an index and an append-only changelog.
- `docs/DOCS.md` as a per-project schema, written by `init-docs` from an interview.
- Front matter: `type`, `title`, `created`, `updated`, `sources`, `confidence`.
- Obsidian-compatible wikilinks, so `docs/` opens as a vault without conversion.
- Contradiction callouts, and the rule that contradictions are never silently resolved.
- `AGENTS.md` as the entry point for any filesystem-capable agent.

[1.1.0]: https://github.com/fentonmartin/llm-starter-kit/releases/tag/v1.1.0
[1.0.0]: https://github.com/fentonmartin/llm-starter-kit/releases/tag/v1.0.0
