---
name: workflow-new-project
description: Greenfield end-to-end workflow that takes a project from zero to first production release. Exercises the full skill set: prompt handling, research, brainstorm, context seeding, database and infra foundations, scaffolding, iterative feature build, quality audits, documentation, and first release. Every phase ends with an approval gate.
subagents_used:
  - subagent-prompt-intake
  - subagent-researcher
  - subagent-context-scout
  - subagent-db-engineer
  - subagent-dev-engineer
  - subagent-devops-engineer
  - subagent-code-reviewer
  - subagent-qa-tester
  - subagent-security-auditor
  - subagent-performance-auditor
  - subagent-accessibility-auditor
  - subagent-doc-writer
  - subagent-debug-investigator
dispatch_note: |
  When any step below says "invoke <skill>", dispatch to the corresponding subagent from `.cursor/workflows/SUBAGENT_DISPATCH.md` when one exists. Skills without a subagent wrapper (brainstorm, data-engineer, ml-engineer, release-manager, incident-commander) are invoked inline by this workflow.
skills_used:
  - prompt-enhancer
  - validate-prompt-intent
  - researcher
  - brainstorm
  - get-project-context
  - db-engineer
  - data-engineer
  - ml-engineer
  - devops
  - dev-engineer
  - reviewer
  - tester
  - security-auditor
  - performance-engineer
  - accessibility-auditor
  - doc-writer
  - release-manager
  - incident-commander
  - debugging
gates:
  - intent-gate
  - research-gate
  - approach-gate
  - design-gate
  - schema-gate
  - infra-plan-gate
  - implementation-gate
  - review-gate
  - test-gate
  - automation-gate
  - security-gate
  - docs-gate
  - release-gate
  - rollout-gate
  - destructive-gate
next_workflows:
  - workflow-incident
  - workflow-release
---

# Workflow: workflow-new-project

End-to-end greenfield delivery. Use for any "start a new project" prompt.

## Chat Rules

Follow the Chat Rules in `workflow-main.md`: no narration, no session files, one decision per turn with a reply menu on the last line. All state lives in the conversation. Each phase ends with a compact phase-summary chat turn before asking for the phase gate.

## Phase 0 — Intent

1. If not already done by `workflow-main`, run **Step 1 of `workflow-main.md` verbatim** (the tightly-coupled `prompt-enhancer` ⇄ `validate-prompt-intent` loop: every enhancer run is immediately followed by a validator run; only `approve` exits the loop; user `refine <feedback>` re-enters at 1a; cap on validator-driven refines is 3).
2. **intent-gate** (mandatory): confirm project goal, audience, scope, non-goals, constraints.

## Phase 1 — Discovery & Direction

1. Invoke `researcher` (depth=`standard` by default) for:
   - Recommended stack given constraints.
   - Comparable systems and patterns.
   - Current best practices and pitfalls.
2. Return research synthesis to the workflow for chat display; do not persist session files.
3. **research-gate** (optional; skip if stack already decided): confirm research is enough to proceed.
4. Invoke `brainstorm`:
   - Expand requirements, impact on existing behavior (N/A for greenfield but still capture future-proofing).
   - Produce 2–3 architecture candidates with trade-offs.
5. **approach-gate** (mandatory): user explicitly picks an approach.
6. Invoke `brainstorm` again to produce the high-level design and the implementation plan.
7. **design-gate** (mandatory): approve the design.
8. Send a compact Phase 1 summary chat turn.

## Phase 2 — Foundations

1. Invoke `get-project-context` in `full` mode to seed an initial `PROJECT_CONTEXT.md` from the approved design and any starter files.
2. Invoke `db-engineer` to design:
   - Engine choice, data model, constraints, indexes.
   - Migration framework setup.
