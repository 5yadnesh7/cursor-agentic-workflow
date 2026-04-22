# Workflow: Refactor
Behavior-preserving refactor. Locks current behavior with characterization tests, then runs a reviewer-designed plan in small green-bar steps. Use for renames, restructuring, module extraction, dependency upgrades without behavior change, and cleanups.

## What This Does

Runs `.cursor/workflows/workflow-refactor.md`: lock behavior with characterization tests → reviewer-designed step plan → execute small steps while suite stays green → performance sanity → final review.

## Steps

1. If the scope implies any behavior change → redirect to `/workflow-feature`. If it's a framework or engine migration → redirect to `/workflow-migration`.
2. Open `.cursor/workflows/workflow-refactor.md` and follow every step.
3. Obey the **Chat Rules** referenced at the top of that workflow: no narration, no session files, one decision per turn with a reply menu on the last line.
4. If invoked directly (not from `/workflow-main`), run the silent `prompt-enhancer` ⇄ `validate-prompt-intent` loop first and present the enhanced prompt for approval.
5. If `PROJECT_CONTEXT.md` is stale for the target surface, run `get-project-context: focused`.
6. Pause at mandatory gates: intent, context, plan, review (every 3–5 steps or at any risky step), test, destructive.

## Inputs

Use any text after `/workflow-refactor` as the refactor brief. If empty, ask the user.

## Must Not

- Narrate bookkeeping or create session files.
- Change public API, schema shape, or observable behavior in this workflow.
- Let the suite go red between steps. Revert and replan if it does.
- Take "quick wins" outside the approved plan.
