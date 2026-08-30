---
name: ask-docs
description: Answer a question from the knowledge base in docs/, with citations back to sources, and log the query. Use when the user asks a question about their notes, says ask the docs, what do my sources say about X, or wants a synthesis across scanned material.
---

# Ask

Answer from `docs/` first. The point of the knowledge base is that the synthesis has already
been done — do not re-read every source when a topic page already answers the question.

## Procedure

1. Read `docs/DOCS.md` and `docs/README.md`.
2. Pick the entry points from the index by title and description. Read those pages, then
   follow their wikilinks one hop out. Two hops if the question is broad.
3. Drop to `docs/sources/` **only** when the docs are thin, the question needs a verbatim quote,
   or you suspect a page is stale. Say so in the answer when you do.
4. Answer directly, then cite. Every substantive claim names the page it came from, and the
   page's own citation carries the source: `Latency rose under batching ([[batching]] ←
   docs/sources/260415-bench.pdf).`
5. Surface uncertainty explicitly:
   - Contradiction in the docs → present both sides, do not pick a winner silently.
   - Nothing in the docs → say the knowledge base does not cover it, name the gap, and offer
     to scan something that would.
6. Append to `docs/CHANGELOG.md`:

```markdown
## 2026-08-30 — ask
- Q: How does batch size affect latency?
- Read: docs/topics/batching.md, docs/entities/vllm.md
- Gap: no source covers batching above 256
```

## Rules

- Never answer from your own background knowledge without labelling it as outside the docs.
- Do not edit topic or entity pages during an ask. If you spot a fix, note it in the report
  and let the user decide — content changes belong to `scan-docs` and `lint-docs`.
