# Workflow: Context
Build or refresh the persistent PROJECT_CONTEXT.md (Tier A) and optional sibling files in .cursor/context/. Supports full, refresh, or focused modes and an optional doc-writer polish pass. Use when onboarding a repo or when other workflows report stale or missing context.

## What This Does

Runs `.cursor/workflows/workflow-context.md`: choose mode (`full` / `refresh` / `focused: <subtree>`) → invoke `get-project-context` → optional `doc-writer` polish → docs-gate. Does not modify application code.

## Steps

1. Open `.cursor/workflows/workflow-context.md` and follow every step.
2. Obey the **Chat Rules** referenced at the top of that workflow: no narration, no session files, one decision per turn with a reply menu on the last line.
3. If invoked directly (not from another workflow), run the silent `prompt-enhancer` ⇄ `validate-prompt-intent` loop first and present the enhanced prompt for approval.
4. Pause at mandatory gates: intent (mode chosen), docs (only if the polish pass ran).
5. `get-project-context` writes `.cursor/context/PROJECT_CONTEXT.md` (and optional siblings `modules.md`, `glossary.md`, `dependencies.md`). This is the one file-write this workflow is expected to produce.

## Inputs

Use any text after `/workflow-context` to specify the mode (`full`, `refresh`, or `focused: <subtree>`). If empty, ask the user which mode.

## Must Not

- Narrate bookkeeping or create session files under `.cursor/context/sessions/`.
- Record any secret values. Only names and shapes.
- Delete existing ADRs or prior context files. Refresh in place or create siblings.
- Treat `PROJECT_CONTEXT.md` as disposable — it is committed Tier A.
