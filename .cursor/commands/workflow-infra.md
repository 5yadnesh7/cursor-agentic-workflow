# Workflow: Infra
Infrastructure-only workflow. Plans and applies IaC and platform changes through devops, with security audit, performance baseline, and progressive rollout across environments. Use for cloud resource changes, CI/CD pipelines, observability setup, networking/identity, cluster work, or FinOps.

## What This Does

Runs `.cursor/workflows/workflow-infra.md`: intent + blast radius → context + research (if new tool) → `devops` plan with drift check → `security-auditor` review → apply-gate → progressive apply (dev → staging → prod) with smoke + golden-signal checks → post-apply verification → docs updates.

## Steps

1. Open `.cursor/workflows/workflow-infra.md` and follow every step.
2. Obey the **Chat Rules** referenced at the top of that workflow: no narration, no session files, one decision per turn with a reply menu on the last line.
3. If invoked directly (not from `/workflow-main`), run the silent `prompt-enhancer` ⇄ `validate-prompt-intent` loop first and present the enhanced prompt for approval.
4. Refresh `PROJECT_CONTEXT.md` for affected areas via `get-project-context: focused` if stale.
5. Pause at mandatory gates: intent, context, plan, security (IAM / network / secrets), apply (per-env), rollout (per-env), destructive.
6. Any anomaly during apply → escalate to `/workflow-incident`.

## Inputs

Use any text after `/workflow-infra` to describe change type, environments in scope, blast radius, and constraints. If empty, ask the user.

## Must Not

- Narrate bookkeeping or create session files.
- Apply directly to prod without dev + staging success.
- Ship a new alert without a linked runbook (have `doc-writer` create one first).
- Broaden IAM without running the security-gate.
