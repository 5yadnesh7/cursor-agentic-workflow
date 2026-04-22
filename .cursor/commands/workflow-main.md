# Workflow: Main (Router)
Universal entry: enhances the prompt, validates intent, classifies, and dispatches to the right workflow under `.cursor/workflows/`.

## What This Does

Runs `.cursor/workflows/workflow-main.md` end-to-end: silent prompt-enhancer ⇄ validate-prompt-intent loop (cap 3, with ask-user escalation for blocking questions) → enhanced-prompt approval → intent classification → intent-gate → dispatch to a sub-workflow (new-project, feature, fix, refactor, review, release, research, audit, incident, migration, docs, infra, or context).

## Steps

1. Open `.cursor/workflows/workflow-main.md` and follow every step exactly.
2. Obey the **Chat Rules** at the top of that workflow: no narration, no session files, one decision per turn with a reply menu on the last line.
3. Do not edit any file until the intent-gate is approved and a sub-workflow begins real work.

## Inputs

Treat any text the user typed after `/workflow-main` as the raw prompt. If the user typed only `/workflow-main` with nothing else, ask them for the task before starting.

## Must Not

- Narrate bookkeeping or "silently bootstrapping a session…".
- Create `SESSION.md`, `INDEX.md`, `prompt.md`, `ACTIVE`, `handoffs/`, `decisions.md`, `progress.md`, or anything under `.cursor/context/sessions/`.
- Skip the intent-gate.
- Dispatch to a workflow not listed in `next_workflows` of `workflow-main.md`.
