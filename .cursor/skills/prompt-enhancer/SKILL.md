---
name: prompt-enhancer
description: Analyzes a raw user prompt, infers the underlying intent, and rewrites it into a clear, complete, and actionable prompt with explicit goals, constraints, inputs, outputs, and success criteria. Use at the very start of any workflow, whenever a prompt feels vague, ambiguous, or underspecified, or when another skill (such as validate-prompt-intent) requests a rewrite.
---

# Prompt Enhancer

Transforms a raw user prompt into a structured, high-signal prompt that downstream skills, agents, and workflows can act on without guesswork.

## When To Use

- The user sends a short, vague, or ambiguous request.
- A workflow or skill (e.g. `validate-prompt-intent`) asks for a rewrite.
- Before handing work to `brainstorm`, `researcher`, `db-engineer`, `dev-engineer`, `debugging`, `tester`, or `devops`.
- Any time the intent, scope, inputs, outputs, or acceptance criteria are unclear.

## Core Principles

1. Preserve the user's original intent — never invent new requirements. If something must be assumed, mark it as an assumption.
2. Prefer clarification over guessing when the gap is large. Ask focused questions only for blocking unknowns.
3. Keep language concrete, testable, and free of filler.
4. Stay tool and stack agnostic unless the user specified technology.
5. Output a single enhanced prompt plus a short delta, not a long essay.

## Inputs

- `raw_prompt`: the user's original message.
- `context` (optional): open files, recent edits, prior conversation, workspace rules, previous skill outputs.
- `feedback` (optional): reasons an earlier version failed validation.

## Workflow

Track progress with this checklist:

```
Prompt Enhancement:
- [ ] Step 1: Extract raw intent
- [ ] Step 2: Classify intent type
- [ ] Step 3: Detect gaps and ambiguities
- [ ] Step 4: Ask blocking questions (only if needed)
- [ ] Step 5: Draft enhanced prompt
- [ ] Step 6: Record assumptions and open questions
- [ ] Step 7: Hand off to validate-prompt-intent
```

### Step 1: Extract Raw Intent

Summarize in one sentence what the user actually wants. Distinguish:

- Goal (the outcome)
- Action (what must be done)
- Subject (what it is done to)

### Step 2: Classify Intent Type

Pick the closest category (one or more):

- Research / discovery
- Brainstorm / design
- Implementation / feature build
- Bug fix / debugging
- Refactor / migration
- Testing / verification
- Infra / devops / deployment
- Data / database modeling
- Documentation / explanation

The classification drives which downstream skill should run next.

### Step 3: Detect Gaps And Ambiguities

Check for missing or unclear:

- Scope boundaries (in-scope vs out-of-scope)
- Target files, modules, services, or environments
- Inputs, outputs, and data shapes
- Constraints (performance, security, compatibility, budget, deadlines)
- Non-functional requirements
- Success criteria and acceptance tests
- Dependencies and affected systems
- User personas or consumers of the result

### Step 4: Handle Blocking Unknowns

This skill never asks the user directly. Instead, for each unknown that would materially change Goals, Non-Goals, Inputs, Outputs, Constraints, Success Criteria, or stack:

1. If a conservative default exists (e.g. "MVP only, defer auth", "start with paper trading", "scope to stocks"), encode it as an explicit **Assumption** in the output. Assumptions are reversible by the user at the approval gate.
2. If no conservative default is safe (e.g. "which market — stocks vs forex vs crypto — has material implications for data pipelines and regulation"), add the item to `## Open Questions` **prefixed with `BLOCKING:`** so the validator routes it to the workflow's ask-user path.
3. Prefer (1) over (2) whenever possible. Five assumptions the user can reject are better than five questions the user has to answer.

Cap blocking questions at 5. If there would be more, prioritize the questions whose answers most change Goals/Non-Goals.

Never ask more than three non-blocking questions. Non-blocking items are optional polish the user can ignore; they go in `## Open Questions` without the `BLOCKING:` prefix.

### Step 5: Draft Enhanced Prompt

Produce the enhanced prompt using this template exactly:

