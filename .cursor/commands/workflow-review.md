# Workflow: Review
Code or PR review workflow. Ensures PROJECT_CONTEXT.md is fresh, runs the reviewer skill over the target diff, and conditionally pulls in specialist auditors (security, performance, accessibility) based on what the diff touches.

## What This Does

Runs `.cursor/workflows/workflow-review.md`: reviewer over the target diff → conditional specialist audits → consolidated severity-tagged findings → review-gate with next-step decision.

## Steps

1. Open `.cursor/workflows/workflow-review.md` and follow every step.
2. Obey the **Chat Rules** referenced at the top of that workflow: no narration, no session files, one decision per turn with a reply menu on the last line.
3. If invoked directly (not from `/workflow-main`), run the silent `prompt-enhancer` ⇄ `validate-prompt-intent` loop first and present the enhanced prompt for approval.
4. Confirm the target (PR URL, branch, diff range, or patch path) at the intent-gate.
5. If `PROJECT_CONTEXT.md` is stale for the changed areas, run `get-project-context: focused`.
6. Pause at mandatory gates: intent, context, review, audit (if specialist audits ran).

## Inputs

Use any text after `/workflow-review` to identify the diff (PR number, branch, patch path, or "unstaged changes"). If empty, ask the user.

## Must Not

- Narrate bookkeeping or create session files.
- Auto-merge. The user owns the final merge decision.
- Claim "no issues" without stating what rules and patterns were checked.
- Treat specialist audit findings as optional polish — they're part of the review-gate.
