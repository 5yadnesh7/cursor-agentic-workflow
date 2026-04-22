---
name: workflow-feature
description: Adds a feature to an existing project end-to-end. Ensures project context is fresh, runs brainstorm with approval, optionally invokes researcher and specialist skills (db-engineer, data-engineer, ml-engineer), then implements iteratively with reviewer, tester, and audit gates, finishing with docs. Use for any new functionality or enhancement on an existing codebase.
skills_used:
  - prompt-enhancer
  - validate-prompt-intent
  - get-project-context
  - brainstorm
  - researcher
  - db-engineer
  - data-engineer
  - ml-engineer
  - dev-engineer
  - reviewer
  - tester
  - security-auditor
  - performance-engineer
  - accessibility-auditor
  - doc-writer
subagents_used:
  - subagent-prompt-intake
  - subagent-context-scout
  - subagent-researcher
  - subagent-db-engineer
  - subagent-dev-engineer
  - subagent-code-reviewer
  - subagent-qa-tester
  - subagent-security-auditor
  - subagent-performance-auditor
  - subagent-accessibility-auditor
  - subagent-doc-writer
gates:
  - intent-gate
  - context-gate
  - approach-gate
  - design-gate
  - schema-gate
  - implementation-gate
  - review-gate
  - test-gate
  - automation-gate
  - docs-gate
  - destructive-gate
next_workflows:
  - workflow-release
  - workflow-incident
---

# Workflow: workflow-feature

Add a feature on an existing codebase.

## Chat Rules

Follow the Chat Rules in `workflow-main.md`: no narration, no session files, one decision per turn with a reply menu on the last line. All state lives in the conversation.

## Step 1 — Intent Gate

If dispatched from `workflow-main` the gate already passed. If invoked directly via `/workflow-feature`, run **Step 1 of `workflow-main.md` verbatim** (the tightly-coupled `prompt-enhancer` ⇄ `validate-prompt-intent` loop: every enhancer run is immediately followed by a validator run; only `approve` exits the loop; user `refine <feedback>` re-enters at 1a; cap on validator-driven refines is 3). Then hold the **intent-gate** on the approved enhanced prompt.

## Step 2 — Context Gate

1. Check `PROJECT_CONTEXT.md` freshness and coverage for the target area.
2. If missing / stale / thin → dispatch `subagent-context-scout` in `focused` or `refresh` mode.
3. **context-gate** (mandatory): confirm the context used.

## Step 3 — Brainstorm

1. Invoke `brainstorm`:
   - Expand requirements.
   - Map impact on existing behavior.
   - Produce 2–3 approaches with trade-offs.
2. **approach-gate** (mandatory): user picks one approach.
3. Invoke `brainstorm` again to produce design + implementation plan.
4. **design-gate** (mandatory): approve the design and plan.

## Step 4 — Specialist Branches (conditional)

If the plan requires any of the following, run them first and gate:

- **Schema / data changes** → dispatch `subagent-db-engineer` → **schema-gate**. Destructive migrations surface `DESTRUCTIVE_GATE_REQUIRED` — hold **destructive-gate** and re-dispatch with approval.
- **Data pipelines / streaming** → `data-engineer` skill (no subagent wrapper; invoked inline) → **review-gate** on the pipeline contract.
- **ML / LLM / RAG** → `ml-engineer` skill (no subagent wrapper; invoked inline) → **review-gate** on the modeling plan.
- **Unknowns** → dispatch `subagent-researcher` (can be invoked at any step).

## Step 5 — Iterative Implementation

For each slice in the plan:

1. Dispatch `subagent-dev-engineer` with slice scope and the approved plan from chat.
2. Dispatch `subagent-code-reviewer` on the change. **review-gate** (mandatory).
3. Dispatch `subagent-qa-tester`. Before writing persistent automation: **automation-gate**. Then **test-gate** (mandatory).
4. **implementation-gate** (mandatory): approve moving to next slice or ending build.

Destructive ops require **destructive-gate** (triggered when any builder subagent returns `DESTRUCTIVE_GATE_REQUIRED`).

## Step 6 — Quality Audits (conditional, parallelizable)

Run when the change touches the relevant surface — dispatch in a single turn for concurrent execution:

- Auth / PII / public API → `subagent-security-auditor`.
- Hot path / scale-sensitive → `subagent-performance-auditor`.
- User-facing UI → `subagent-accessibility-auditor`.

Apply fixes via `subagent-dev-engineer` / `subagent-db-engineer` / `subagent-devops-engineer`. Re-audit impacted areas.

## Step 7 — Documentation

1. Dispatch `subagent-doc-writer` to update:
   - API reference (if contract changed).
   - README / architecture (if significant).
   - ADR (if a new decision was made).
   - CHANGELOG entry.
2. Refresh `PROJECT_CONTEXT.md` by dispatching `subagent-context-scout` with `mode=refresh`.
3. **docs-gate** (mandatory).

## Step 8 — Hand Off Or Release

- If the user wants to ship now → invoke `workflow-release` as a sub-workflow.
- Otherwise hand back to `workflow-main` (or exit) with a summary.

## Step 9 — Close Out

Send one final chat turn summarizing the run: approach picked, slices shipped, gates passed, artifacts produced (by repo path), and any follow-up items.

## Safety Rules

- Never implement before `approach-gate` + `design-gate`.
- Never merge without `review-gate` + `test-gate`.