3. **schema-gate** (mandatory): approve the schema and migration approach.
4. Invoke `devops` to plan:
   - Repo scaffolding conventions.
   - CI/CD baseline (build, test, lint, type, scan, artifact signing).
   - Environments and promotion topology.
   - IaC baseline (network, identity, secrets, observability).
   - Container / packaging strategy.
5. **infra-plan-gate** (mandatory): approve the infra plan.
6. Send a compact Phase 2 summary chat turn.

## Phase 3 — Scaffolding

1. Invoke `dev-engineer` to scaffold:
   - Backend skeleton: config, errors, logging, health, auth stub, base routes.
   - Frontend skeleton (if applicable): app shell, routing, state, API client.
   - Cross-cutting: feature flags, observability hooks, error boundaries.
   - Initial tests scaffold (unit harness, integration harness).
2. Invoke `reviewer` to check scaffold quality against PROJECT_CONTEXT.md.
3. **review-gate** (mandatory): all blockers resolved.
4. **implementation-gate** (mandatory): user approves proceeding to feature build.
5. Send a compact Phase 3 summary chat turn.

## Phase 4 — Iterative Feature Build

For each slice from the implementation plan:

1. Invoke `researcher` only if new unknowns appear.
2. Invoke specialists if the slice requires:
   - `db-engineer` — schema change → **schema-gate**.
   - `data-engineer` — pipeline / stream.
   - `ml-engineer` — ML / LLM / RAG.
3. Invoke `dev-engineer`.
4. Invoke `reviewer`. **review-gate** (mandatory).
5. Invoke `tester`. Before writing automation, **automation-gate** (mandatory if Playwright/equivalent). **test-gate** (mandatory).
6. **implementation-gate** (mandatory): user approves starting next slice, or ends the build phase.
7. Every few slices, send a compact progress summary chat turn.

Any destructive change triggers **destructive-gate**.

## Phase 5 — Quality Bar

1. Invoke `security-auditor` → **security-gate** (mandatory): approve remediation plan.
2. Apply fixes via `dev-engineer` / `db-engineer` / `devops`; re-run audit on impacted areas.
3. Invoke `performance-engineer` to baseline SLOs and do targeted tuning.
4. Invoke `accessibility-auditor` (if user-facing UI); fixes via `dev-engineer`.
5. Final pass: `reviewer` over the whole change-set.
6. **review-gate** (mandatory).
7. Send a compact Phase 5 summary chat turn.

## Phase 6 — Documentation

1. Invoke `doc-writer` to produce:
   - README.
   - Architecture overview.
   - ADRs for major decisions.
   - API reference.
   - Runbook(s).
   - Onboarding guide.
   - Initial CHANGELOG entry.
2. Refresh `PROJECT_CONTEXT.md` via `get-project-context` in `refresh` mode.
3. **docs-gate** (mandatory): approve the doc set.

## Phase 7 — First Release

1. Invoke `release-manager` to:
   - Freeze scope.
   - Choose initial version (e.g. `0.1.0` or project convention).
   - Generate release notes (via `doc-writer`).
   - Prepare the release pipeline (`devops` coordinates).
2. **release-gate** (mandatory): approve the release manifest.
3. Progressive rollout (coordinated by `devops`):
   - Dev → **rollout-gate**.
   - Staging → `tester` smoke → **rollout-gate**.
   - Prod canary → monitor golden signals → **rollout-gate**.
   - Prod full.
4. `incident-commander` is on standby. Any SLO breach → escalate to `workflow-incident`; return here after resolution.
5. On full rollout, mark release done.

## Phase 8 — Close Out

1. Refresh `PROJECT_CONTEXT.md` (`get-project-context: refresh`).
2. Update runbooks and release notes with anything learned during rollout.
3. Send one final chat turn summarizing: phases completed, gates passed, artifacts delivered (with repo paths), open follow-ups.

## Safety Rules

- Never advance past any mandatory gate without explicit `approve`.
- Never run prod rollout without staging smoke passing.
- Any destructive infra/data op requires **destructive-gate**.
