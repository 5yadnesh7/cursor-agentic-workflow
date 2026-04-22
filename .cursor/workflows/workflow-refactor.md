---
name: workflow-refactor
description: Behavior-preserving refactor workflow. Locks behavior with characterization tests, runs a reviewer-designed plan in small green-bar steps, and verifies no behavior change. Use for renames, restructuring, module extraction, dependency upgrades without behavior change, and codebase cleanups.
skills_used:
  - prompt-enhancer
  - validate-prompt-intent
  - get-project-context
  - reviewer
  - dev-engineer
  - db-engineer
  - tester
  - performance-engineer
subagents_used:
  - subagent-prompt-intake
  - subagent-context-scout
  - subagent-code-reviewer
  - subagent-dev-engineer
  - subagent-db-engineer
  - subagent-qa-tester
  - subagent-performance-auditor
gates:
  - intent-gate
  - context-gate
  - plan-gate
  - review-gate
  - test-gate
  - destructive-gate
next_workflows:
  - workflow-release
  - workflow-migration
---

# Workflow: workflow-refactor

Change shape without changing behavior.

## Chat Rules

Follow the Chat Rules in `workflow-main.md`: no narration, no session files, one decision per turn with a reply menu on the last line. All state lives in the conversation.

## Step 1 — Intent Gate

If dispatched from `workflow-main` the gate already passed. If invoked directly via `/workflow-refactor`, run **Step 1 of `workflow-main.md` verbatim** (the tightly-coupled `prompt-enhancer` ⇄ `validate-prompt-intent` loop: every enhancer run is immediately followed by a validator run; only `approve` exits the loop; user `refine <feedback>` re-enters at 1a; cap on validator-driven refines is 3).

Capture the refactor goal and **explicit non-goal**: no observable behavior change. **intent-gate**.

## Step 2 — Context Gate

Ensure `PROJECT_CONTEXT.md` covers the target surface; if not, dispatch `subagent-context-scout` in `focused` mode. **context-gate**.

## Step 3 — Scope Classification

Is this truly a refactor, or does it cross into:

- **Migration** (framework / engine / major rewrite) → redirect to `workflow-migration`.
- **Feature** (any behavior change) → redirect to `workflow-feature`.

If scope slips mid-workflow, pause and re-classify at the next gate.

## Step 4 — Characterization Tests

1. Dispatch `subagent-qa-tester` to write tests that capture *current* behavior of the target surface (endpoints, functions, UI flows). Persistent automation requires **automation-gate** first.
2. Tests must go green before any refactor change.
3. **test-gate** (mandatory) on the green baseline.

## Step 5 — Plan

1. Dispatch `subagent-code-reviewer` with `scope=refactor-plan` to propose a small-step plan: each step is a named rename, extraction, move, or inline that keeps the suite green.
2. Call out high-risk areas (shared utilities, implicit invariants, hot paths).
3. **plan-gate** (mandatory): approve the step list and order.

## Step 6 — Execute Small Steps

For each step in the plan:

1. Dispatch `subagent-dev-engineer` (or `subagent-db-engineer` for schema-only refactors) to apply the step.
2. Run full suite; must stay green.
3. Dispatch `subagent-code-reviewer` to spot-check the diff. **review-gate** every 3–5 steps or at any risky step.
4. Any schema or data change → **destructive-gate** (triggered when `subagent-db-engineer` returns `DESTRUCTIVE_GATE_REQUIRED`).

## Step 7 — Performance Sanity (conditional)

If the refactor touches a hot path, dispatch `subagent-performance-auditor` to compare before/after against recorded baselines. Regressions block progress.

## Step 8 — Final Review

1. Full diff review by `subagent-code-reviewer`. **review-gate** (mandatory).
2. `subagent-qa-tester` final sweep. **test-gate** (mandatory).

## Step 9 — Docs Light-touch

Update inline module notes and refresh `PROJECT_CONTEXT.md` by dispatching `subagent-context-scout` in `focused` mode only if module boundaries moved. No new ADR unless a decision changed.

## Step 10 — Close Out

Send one final chat turn summarizing the run: steps executed, tests green at each step, any perf deltas, and files/modules moved.

## Safety Rules

- No public API or DB shape changes allowed here. Route to `workflow-feature` or `workflow-migration`.
- Every step must keep the suite green. Red tests = revert the step and replan.
- No "quick wins" outside the approved plan.
