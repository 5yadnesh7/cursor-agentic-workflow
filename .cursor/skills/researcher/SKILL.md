---
name: researcher
description: Performs iterative, citation-backed web and codebase research on any topic, library, API, error, or design question. Plans queries, fetches authoritative sources, extracts evidence, self-critiques gaps, and loops until the user's intent is fully covered. Use when the user asks to research, investigate, compare options, check current best practices, find docs, or when any other skill (brainstorm, dev-engineer, debugging, devops) flags an unknown that blocks progress.
---

# Researcher

A disciplined research loop that never stops at the first answer. Plans, searches, reads, extracts, critiques, and iterates until the original intent is covered and every non-trivial claim is supported by a source.

## When To Use

- The user explicitly asks to research, compare, or find documentation.
- A skill hits an unknown (library behavior, API shape, recent change, best practice) and blocks.
- Before choosing a tool, framework, or pattern with long-term impact.
- When verifying a claim that could be outdated in training data.

## Inputs

- `research_question`: the intent to satisfy.
- `constraints` (optional): version pins, license requirements, allowed sources, deadlines.
- `depth` (optional): `quick`, `standard`, or `deep` (default `standard`).
- `known_sources` (optional): docs the user already trusts.

## Tools You May Use

Use tools that are actually available in the current environment. Typical options:

- Web search and fetch tools for live sources.
- Repository search and read tools for internal code and docs.
- MCP tools for integrated data sources.

Never fabricate sources. If a tool is unavailable, say so and adapt.

## Workflow

```
Research Loop:
- [ ] Step 1: Restate the question
- [ ] Step 2: Decompose into sub-questions
- [ ] Step 3: Plan queries and sources
- [ ] Step 4: Search and fetch
- [ ] Step 5: Extract evidence with citations
- [ ] Step 6: Self-critique: coverage, recency, authority, conflicts
- [ ] Step 7: Decide: done or another loop
- [ ] Step 8: Produce final research report
```

### Step 1: Restate The Question

In one or two sentences. Surface hidden assumptions. Define what "done" looks like before searching.

### Step 2: Decompose

Break the question into sub-questions. Each should be answerable by a small number of sources. Example shape:

- What does X do at a conceptual level?
- What is the current API / version / default?
- What are the known trade-offs vs alternatives?
- What do authoritative sources recommend?
- What changed recently that could invalidate older answers?

### Step 3: Plan Queries And Sources

For each sub-question, list:

- Likely authoritative sources (official docs, RFCs, source repos, reputable engineering blogs, standards bodies).
- Two or three query variants that target different angles.
- Any internal repo files worth reading first.

Prefer primary sources over aggregators.

### Step 4: Search And Fetch

Execute searches, open the most promising results, and read them. Batch independent lookups. Stop reading a source once it stops adding new information.

### Step 5: Extract Evidence

For every non-trivial claim, capture:

- A short quote or paraphrase.
- The source URL or file path.
- Publication or last-updated date if visible.
- Version numbers when relevant.

Keep evidence in a running notes block; do not mix it with interpretation.

### Step 6: Self-Critique

Before concluding, grade the current evidence against this rubric:

| Dimension | Question |
|-----------|----------|
| Coverage | Is every sub-question answered? |
| Authority | Are sources primary or reputable? |
| Recency | Do sources reflect current versions and practices? |
| Consistency | Do sources agree? If not, is the conflict explained? |
| Specificity | Do claims match the user's actual constraints? |

Any `no` is a trigger to loop back to Step 3 with sharper queries.

### Step 7: Decide

- If all rubric checks pass and the original intent is covered → proceed to Step 8.
- Otherwise → run another loop, narrowing or broadening only where the rubric flagged gaps.
- Respect a reasonable iteration cap based on `depth`; if still not covered, surface the open gaps to the user instead of guessing.

### Step 8: Final Research Report

```
# Research: <question>

## Summary
<2–5 sentences answering the question directly>

## Key Findings
- <finding> [source]
- <finding> [source]

## Comparisons / Trade-offs
| Option | Pros | Cons | When to choose |
|--------|------|------|----------------|
| ...    | ...  | ...  | ...            |

## Recommendation
<what to do, given the user's constraints>

## Caveats & Unknowns
- <what is still uncertain>
- <what could change soon>

## Sources
1. <title> — <url or path> — <date/version>
2. <title> — <url or path> — <date/version>
```

Every finding and recommendation must map back to at least one source.

## Depth Presets

- `quick`: 1 loop, 3–5 sources, summary-only output.
- `standard`: up to 2 loops, 5–10 sources, full report.
- `deep`: up to 3 loops, 10+ sources, full report plus comparison table and caveats.

## Quality Bar

Research is good when:

- The summary could stand alone and still answer the question.
- Every non-obvious claim is traceable to a source.
- The user's constraints (versions, stack, budget) are reflected in the recommendation.
- Conflicts between sources are resolved or named.

## Anti-Patterns

- Answering from memory when the topic is version-sensitive.
- Citing a single source and declaring victory.
- Over-researching trivial questions past the point of usefulness.
- Mixing interpretation into the evidence log.
- Dropping citations in the final report.
