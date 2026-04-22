# Workflow: Release
Ship an approved change-set to users. Runs release-manager to freeze scope, cut version, produce release notes and a rollback plan, and drive progressive rollout with devops, with tester smokes and incident-commander on standby.

## What This Does

Runs `.cursor/workflows/workflow-release.md`: pre-release checks → release manifest + notes → build & sign → progressive rollout (dev → staging → canary → prod) with a rollout-gate at each step → post-release close-out → watch window.

## Steps

1. Open `.cursor/workflows/workflow-release.md` and follow every step.
2. Obey the **Chat Rules** referenced at the top of that workflow: no narration, no session files, one decision per turn with a reply menu on the last line.
3. If invoked directly (not from `/workflow-main`), run the silent `prompt-enhancer` ⇄ `validate-prompt-intent` loop first and present the enhanced prompt for approval.
4. Confirm upstream gates passed (review-gate, test-gate, audits where applicable). If anything is missing, pause and route to the correct workflow before continuing.
5. Pause at mandatory gates: intent, docs (release notes), release (full manifest), destructive (any destructive DB change), rollout (before EACH environment).
6. If SLOs breach during rollout → escalate to `/workflow-incident`, return here after resolution.

## Inputs

Use any text after `/workflow-release` to identify the release scope (branch, tag, or commit range) and type (`patch | minor | major | hotfix`). If empty, ask the user.

## Must Not

- Narrate bookkeeping or create session files.
- Advance environments without an explicit rollout-gate.
- Ship a destructive DB change without destructive-gate plus a tested rollback plan.
- Reach prod without a green staging smoke from the `tester` skill.
