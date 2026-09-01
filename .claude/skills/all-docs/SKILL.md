---
name: all-docs
description: Run the whole maintenance pass in order — scan new material, lint the base, test the scenarios — skipping stages with nothing to do, and finish with one consolidated report. Use when the user says all-docs, run everything, do the whole pass, catch up the docs, bring the knowledge base up to date, or sits down to the base after time away.
---

# All

The sit-down-to-the-docs command. **Scan, lint, test — in that order, skipping what has nothing to
do, and reporting once at the end.**

It orchestrates; it does not re-implement. Each stage runs the real skill, following that
`SKILL.md` exactly. Nothing here overrides them, and nothing here is allowed to be a quicker
version of them.

Read `docs/DOCS.md` **Part 1** once, at the start. The stages do not each re-read it.

**The declarations.** Part 1 opens with three paths — root, source folder, index — plus a
**Commits** setting. Every path in this skill is written using the defaults; read them as whatever
Part 1 declares. Honour the commit setting at every stage.

## Why the order

Each stage depends on the one before, which is the whole reason this exists as a command rather
than as advice to run three things:

```
scan   new material becomes pages          → changes what there is to check
lint   the base is checked and tidied      → changes what the scenarios will see
test   the scenarios are run against it    → tells you whether it still answers
```

Running them in any other order tests a base you are about to change.

## 1. Say what you are about to do, and what it will cost

Before anything runs, count and report. This is the one chance to stop.

```
14 files in docs/core-sources/, 3 with no summary.
Plan: scan 3 → lint → test 5 scenarios.
Scanning reads sources in full, so this is the expensive part — roughly the
size of those 3 files, in tokens.

Commits: none. I'll leave everything in the working tree for you to review.
```

**If the scan stage is large, stop and ask.** More than roughly ten unread sources, or sources
that are clearly huge, means a run whose cost the user has not agreed to. Offer to do a batch:
*"I'll scan the five oldest, then lint and test — run this again for the rest."* A user who
dropped in eighty PDFs wants to know before, not after.

If nothing at all needs doing, say so in one line and stop. Do not run three stages to prove the
base is fine — that is what `/help-docs` is for, and it is free.

## 2. Run the stages

Skip any stage with nothing to do, and say you skipped it.

| Stage | Skip when | Follow |
|---|---|---|
| **Scan** | No source has a missing or outdated summary | `.claude/skills/scan-docs/SKILL.md` |
| **Lint** | Never skipped. It is the cheap stage and the one that catches what scan broke | `.claude/skills/lint-docs/SKILL.md` |
| **Test** | `scenarios/questions.yaml` is empty or absent | `.claude/skills/test-docs/SKILL.md` |

**Between stages, do not summarize.** Carry the findings forward and report once at the end. Three
reports in a row is what this command exists to avoid.

**When to stop early**, rather than pressing on:

- **A security exclusion fired.** Stop immediately, report, commit nothing. This overrides every
  other instruction here.
- **Lint found ERRORs that make the base unreadable** — unparseable front matter, a declared source
  folder that does not exist. Testing a base that cannot be read produces noise, not signal. Report
  the errors, skip test, and say why you skipped it.
- **The user interrupted.** Report what completed and what did not.

Everything else runs to the end. Open contradictions, gaps, stale pages and failing scenarios are
findings, not failures — they are the output, and stopping on them would defeat the point.

## 3. One report

The whole value of this command is that the owner reads one thing. Lead with what needs them; put
what ran underneath.

```
Needs you (3):

  · Contradiction — session TTL. 260415-auth-spec says 30 minutes, 260710-ops-runbook
    says 24 hours. Opened on [[session-management]]; both sides recorded.
  · Scenario failed — "session-revocation" expected unknown, got known. A page now
    asserts a revocation policy neither source states. docs/topics/session-management.md
  · Gap — [[rate-limiting]] has 5 inbound links and no page.

Ran:
  scan   3 sources → 3 summaries, 2 topics updated, 1 entity created
  lint   17 checks · 0 errors · 4 warnings (fixed 3: index, 2 defaulted claim_types)
  test   5 scenarios · 4 pass · 1 fail

Skipped: nothing.
Commits: none — 9 files changed, in the working tree. `git diff` to review.
```

Rules for the report:

- **Needs-you first, and count it.** Someone who reads only the first line should learn whether
  this run is asking anything of them.
- **Never merge a finding into a summary.** Three contradictions are three lines, not
  *"3 contradictions found."* A collapsed finding is one nobody acts on.
- **Say what you skipped and why.** A skipped stage is information — *"test skipped, no scenarios
  written"* is how someone learns that file exists.
- **Say what was committed, or that nothing was.** With `Commits: none`, name the file count and
  point at `git diff`.

## 4. Changelog

**One entry, not three.** The stages each want to append; suppress that and write a single entry
for the pass, so `docs/CHANGELOG.md` records one thing the owner actually did.

```markdown
## 2026-08-30 — all
- Scanned: 3 sources → docs/summaries/…, docs/summaries/…, docs/summaries/…
- Lint: 0 errors, 4 warnings, 3 fixed
- Test: 5 scenarios, 1 failed (session-revocation)
- Open: contradiction on session TTL; gap on [[rate-limiting]]
```

## Rules

- **Never re-implement a stage.** Read its `SKILL.md` and follow it. If a stage's rules and this
  file disagree, the stage wins — this file only decides what runs and in what order.
- **Never resolve anything a stage escalated.** Running three commands together does not confer
  authority the commands do not have. Contradictions, duplicates and owner-only fields are as
  untouchable here as anywhere.
- **Never skip lint.** It is the cheapest stage and the one that catches what the scan just did.
- **Never run `ask-docs` as part of a pass.** A question is the owner's act, with their question in
  it, and there is no such thing as asking on someone's behalf.
- **Never push**, whatever `Commits` says.
