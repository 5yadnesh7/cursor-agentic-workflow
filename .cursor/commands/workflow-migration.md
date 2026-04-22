# Workflow: Migration
Large cross-cutting migration using the expand -> migrate -> contract discipline. Coordinates db-engineer, data-engineer, dev-engineer, devops, tester, and release-manager across phases with checkpoint gates between each.

## What This Does

Runs `.cursor/workflows/workflow-migration.md`: strategy via the brainstorm skill's Large Migration Template → phased plan (expand → migrate → contract) → owner skills execute each phase → progressive consumer waves with a kill switch → contract-gate before removal → release and watch window.

## Steps

1. Open `.cursor/workflows/workflow-migration.md` and follow every phase.
2. Obey the **Chat Rules** referenced at the top of that workflow: no narration, no session files, one decision per turn with a reply menu on the last line.
3. If invoked directly (not from `/workflow-main`), run the silent `prompt-enhancer` ⇄ `validate-prompt-intent` loop first and present the enhanced prompt for approval.
4. Refresh `PROJECT_CONTEXT.md` for affected areas via `get-project-context: focused` if stale.
5. Pause at mandatory gates: intent, context, approach, plan, phase (between each of expand/migrate/contract), cutover (per consumer wave), contract (before removal), rollback, destructive.
6. At every phase boundary, post a compact phase-summary chat turn before asking for the phase-gate.

## Inputs

Use any text after `/workflow-migration` to state migration type, why now, deadlines, and known consumers. If empty, ask the user.

## Must Not

- Narrate bookkeeping or create session files.
- Combine Migrate and Contract in a single deployment.
- Start Migrate without a tested kill switch.
- Run Contract without verified backups and a currently-valid rollback plan.