```
# Enhanced Prompt

## Intent
<one-sentence summary of what the user wants>

## Intent Type
<one or more categories from Step 2>

## Context
- <relevant files, components, services, environments, prior decisions>

## Goals
- <specific, testable goal>
- <specific, testable goal>

## Non-Goals
- <explicitly out of scope>

## Inputs
- <data, arguments, triggers, events>

## Outputs / Deliverables
- <artifacts, files, behaviors, docs>

## Constraints
- <performance, security, compatibility, style, budget, deadlines>

## Success Criteria
- <observable, verifiable condition>
- <observable, verifiable condition>

## Assumptions
- <assumption made to fill a gap — reversible at the approval gate>

## Open Questions
- BLOCKING: <question the user must answer before work can start — include a sensible default in parentheses, e.g. "(default: stocks)">
- <non-blocking, optional polish question>

## Suggested Next Skill
<prompt-enhancer | validate-prompt-intent | brainstorm | researcher | db-engineer | dev-engineer | debugging | tester | devops>
```

### Step 6: Record Assumptions And Open Questions

Every assumption must be explicit. Open questions must be answerable, not rhetorical. If an open question is blocking, surface it to the user before handoff.

### Step 7: Hand Off (hard-coupled to validate-prompt-intent)

This skill's output is **never** consumed directly. Return the enhanced prompt to the caller workflow with the explicit requirement that `validate-prompt-intent` runs on it immediately, before the prompt is presented to the user, reused, or acted on in any way.

Coupling rules:

- Every call to `prompt-enhancer` must be followed by a call to `validate-prompt-intent`. There is no "good enough to skip validation" path.
- If `validate-prompt-intent` returns `refine`, accept its feedback verbatim and rerun this skill, changing only the sections it flagged. Untouched sections stay stable.
- If `validate-prompt-intent` returns `ask-user`, the workflow will collect user answers and call this skill again with those answers appended as `user_answers` (not as `feedback`). Treat `user_answers` as ground truth that fills gaps; do not rewrite sections the user did not touch.
- If `validate-prompt-intent` returns `approve`, the workflow may present the prompt to the user. A subsequent `refine <feedback>` from the user loops back into this skill, which again must be followed by validation.

## Chat Presentation

The workflow (not this skill) owns the chat turn. This skill produces the enhanced-prompt body and a one-line delta, returns both to the workflow, and stays silent. Specifically:

- **Do not narrate work.** Do not write messages like "I'll now analyze your prompt and extract intent…". Just produce the output.
- **Do not emit any chat directly.** Return structured output to the workflow. The workflow decides how much of it to render — but only *after* `validate-prompt-intent` has approved it.
- **Always include a one-line delta summary** (separate from the body) that the workflow can use in auto-refinement disclosures, e.g. "Tightened Goals; added explicit Non-Goals; marked 2 assumptions."
- **Never ask the user a question directly.** If there is a blocking unknown, add it to `## Open Questions` in the enhanced prompt with the `BLOCKING:` prefix. The validator escalates it and the workflow surfaces it at the next gate.
- **No file writes.** No session files, no handoffs, no logs. State is returned in-memory to the caller.
- **Never self-bypass validation.** Do not tell the workflow "this is ready, skip validation". Always signal that `validate-prompt-intent` must run next.

## Quality Bar

An enhanced prompt is good when:

- A competent engineer could act on it without further clarification.
- Every goal is testable.
- Scope, constraints, and success criteria are all present.
- Assumptions are visible, not hidden.
- It is shorter and sharper than the raw prompt, not longer for its own sake.

## Anti-Patterns

- Inventing features the user never asked for.
- Padding with generic best-practice advice.
- Converting a simple request into a multi-phase program.
- **Listing blocking unknowns as plain Open Questions without the `BLOCKING:` prefix** — the validator will not escalate them and the workflow will hand the user a half-formed prompt.
- **Returning an enhanced prompt without signalling that `validate-prompt-intent` must run next** — the skill's hand-off is ONE HALF of a tightly coupled pair, never a standalone output.
- **Rewriting sections the validator or user did not touch** — untouched sections stay stable across iterations to prevent thrash.
- Asking the user many questions when one well-chosen assumption would do.
- Locking in a specific stack when the user stayed neutral.
