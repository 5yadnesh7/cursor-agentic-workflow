---
name: workflow-audit
description: Specialist audit workflow. Runs one or more of security-auditor, performance-engineer, and accessibility-auditor against a defined scope, consolidates findings, and produces a remediation plan that hands off to the correct builder skill. Use when the user asks for a security review, perf investigation, or a11y audit, or when another workflow routes a finding here.
skills_used:
  - prompt-enhancer
  - validate-prompt-intent
  - get-project-context
  - security-auditor
  - performance-engineer
  - accessibility-auditor
  - reviewer
  - doc-writer
subagents_used:
  - subagent-prompt-intake
  - subagent-context-scout
  - subagent-security-auditor
  - subagent-performance-auditor
  - subagent-accessibility-auditor
  - subagent-code-reviewer
  - subagent-doc-writer
gates:
  - intent-gate
  - context-gate
  - audit-gate
  - remediation-gate
next_workflows:
  - workflow-fix
  - workflow-feature
  - workflow-infra
  - workflow-incident
---

# Workflow: workflow-audit

Run expert audits and produce a remediation plan.

## Chat Rules

Follow the Chat Rules in `workflow-main.md`: no narration, no session files, one decision per turn with a reply menu on the last line. All state lives in the conversation.

## Step 1 — Intent Gate

If dispatched from `workflow-main` or another workflow, the gate already passed. If invoked directly via `/workflow-audit`, run **Step 1 of `workflow-main.md` verbatim** (the tightly-coupled `prompt-enhancer` ⇄ `validate-prompt-intent` loop: every enhancer run is immediately followed by a validator run; only `approve` exits the loop; user `refine <feedback>` re-enters at 1a; cap on validator-driven refines is 3).

Capture:

- Audit mode(s): `security` | `performance` | `accessibility` (one or more).
- Target scope: repo, subtree, service, endpoint(s), UI surface.
- Threshold / SLOs / WCAG level / threat model depth.

**intent-gate** (mandatory).

## Step 2 — Context Gate

Ensure `PROJECT_CONTEXT.md` covers the target. If not, dispatch `subagent-context-scout` with `mode=focused` on the target subtree. **context-gate** passes when the scout returns a report covering the audit scope.

## Step 3 — Run Audits (parallel, single-turn dispatch)

For each selected mode, dispatch the corresponding subagent. When multiple modes are selected, emit all the `Task` calls in a single assistant turn so they run concurrently:

### 3a. Security

Dispatch `subagent-security-auditor` (wraps `security-auditor`; see `.cursor/workflows/SUBAGENT_DISPATCH.md`) with `scope`, optional `change`, and optional `compliance`. Returns an `AUDIT_REPORT` with Critical / High / Medium / Low findings.

### 3b. Performance

Dispatch `subagent-performance-auditor` (wraps `performance-engineer`) with `target`, `workload`, and `slo`. Returns a `PERF_REPORT` with before/after numbers and ranked bottlenecks.

### 3c. Accessibility

Dispatch `subagent-accessibility-auditor` (wraps `accessibility-auditor`) with `target`, `scope`, and `standard` (default WCAG 2.2 AA). Returns an `A11Y_REPORT` with WCAG-cited findings.

Any auditor returning `CONTEXT_REQUIRED` means the context pass for its surface is insufficient — dispatch `subagent-context-scout` targeted at that area, then re-dispatch the auditor.

## Step 4 — Consolidate

Optionally dispatch `subagent-code-reviewer` with `scope=<audit-consolidation>` so it composes a merged audit summary rendered in chat, or have the workflow merge the three reports directly:

- Findings grouped by severity.
- Each finding: title, evidence, impact, recommended fix, owner subagent (`subagent-dev-engineer` / `subagent-db-engineer` / `subagent-devops-engineer`).
- Quick wins vs deep work.

## Step 5 — Audit Gate

**audit-gate** (mandatory). User options:

- `approve` — accept findings as-is and move to remediation plan.
- `modify <feedback>` — re-run specific auditors with adjustments.
- `defer <items>` — move items to follow-ups.
- `abort`.

## Step 6 — Remediation Plan

Build a prioritized plan:

- Blockers (critical/high) — must fix before next release.
- Majors — fix within next sprint / defined window.
- Minors — backlog.

**remediation-gate** (mandatory).

## Step 7 — Hand Off To Builders

Dispatch remediation items to:

- Code changes → `workflow-fix` or `workflow-feature`.
- Infra / IaC / CI → `workflow-infra`.
- Live production impact during audit → `workflow-incident`.

Each dispatched workflow carries the remediation items forward from chat.

## Step 8 — Docs

Dispatch `subagent-doc-writer` to record:

- Short audit report for the repo (under `docs/audits/` or project convention).
- Updated runbook entries where relevant.

## Step 9 — Close Out

Send one final chat turn summarizing the audit: modes run, scope, findings by severity, remediations dispatched, and the audit report path.

## Safety Rules

- A "clean audit" must state exactly what was in scope and what was not.
- Findings with production impact during audit → escalate immediately.
- Severity ratings are the auditor's call; user can reclassify only by stating a reason in chat.
