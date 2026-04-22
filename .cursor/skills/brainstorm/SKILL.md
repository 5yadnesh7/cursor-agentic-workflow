---
name: brainstorm
description: Takes a validated prompt and performs end-to-end brainstorming, requirement coverage, impact analysis on existing behavior, clarifying Q&A with the user, approach comparison, approval gating, high-level design, and an implementation plan. Use after validate-prompt-intent approves a prompt and before any code, schema, or infra work begins, or whenever a feature needs deliberate design thinking before execution.
---

# Brainstorm

Turns a validated prompt into an agreed-upon approach, design, and implementation plan. Forces explicit trade-offs, impact analysis, and user approval before any builder skill runs.

## When To Use

- After `validate-prompt-intent` approves an enhanced prompt.
- For any non-trivial feature, refactor, migration, or architectural change.
- Whenever more than one reasonable approach exists.
- Before handing work to `db-engineer`, `dev-engineer`, or `devops`.

## Inputs

- `approved_prompt`: the validated enhanced prompt.
- `repo_context` (optional): relevant files, modules, workspace rules.
- `constraints` (optional): non-negotiables from the user or org.

## Workflow

```
Brainstorm:
- [ ] Step 1: Expand requirements
- [ ] Step 2: Map impact on existing behavior
- [ ] Step 3: Identify unknowns and ask user
- [ ] Step 4: Generate candidate approaches
- [ ] Step 5: Compare trade-offs and recommend
- [ ] Step 6: Get explicit user approval on approach
- [ ] Step 7: Produce high-level design
- [ ] Step 8: Produce implementation plan
- [ ] Step 9: Hand off to the right builder skill
```

### Step 1: Expand Requirements

From the approved prompt, enumerate:

- Functional requirements (what the system must do)
- Non-functional requirements (performance, security, reliability, observability, accessibility, i18n)
- Edge cases and failure modes
- Data and state requirements
- User-visible behavior, including empty, loading, and error states

Flag anything missing rather than silently filling it in.

### Step 2: Impact Analysis On Existing Behavior

For each requirement, identify:

- Affected modules, services, schemas, APIs, UI surfaces.
- Backward-compatibility risks.
- Contract changes (public APIs, events, message shapes, DB columns).
- Migration needs (data, config, infra).
- Tests and docs that must change.
- Downstream consumers who could break.

Produce a short impact table:

```
| Area | Change | Risk | Mitigation |
|------|--------|------|------------|
| ...  | ...    | ...  | ...        |
```

### Step 3: Ask The User

Ask only questions whose answers change the design. Use the AskQuestion tool when available, with sensible defaults. Good question shapes:

- "Should this be backward compatible with v1 clients?"
- "Is real-time or near-real-time acceptable?"
- "Are we optimizing for cost, latency, or developer speed first?"

Keep it focused: three questions maximum per round.

### Step 4: Generate Candidate Approaches

Produce at least two, usually three, distinct approaches. For each:

```
### Approach <name>
- Summary: <one sentence>
- How it works: <2–4 bullets>
- Pros: <bullets>
- Cons: <bullets>
- Impact: <bullets>
- Cost / effort: <S / M / L>
- Risk: <low / medium / high>
```

Avoid straw-man options; each approach must be genuinely viable.

### Step 5: Compare And Recommend

Produce a comparison table keyed to the user's priorities, then give a clear recommendation with a one-paragraph rationale.

```
| Criterion | Approach A | Approach B | Approach C |
|-----------|------------|------------|------------|
| ...       | ...        | ...        | ...        |
```

### Step 6: Approval Gate

Present the recommendation and ask explicitly:

> "Shall I proceed with Approach <X>, or would you prefer <Y> / <Z>, or a hybrid?"

Do not move to design until the user confirms. Record the chosen approach verbatim.

### Step 7: High-Level Design

Once approved, produce a design doc with these sections:

