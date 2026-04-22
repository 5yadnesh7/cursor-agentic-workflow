# Workflow: Audit
Specialist audit workflow. Runs security-auditor, performance-engineer, and/or accessibility-auditor against a defined scope, consolidates severity-tagged findings, and produces a remediation plan that hands off to the correct builder workflow.

## What This Does

Runs `.cursor/workflows/workflow-audit.md`: pick audit mode(s) — security / performance / accessibility — and scope → run each auditor → consolidate findings → audit-gate → prioritized remediation plan → remediation-gate → dispatch to builder workflows.

## Steps

1. Open `.cursor/workflows/workflow-audit.md` and follow every step.
2. Obey the **Chat Rules** referenced at the top of that workflow: no narration, no session files, one decision per turn with a reply menu on the last line.
3. If invoked directly (not from another workflow), run the silent `prompt-enhancer` ⇄ `validate-prompt-intent` loop first and present the enhanced prompt for approval.
4. If `PROJECT_CONTEXT.md` is stale for the target, run `get-project-context: focused`.
5. Pause at mandatory gates: intent (mode + scope + thresholds), context, audit (findings accepted), remediation (plan approved).
6. Dispatch remediation items to `/workflow-fix`, `/workflow-feature`, `/workflow-infra`, or `/workflow-incident` (if live impact).

## Inputs

Use any text after `/workflow-audit` to list audit mode(s) and target scope. If empty, ask the user.

## Must Not

- Narrate bookkeeping or create session files.
- Claim a "clean audit" without stating exactly what was in scope and what was not.
- Let production impact discovered during the audit wait — escalate to `/workflow-incident`.
- Quietly reclassify auditor severities; any reclassification must be stated in chat with a reason.
