---
name: help-docs
description: Show what this knowledge base is, what state it is in, and what to do next. Use when the user says help, help-docs, what can I do, what commands are there, how do I use this, what should I do next, or asks what state the knowledge base is in.
---

# Help

Orient someone in under a screen. **What this base is, what state it is in, what to do next, then
the commands.** In that order — a command list is the least useful part of help, because the
question behind *"what can I do?"* is almost always *"what should I do now?"*

**This must be cheap.** Read `docs/DOCS.md` Part 1 for the declarations and page types, read the
index, and count files. **Never read a page body, never read a source, never glob the base.** Help
that costs a full context read is not help. If the base is large, counting is still cheap — you
are listing directories, not opening them.

Do not write anything. Help changes nothing, appends no changelog entry, and fixes nothing it
finds. If it spots a problem, it names the command that fixes it.

## 1. Work out the state

In this order, and stop at the first that matches:

| State | How you know | What to say |
|---|---|---|
| **Not installed** | No `DOCS.md` under `docs/`, or no `docs/` at all | The kit is not set up here. Explain it in two lines and point at `/init-docs`. Do not survey the project or start interviewing. |
| **Installed, no material** | Source folder empty or absent | Setup is done. They need to put material in — name the exact folder from the declaration. |
| **Material, nothing scanned** | Sources present, `summaries/` empty | The obvious next step is `/scan-docs`. Say how many files are waiting. |
| **Unread sources** | Files in the source folder with no matching summary | Say how many, and that `/scan-docs` will read them. |
| **Scanned, no scenarios** | Pages exist, `scenarios/questions.yaml` is `questions: []` or missing | Suggest writing two or three. Say plainly this is the most-skipped step and the one that catches drift later. |
| **Healthy** | Sources read, pages present, scenarios written | Nothing is owed. Say so, and give the rhythm rather than inventing work. |

A base can be in more than one of these; report the first that matches and mention the rest in one
line. Do not produce a list of six recommendations — one clear next step is the point.

## 2. Report

Keep it to roughly a screen. Adapt the shape to the state; this is the healthy case:

```
Knowledge base: docs/          (codebase — one page per service, decision, endpoint)
Sources:        docs/core-sources/       14 files, all read
Pages:          9 summaries · 6 topics · 4 entities
Scenarios:      3
Last run:       scan, 2026-08-30

Next: nothing owed. 2 open contradictions are waiting on you —
      /ask-docs "what is contradicted?" to see them.

  /scan-docs    read new material into pages
  /ask-docs Q   answer from the pages, with citations
  /lint-docs    health-check and fix what is mechanical
  /test-docs    check the base still answers its known questions
  /help-docs    this

Rules and page types: docs/DOCS.md
```

And the not-installed case, which is a different job — it is a pitch, not a status report:

```
No knowledge base here yet.

This kit turns material you drop in a folder into linked markdown pages
that any agent can answer from, with every claim traced to its source.
No database, no index to rebuild — markdown, YAML front matter and git.

  /init-docs    set it up. asks about six questions, takes a few minutes.

Read first, if you want: docs/DOCS.md is the contract it will write for you.
```

Rules for the report:

- **Name real paths and real numbers**, from the declarations and from counting. Never print
  `docs/core-sources/` when the base declares `sources/`, and never say "some files".
- **The page types line comes from their `DOCS.md`, not from the preset name.** *"one page per
  service, decision, endpoint"* tells them what their base is for; *"codebase preset"* does not.
- **One next step, in one sentence**, with the command that does it.
- Mention open contradictions if the index or changelog makes them visible for free. Do not go
  looking — that is `/lint-docs`.
- If `DOCS.md` still carries the shipped default rules, say so: it means the interview never
  happened, and it is the main way this kit fails. Point at `/init-docs`.

## 3. When they asked something narrower

*"How do I add a source?"* deserves an answer, not a status report. Answer the question in a line
or two, name the command, and stop. Offer the full picture only if it is genuinely relevant:

> Drop the file in `docs/core-sources/` — any name, `YYMMDD-` prefix if it has a meaningful date
> — then run `/scan-docs`. It reads anything new and writes the pages.

Common ones worth answering directly:

| They ask | Short answer |
|---|---|
| How do I add material? | Put it in the source folder, run `/scan-docs`. Name the folder. |
| Why did it not read my file? | It only reads what has no summary yet. Check the security exclusions and the out-of-scope list in `DOCS.md`. |
| How do I change the rules? | Edit *Project-specific rules* in `docs/DOCS.md`. Free, no migration — Part 2's *Changing the shape later* says what each kind of change costs. |
| Why is the answer "unknown"? | The base does not know, and said so instead of guessing. That is the feature. Add a source that covers it. |
| Can I move the folders? | The root and source folder are declarations in Part 1. Layers and page types are fixed. Part 2, *Changing the shape later*. |
| Is my base healthy? | `/lint-docs` for structure, `/test-docs` for whether it still answers correctly. |

## Rules

- Read only `docs/DOCS.md`, the index, and directory listings. Never a page body, never a source.
- Write nothing. No changelog entry, no fixes, no index update — even if you notice it is stale.
  Say it and name `/lint-docs`.
- Never invent work to recommend. "Nothing is owed" is a valid and useful answer.
- Never re-explain the whole contract. Link to `docs/DOCS.md` and stop.
