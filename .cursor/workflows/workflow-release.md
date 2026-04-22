---
name: workflow-release
description: Ships an approved change set to users. Runs release-manager to freeze scope, cut version, produce release notes, and drive progressive rollout with devops, with tester smokes and incident-commander on standby. Use after a feature, fix, or migration is merge-ready, or to coordinate a standalone release.
skills_used:
  - prompt-enhancer
  - validate-prompt-intent
  - release-manager
  - devops
  - tester
  - doc-writer
  - incident-commander
  - performance-engineer
gates:
  - intent-gate
  - release-gate
  - rollout-gate
  - docs-gate
  - destructive-gate
next_workflows:
  - workflow-incident
---

# Workflow: workflow-release

Ship safely with explicit gates at every environment.

## Chat Rules

Follow the Chat Rules in `workflow-main.md`: no narration, no session files, one decision per turn with a reply menu on the last line. All state lives in the conversation.

## Step 1 — Intent Gate

If dispatched from `workflow-main` the gate already passed. If invoked directly via `/workflow-release`, run **Step 1 of `workflow-main.md` verbatim** (the tightly-coupled `prompt-enhancer` ⇄ `validate-prompt-intent` loop: every enhancer run is immediately followed by a validator run; only `approve` exits the loop; user `refine <feedback>` re-enters at 1a; cap on validator-driven refines is 3).

Capture:

- Target release (scope summary, commit / branch / tag).
- Release type: `patch | minor | major | hotfix`.
- Rollout plan preference: `canary` vs `blue-green` vs `full` (release-manager will validate).

**intent-gate** (mandatory).

## Step 2 — Pre-Release Checks

`release-manager` verifies:

- All gates from upstream workflows passed (review, test, audits where applicable).
- Green build, green tests, clean audit scans.
- Migration plan present if DB changes.
- Feature flags set appropriately.
- Rollback plan documented.

Missing item → pause and route back to the right workflow (`workflow-feature` / `workflow-fix` / `workflow-audit` / `workflow-docs`).

## Step 3 — Release Manifest

`release-manager` produces:

- Version number (following repo's scheme).
- Release notes draft (user-facing + engineering).
- Change manifest (commits, artifacts, migrations, flags).
- Rollback plan.
- `doc-writer` polishes release notes and updates CHANGELOG.

**docs-gate** (mandatory): approve the release notes.

**release-gate** (mandatory): approve the full manifest.

## Step 4 — Build & Sign

`devops` runs the release pipeline:

- Build artifacts.
- Run final test suite.
- Sign artifacts / images.
- Attach SBOM and scan results.
- Publish to registry (no user-visible change yet).

Any DB migration that is destructive (drop / rename / backfill with possible data loss) → **destructive-gate**.

## Step 5 — Progressive Rollout

For each environment in order (dev → staging → canary → prod):

1. `devops` promotes.
2. `tester` runs smoke tests and critical user journeys.
3. `performance-engineer` watches golden signals vs baseline.
4. **rollout-gate** (mandatory) before advancing.
5. If anomaly → pause and escalate to `workflow-incident`; rollback if needed.

Pre-flight each gate with a compact status:

```
Env: <env>
Smoke:   pass / fail / partial (link to findings)
Golden:  latency <n>ms p95 (baseline <m>), errors <x>%, saturation <y>%
Health:  <healthy | degraded | unhealthy>
Next:    promote | hold | rollback
```

## Step 6 — Post-Release

1. `release-manager` closes the release:
   - Tag the repo.
   - Publish release notes.
   - Move tracked tickets to Released.
2. `devops` confirms dashboards and alerts are tied to this release.
3. `doc-writer` finalizes the CHANGELOG entry.
4. Refresh `PROJECT_CONTEXT.md` if infra or public API changed.

## Step 7 — Watch Window

Stay attentive in chat for a defined watch window (default 60 min or release-manager's recommendation). If SLOs breach → `workflow-incident`.

## Step 8 — Close Out

Send one final chat turn summarizing the release: version shipped, environments promoted, smoke/golden-signal results, release notes link, and the status of the watch window.

## Safety Rules

- No rollout advancement without `rollout-gate`.
- No destructive DB change in the release without `destructive-gate` + a tested rollback plan.
- Prod rollout happens only after staging smoke passes.
- Release-manager or devops can call a halt at any stage; agent must obey.
