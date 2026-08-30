# DOCS.md — the knowledge governance contract

*This is the example project's schema. It is the shipped `docs/DOCS.md` with only the three
sections `/init-docs` customizes — page types, out of scope, project-specific rules — replaced.
Everything else is inherited verbatim and abbreviated here with a pointer, so this example
cannot drift out of sync with the real contract.*

**Everything in Part 1 of the shipped contract — layers, authority, front matter, claim types,
lifecycle, citations and provenance, contradictions, retrieval and context, security, git
conflicts, naming and dates — applies here unchanged. Read it in
[`../../../docs/DOCS.md`](../../../docs/DOCS.md).**

In a real project they are copied into this file in full, so the knowledge base carries its own
contract and stays readable without the kit.

## Page types

| Type | Path | One page per | Purpose |
|---|---|---|---|
| Summary | `docs/summaries/<source-slug>.md` | source document | What the document says, its evidence, and its limits. |
| Topic | `docs/topics/<slug>.md` | mechanism or decision | How something works end to end. Where sources are compared and disagreements surface. |
| Entity | `docs/entities/<slug>.md` | service, store, or config key | Stable facts about one component, plus every place it is referenced. |

## Out of scope

```
docs/sources/archive/**
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
