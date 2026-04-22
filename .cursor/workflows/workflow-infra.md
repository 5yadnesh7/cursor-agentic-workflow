---
name: workflow-infra
description: Infrastructure-only workflow. Plans and applies IaC and platform changes through devops with research as needed, with security audit, performance baseline, and progressive rollout. Use for cloud resource changes, CI/CD changes, observability setup, networking/identity, cluster work, or pipeline changes.
skills_used:
  - prompt-enhancer
  - validate-prompt-intent
  - get-project-context
  - researcher
  - devops
  - security-auditor
  - performance-engineer
  - reviewer
  - tester
  - doc-writer
  - release-manager
  - incident-commander
gates:
  - intent-gate
  - context-gate
  - plan-gate
  - security-gate
  - apply-gate
  - rollout-gate
  - destructive-gate
next_workflows:
  - workflow-release
  - workflow-incident
---

# Workflow: workflow-infra

Infra work with plans, gates, and blast-radius awareness.

## Chat Rules

Follow the Chat Rules in `workflow-main.md`: no narration, no session files, one decision per turn with a reply menu on the last line. All state lives in the conversation.

## Step 1 — Intent Gate

If dispatched from `workflow-main` the gate already passed. If invoked directly via `/workflow-infra`, run **Step 1 of `workflow-main.md` verbatim** (the tightly-coupled `prompt-enhancer` ⇄ `validate-prompt-intent` loop: every enhancer run is immediately followed by a validator run; only `approve` exits the loop; user `refine <feedback>` re-enters at 1a; cap on validator-driven refines is 3).

Capture:

- Change type: `cloud resource | network | identity | secrets | cluster | pipeline | observability | cost / FinOps`.
- Environments in scope: `dev | staging | prod`.
- Blast radius estimate.
- Constraints (compliance, cost ceilings, change window).

**intent-gate** (mandatory).

## Step 2 — Context Gate

Read existing IaC layout from `PROJECT_CONTEXT.md` and `.cursor/rules/` (Terraform / infra rules). Refresh if stale. **context-gate**.

## Step 3 — Research (conditional)

If picking a new provider / tool / pattern → dispatch `workflow-research`, or invoke `researcher` inline and return to this workflow.

## Step 4 — Plan

Invoke `devops`:

- Produce the concrete change plan: files to touch, modules involved, drift check results, `terraform plan` / equivalent output, rollout order, rollback plan.
- Include observability changes (dashboards, SLOs, alerts with runbooks) and FinOps tags.
- Flag destructive changes (replace, delete, recreate-with-data-loss).

**plan-gate** (mandatory).

## Step 5 — Security & Compliance Review

Invoke `security-auditor` on the plan for:

- IAM surface.
- Network exposure.
- Secret handling.
- Public / private data paths.
- Compliance posture (if relevant).

**security-gate** (mandatory). Remediations fed back to `devops`.

## Step 6 — Apply Gate

Final go/no-go before any apply:

```
┌─── Approval Gate: apply-gate ───┐
│ Stage:   pre-apply
│ Env:     <env>
│ Plan:    summary (full plan already shown above in chat)
│ Destr:   <list of destructive actions or "none">
│ Roll:    <rollback plan summary>
│
│ Reply: approve | modify <feedback> | abort
└─────────────────────────────────┘
```

Destructive items also require **destructive-gate**.

## Step 7 — Progressive Apply

For each environment in order (dev → staging → prod):

1. `devops` applies the plan.
2. `tester` runs smoke / synthetic checks.
3. `performance-engineer` spot-checks golden signals.
4. **rollout-gate** (mandatory) before advancing.
5. Any anomaly → `workflow-incident`.

## Step 8 — Post-Apply

- `devops` verifies drift = 0, alerts firing correctly in non-prod tests, dashboards owned.
- `doc-writer` updates:
  - Runbook(s) for any new alerts.
  - ADR if a notable decision was made.
  - `PROJECT_CONTEXT.md` via `get-project-context: refresh` if topology changed.

## Step 9 — Close Out

Send one final chat turn summarizing the run: change type, environments applied, plan highlights, security remediations, drift status, new alerts/runbooks, and any follow-ups.

## Safety Rules

- No direct prod apply without dev + staging success.
- Every new alert must link to a runbook (`doc-writer` creates it if missing) before **apply-gate** for prod.
- Destructive actions always hold **destructive-gate**, even in dev if data loss possible.
- IAM broadening is never a "minor" — always goes through `security-gate`.
