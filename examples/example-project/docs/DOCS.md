# DOCS.md — the knowledge governance contract

> [!NOTE]
> **This file is abbreviated, and real `/init-docs` output is not.** In a real project this file
> is standalone: `/init-docs` copies Part 1 of the shipped contract into it in full, so the
> knowledge base carries its own rules and stays readable without the kit.
>
> Here it is reduced to a pointer instead, purely so the example cannot drift out of sync with
> the contract it is demonstrating. The cost is that this one directory is not portable — copy
> it elsewhere and the link below breaks. That is the only way it differs from real output.

**Everything in Part 1 of the shipped contract — layers, authority, front matter, claim types,
lifecycle, citations and provenance, contradictions, retrieval and context, security, git
conflicts, naming and dates — applies here unchanged. Read it in
[`../../../docs/DOCS.md`](../../../docs/DOCS.md).**

Below are the only three sections `/init-docs` rewrites per project.

## Page types

| Type | Path | One page per | Purpose |
|---|---|---|---|
| Summary | `docs/summaries/<source-slug>.md` | source document | What the document says, its evidence, and its limits. |
| Topic | `docs/topics/<slug>.md` | mechanism or decision | How something works end to end. Where sources are compared and disagreements surface. |
| Entity | `docs/entities/<slug>.md` | service, store, or config key | Stable facts about one component, plus every place it is referenced. |

## Out of scope

```
docs/core-sources/archive/**
```

Plus the security exclusions in the shipped contract.

## Project-specific rules

Four rules, from the interview. Each one is checkable.

- **Never state a spec value as production behavior.** A specification says what should be
  true; a runbook says what is. When they differ, that is a contradiction, and this project has
  a live one on session TTL.
- **Every configuration value carries its unit and its as-of date.** `86400 seconds (24 hours,
  since 2026-06-02)`, not `86400`. A bare number in a topic page is a lint finding.
- **Config keys are entities, never bullet lists inside a summary.** `SESSION_TTL` is referenced
  from too many places to live inside one page.
- **Every topic page ends with an Open questions section.** An empty one means nothing is
  unresolved, which is a signal rather than a defect.
