---
name: ask-docs
description: Answer a question from the knowledge base in docs/, with citations back to sources, and log the query. Use when the user asks a question about their notes, says ask the docs, what do my sources say about X, or wants a synthesis across scanned material.
---

# Ask

Answer from `docs/` first. The point of the knowledge base is that the synthesis has already
been done — do not re-read every source when a topic page already answers the question.

Two failures matter more than a slow answer: **loading everything** and **answering past the
evidence**. The procedure below exists to prevent both.

## Procedure

### 1. Read the contract

Read `docs/DOCS.md` **Part 1** in full, and the index. Part 2 is examples and rationale —
read it only if you need it. `DOCS.md` overrides anything here: page types, link style,
retrieval scope, security exclusions, and the project's own rules.

**The declarations.** Part 1 of `docs/DOCS.md` opens with three paths. Every path in this skill is
written using their defaults; read them as whatever Part 1 declares:

| Written here | Declared as |
|---|---|
| `docs/…` | **Root** — `docs/` unless the base was scaffolded beside a published docs site. |
| `docs/core-sources/` | **Source folder** — one folder: `docs/core-sources/`, a top-level `sources/`, or wherever the material already lives, including outside the root. Immutable to you wherever it is. Part 1 may also list *read-in-place sources*, which you cite but never file. |
| `docs/INDEX.md` | **Index** — `docs/INDEX.md` or `docs/README.md`. |

If the user passed a root as an argument, read `<root>/DOCS.md` and use that base only.

**Page layers may have subfolders.** `docs/topics/auth/session-management.md` is a topic page like
any other: the layer folder decides what a page is, however deep it sits. Slugs are unique across a
whole layer, so a wikilink resolves by slug and does not care about the path. The layers, their
names, and their rules never change with any declaration.

**Commits.** Part 1 also declares whether you commit your own work — `none` by default. This
command writes only a changelog entry; at `per-run` or `per-file`, stage that file alone, and
never push.

**Address.** Part 1 declares what to call the owner. Use it when you speak to them directly —
opening an answer, flagging something that needs them — and not in every sentence. If it reads
`(not set)`, say *"you"* and never guess a name from git history or an email address. It never
appears inside a page.

### 2. Select candidates — never load the whole base

Selection is by index and links. **Do not read every page, and do not glob `docs/**/*.md`.**
A knowledge base is expected to outgrow your context; behave as though it already has.

1. Match the question against titles and descriptions in the index.
2. Read those entry-point pages.
3. Follow their wikilinks one hop out. Two hops only if the question is broad.
4. Stop. If the answer is not in reach after two hops, that is a finding — report the gap
   rather than widening the sweep.

Honor explicit scope when given, and say what you scoped to:

| Scope | Effect |
|---|---|
| `--topic <slug>` | Start from that topic page and its links only. |
| `--entity <slug>` | Start from that entity page and its links only. |
| `--source <path>` | Answer from that source and its summary only. |

Skip pages with `status: superseded` unless the user is asking
about history. Include `status: stale` pages, and label them in the answer.

Drop to the source folder **only** when the pages are thin, the question needs a verbatim quote,
or you suspect a page is stale. Say so in the answer when you do.

*This step is the retriever. It is deliberately replaceable — a project that later indexes
`docs/` differently changes only this step, and nothing about the page format changes with it.*

### 3. Check the budget before reading

Estimate the cost of the candidate set before you open it. File sizes are enough: roughly
**4 characters per token**, and a PDF costs several times what its byte count suggests.

The budget is **about half your context window**, leaving room for reasoning and the answer.

**If the candidates fit, continue. If they do not, stop and say so.** Do not read the largest
few and answer anyway, do not skim, and do not quietly drop the tail of the list. An answer
built on silently discarded evidence is indistinguishable from a good one, which is exactly
what makes it the worst outcome available.

Report the overflow with a way out:

```
The evidence for this question exceeds the context budget:
34 pages across 6 topics, roughly 180k tokens.

Narrow the scope and I can answer precisely:
  --topic authentication      7 pages
  --topic rate-limiting       5 pages
  --source docs/core-sources/260415-auth-spec.pdf
```

### 4. Answer, then cite

Answer directly first. Then attribute every substantive claim to the page it came from, with
that page's own citation carrying the source:

```
Latency rose under batching ([[batching]] ← docs/core-sources/260415-bench.pdf, p.4).
```

**Cite only what you actually opened.** A citation is a claim that you read the document. A
plausible-looking path you did not open is fabrication, and `test-docs` scores it as a hard
failure. If a page you would like to cite was outside the budget, say it was not consulted.

Close the answer with what you consulted, so the user can audit the retrieval rather than
trusting it:

```
Consulted: docs/topics/batching.md, docs/entities/vllm.md, docs/summaries/260415-bench.md
Not consulted: docs/topics/throughput.md (outside scope --topic batching)
```

### 5. Separate what is known from what is not

Label every part of the answer:

- **Known** — supported by cited evidence.
- **Inferred** — your reasoning across sources. Say so explicitly, every time. Never present
  an inference as a fact, even a confident one.
- **Contradicted** — sources disagree. Present both sides and do not pick a winner.
- **Unknown** — no source covers it. Name the gap and offer to scan something that would.

"I don't know based on the available knowledge" is a correct answer and often the most useful
one. An unsupported inference dressed as a fact is a defect regardless of whether it is right.

A worked shape:

```
Known — sessions are stored in Redis with a 24-hour TTL
  ([[session-management]] ← docs/core-sources/260710-ops-runbook.pdf, p.2).

Contradicted — the auth spec gives the TTL as 30 minutes
  ([[docs/core-sources/260415-auth-spec|260415-auth-spec]], §4.2). Unresolved; see
  [[session-management]].

Unknown — nothing in the sources describes revocation on password change.
```

### 6. Log the query

Append to `docs/CHANGELOG.md`:

```markdown
## 2026-08-30 — ask
- Q: How does batch size affect latency?
- Read: docs/topics/batching.md, docs/entities/vllm.md
- Gap: no source covers batching above 256
```

Log an overflow too — a question that does not fit is a finding about the knowledge base:

```markdown
## 2026-08-30 — ask
- Q: How does the whole ingestion pipeline work?
- Result: over context budget, 34 candidate pages. Asked for narrower scope.
```

## Rules

- Never answer from your own background knowledge without labelling it as outside the docs.
- Never read a path excluded by the security section of `docs/DOCS.md`.
- Do not edit topic or entity pages during an ask. If you spot a fix, note it in the report
  and let the user decide — content changes belong to `scan-docs` and `lint-docs`.
- Do not resolve a contradiction in order to give a cleaner answer. Present both sides.
