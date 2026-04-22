---
name: workflow-migration
description: Large cross-cutting migration workflow using brainstorm's expand -> migrate -> contract discipline. Coordinates db-engineer, data-engineer, dev-engineer, devops, tester, and release-manager across phases with checkpoint gates between each. Use for framework upgrades, engine swaps, monolith-to-services splits, multi-tenant isolation, datacenter moves, and major API rewrites.
skills_used:
  - prompt-enhancer
  - validate-prompt-intent
  - get-project-context
  - brainstorm
  - researcher
  - db-engineer
  - data-engineer
  - dev-engineer
  - devops
  - reviewer
  - tester
  - security-auditor
  - performance-engineer
  - doc-writer
  - release-manager
  - incident-commander
gates:
  - intent-gate
  - context-gate
  - approach-gate
  - plan-gate
  - phase-gate
  - cutover-gate
  - rollback-gate
  - destructive-gate
  - contract-gate
next_workflows:
  - workflow-release
  - workflow-incident
---

# Workflow: workflow-migration

Change the substrate without losing users.

## Chat Rules

Follow the Chat Rules in `workflow-main.md`: no narration, no session files, one decision per turn with a reply menu on the last line. All state lives in the conversation. For migrations that span days, the user carries the key decisions into subsequent chats; the workflow asks for the relevant prior decisions when resuming.

## Step 1 — Intent Gate

If dispatched from `workflow-main` the gate already passed. If invoked directly via `/workflow-migration`, run **Step 1 of `workflow-main.md` verbatim** (the tightly-coupled `prompt-enhancer` ⇄ `validate-prompt-intent` loop: every enhancer run is immediately followed by a validator run; only `approve` exits the loop; user `refine <feedback>` re-enters at 1a; cap on validator-driven refines is 3).

Capture:

- Migration name and type (framework upgrade, engine swap, services split, IaC provider move, authn/authz overhaul, datacenter move, API v1 → v2, multi-tenant isolation, etc.).
- Why now; deadlines and external constraints.
- Known consumers and blast radius.

**intent-gate** (mandatory).

## Step 2 — Context Gate

Run `get-project-context: focused` on affected areas if stale or missing. **context-gate**.

## Step 3 — Strategy (brainstorm)

Invoke `brainstorm` with the **Large Migration Template**:

- Current State.
- Target State.
- Strategy options (e.g. dual-write, shadow read, parallel stack, strangler fig, big bang — with trade-offs).
- Consumers and compatibility policy.
- Data plan and backfill.
- Observability and kill switch.
- Exit criteria.

Use `researcher` as needed for unknowns. **approach-gate** (mandatory): pick a strategy.

## Step 4 — Plan

`brainstorm` produces the phase plan following **Expand → Migrate → Contract**:

### Expand
- Introduce new shape alongside old.
- Both paths supported; no consumers switched yet.
- Tests cover old and new.

### Migrate
- Shift consumers in controlled waves.
- Feature flags, dual-write, shadow read as appropriate.
- Reconciliation reports.

### Contract
- Remove old shape.
- Drop migrations, deprecate endpoints, delete dead paths.

**plan-gate** (mandatory): approve the phase plan, consumer waves, data plan, and kill switch.

## Step 5 — Execute Phases

For each phase (Expand, Migrate, Contract):

1. Dispatch to owner skills:
   - Schema / engine → `db-engineer`.
   - Pipelines / streams → `data-engineer`.
   - Application code → `dev-engineer`.
   - Infra → `devops`.
2. Every slice: `reviewer` → **review-gate**; `tester` → **test-gate**.
3. Any destructive op → **destructive-gate**.
4. When a consumer wave goes live → progressive rollout by `devops` (see `workflow-release`'s rollout pattern) with **cutover-gate** before each wave.
5. `performance-engineer` monitors against baselines; `security-auditor` re-checks if trust boundaries moved.
6. End of phase: present a compact phase summary in chat (scope completed, deltas observed, risks seen, open items).
7. **phase-gate** (mandatory): explicit approval before moving to the next phase.

## Step 6 — Contract Gate

Before removing old code / schema / infra:

- Confirm all consumers migrated (reconciliation, traffic, telemetry).
- Confirm backups / snapshots exist.
- Confirm rollback plan still valid for an emergency rollback of the contract step itself.

**contract-gate** + **destructive-gate** (both mandatory).

## Step 7 — Release & Watch

- `release-manager` ships each contract step like a normal release (via `workflow-release`).
- `incident-commander` on standby. Breach → `workflow-incident`.
- After final contract, run a long watch window (release-manager recommends).

## Step 8 — Documentation

- `doc-writer` updates architecture, ADRs for migration decisions, runbooks, and the CHANGELOG.
- Refresh `PROJECT_CONTEXT.md`.

## Step 9 — Close Out

Send one final chat turn summarizing the migration: strategy chosen, phases executed, consumers migrated, data reconciled, infra state, and any residual risks for the watch window.

## Safety Rules

- Never skip **phase-gate**. Speeding through phases is how migrations fail.
- Never combine Migrate and Contract in one deployment — they are separate phases with separate gates.
- Always define a kill switch before starting Migrate; verify it works in a staging drill.
- Rollback plan must remain valid at every step; when it no longer does, stop and re-plan.
- Cross-cutting data ops always require **destructive-gate** + verified backups.
