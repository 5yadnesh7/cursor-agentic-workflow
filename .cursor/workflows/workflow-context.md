---
name: workflow-context
description: Builds or refreshes the persistent PROJECT_CONTEXT.md (Tier A). Runs the get-project-context skill in full, refresh, or focused mode and optionally asks doc-writer to polish the output. Use when onboarding a repo, when other workflows report stale or missing context, or on a schedule.
skills_used:
  - prompt-enhancer
  - validate-prompt-intent
  - get-project-context
  - doc-writer
subagents_used:
  - subagent-prompt-intake
  - subagent-context-scout
  - subagent-doc-writer
gates:
  - intent-gate
  - docs-gate
next_workflows: []
---

# Workflow: workflow-context

Produces or updates the Tier A persistent project map. This workflow does not modify application code.

## Chat Rules

Follow the Chat Rules in `workflow-main.md`: no narration, no session files, one decision per turn with a reply menu on the last line. All state lives in the conversation. This workflow is the one exception that writes a durable file at the repo level: `.cursor/context/PROJECT_CONTEXT.md` (and optional siblings `modules.md`, `dependencies.md`, `data_layer.md`, `infra.md`, `glossary.md`).

## Step 0 — Prompt Normalization

If dispatched from `workflow-main` or another workflow, skip this step. If invoked directly via `/workflow-context`, run **Step 1 of `workflow-main.md` verbatim** (the tightly-coupled `prompt-enhancer` ⇄ `validate-prompt-intent` loop: every enhancer run is immediately followed by a validator run; only `approve` exits the loop; user `refine <feedback>` re-enters at 1a; cap on validator-driven refines is 3) against the user's stated goal for the context pass (for example: onboarding, refactor prep, audit prep, focused subtree map).

## Step 1 — Mode Selection + Intent Gate

Infer or ask:

- `full` → first scan of the repo, or major restructure.
- `refresh` → update an existing `PROJECT_CONTEXT.md`.
- `focused: <subtree or topic>` → write a sibling scoped file.

Send one chat turn:

```
Ready to run **get-project-context** in mode=<mode>.

- Impact: writes/updates .cursor/context/PROJECT_CONTEXT.md (and optional siblings)
- Risk: none outside .cursor/context/

Reply: `approve` | `modify <feedback>` | `abort`
```

## Step 2 — Run get-project-context (dispatched)

Dispatch `subagent-context-scout` (wraps the `get-project-context` skill; see `.cursor/workflows/SUBAGENT_DISPATCH.md`) with the chosen `mode`, `focus_area` (if `focused`), and default `exclude_patterns`. The subagent writes `PROJECT_CONTEXT.md` (and optionally `modules.md`, `dependencies.md`, `data_layer.md`, `infra.md`, `glossary.md`) and returns a preview summary + freshness metadata.

If the subagent returns its writes-blocked fallback (output contains `ACTION REQUIRED FOR CALLER`), the workflow persists the returned doc body to `.cursor/context/PROJECT_CONTEXT.md` itself.

## Step 3 — Polish (optional)

If the generated doc needs tightening, dispatch `subagent-doc-writer` with `doc_type=architecture` and the generated file as source. The polished draft is returned for chat preview before being written back.

### Docs Gate (mandatory if Step 3 ran)

```
PROJECT_CONTEXT.md is ready — preview above.

- Path: .cursor/context/PROJECT_CONTEXT.md
- Diff vs previous: <N lines added / M changed / K removed>
- Persistent: yes (will be committed by you)

Reply: `approve` | `modify <feedback>` | `abort`
```

## Step 4 — Close Out

Send one final chat turn with the final path, line count, and a one-paragraph summary of what the context now covers.

## Safety Rules

- Never record secret values. Only names and shapes.
- Never delete existing ADRs or prior context files; refresh in place or create sibling files.
- If `PROJECT_CONTEXT.md` is missing and another workflow is waiting on it, run this workflow in `full` mode and return control.
