# Workflow: Fix
Bug or regression fix workflow. Reproduces the issue, uses the debugging skill to find root cause with evidence, applies a minimal source-level fix via the correct owner skill, adds a regression test, and reviews.

## What This Does

Runs `.cursor/workflows/workflow-fix.md`: reproduce → debugging (root cause + evidence) → minimal fix by the owner skill → regression test → reviewer + tester gates → optional docs / runbook update.

## Steps

1. If the issue is impacting production right now, run `/workflow-incident` first; return here after resolution.
2. Open `.cursor/workflows/workflow-fix.md` and follow every step.
3. Obey the **Chat Rules** referenced at the top of that workflow: no narration, no session files, one decision per turn with a reply menu on the last line.
4. If invoked directly (not from `/workflow-main`), run the silent `prompt-enhancer` ⇄ `validate-prompt-intent` loop first and present the enhanced prompt for approval.
5. Refresh `PROJECT_CONTEXT.md` for the failing area via `get-project-context: focused` if stale.
6. Pause at mandatory gates: intent, context, review, test, automation (if new automation is written), destructive (any data change), docs (if a runbook was updated).

## Inputs

Use any text after `/workflow-fix` as the bug description. If empty, ask the user.

## Must Not

- Narrate bookkeeping or create session files.
- Patch a symptom. Root cause must be recorded by the `debugging` skill before a fix is applied.
- Skip the regression test.