```
# Design: <feature>

## Overview
<1–2 paragraphs>

## Architecture
<components, responsibilities, interactions; a textual diagram is fine>

## Data Model
<entities, fields, relationships, indexes, invariants>

## Interfaces
<public APIs, events, CLI, UI contracts>

## Behavior
<key flows, state transitions, edge cases>

## Non-Functional
<performance, security, reliability, observability, cost>

## Rollout
<feature flags, migrations, backfills, phased release>

## Risks & Mitigations
<bullets>
```

Keep each section tight; link out to reference files if needed.

### Step 8: Implementation Plan

Break work into small, ordered, independently verifiable steps:

```
# Plan

## Milestones
1. <milestone> — acceptance: <observable criterion>
2. <milestone> — acceptance: <observable criterion>

## Tasks
- [ ] <task> → owner skill: <db-engineer | dev-engineer | devops | tester>
- [ ] <task> → owner skill: <...>

## Test Strategy
- Unit: <what>
- Integration: <what>
- E2E: <what> (coordinate with `tester`)
- Non-functional: <load, security, etc.>

## Rollback Plan
<how to revert safely>
```

Each task should be small enough that a single skill invocation can deliver it.

### Step 9: Hand Off

Recommend the next skill(s) based on the plan:

- Unfamiliar repo or thin context → `get-project-context` first
- Schema or data work → `db-engineer`
- Data pipelines / ETL / streaming → `data-engineer`
- ML / LLM / RAG / agents → `ml-engineer`
- Feature code, APIs, services, UI → `dev-engineer`
- Infra, CI/CD, deployment → `devops`
- Verification → `tester`
- Unknown tech choices → loop through `researcher` before building
- Pre-merge review → `reviewer`; security-sensitive surfaces → `security-auditor`; a11y-impacting UI → `accessibility-auditor`
- Rollout → `release-manager`

## Large Migration Template

Use this template when the plan is a cross-cutting migration (framework upgrade, monolith-to-services split, cloud move, DB engine swap, authn/authz overhaul, major API rewrite, multi-tenant isolation, datacenter move).

Apply the **expand → migrate → contract** discipline at system level, not just schema level.

```
# Migration Plan: <name>

## Why
<one paragraph: the forcing function and the cost of doing nothing>

## Current State
<what exists today, including hidden consumers and manual flows>

## Target State
<what "done" looks like; define acceptance criteria measurably>

## Strategy
- Pattern: <expand/migrate/contract | strangler fig | blue/green | dual-run | shadow>
- Scope slices: <how the work is cut into shippable, reversible pieces>

## Phases

### Phase 1: Expand
- Land the new surface alongside the old; no consumer switches yet.
- Dual write or dual read where needed.
- Feature flags gating the new path.
- Observability in place for both paths.

### Phase 2: Migrate
- Migrate consumers one slice at a time (by tenant / region / route / client version).
- Compare outputs of old vs new (shadow or replay) before cutting traffic.
- Backfill data with batched, throttled, idempotent jobs.
- Maintain a real rollback path at every slice.

### Phase 3: Contract
- Remove old surface only after all consumers are migrated and a stabilization window has passed.
- Clean up flags, dual-writes, and compatibility shims.
- Archive or delete old data per retention policy.

## Consumers & Communication
| Consumer | Owner | Migration step | Deadline | Notified |
|----------|-------|----------------|----------|----------|

## Data Plan
<what moves, how, with what safety checks; coordinate with db-engineer / data-engineer>

## Risks & Mitigations
- <risk> → <mitigation> → <owner>

## Kill Switch
<how to halt the migration and return to a safe state at each phase>

## Exit Criteria
- <observable, measurable conditions for declaring the migration done>
```

Hand off to builder skills per phase and require a checkpoint review between phases.

## Quality Bar

A brainstorm is good when:

- Requirements are complete enough that design choices are informed, not guessed.
- Impact on existing behavior is visible and owned.
- The user has explicitly approved one approach over the alternatives.
- The plan is executable by builder skills with no re-design needed.

## Anti-Patterns

- Producing only one approach and calling it a comparison.
- Skipping the approval gate and jumping to code.
- Deep-diving implementation details inside the design doc.
- Hiding risks instead of naming them.
- Re-deriving requirements that the enhanced prompt already locked down.
