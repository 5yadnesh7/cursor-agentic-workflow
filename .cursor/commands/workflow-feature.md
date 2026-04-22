# Workflow: Feature
Add a feature to an existing project end-to-end with brainstorm, specialist skills, implementation, reviewer, tester, conditional audits, and docs.

## What This Does

Runs `.cursor/workflows/workflow-feature.md`: context check → brainstorm (approach + design gates) → specialist branches (schema / data / ML) → iterative slices with review and test gates → conditional quality audits → docs.

## Steps

1. Open `.cursor/workflows/workflow-feature.md` and follow every step.
2. Obey the **Chat Rules** referenced at the top of that workflow: no narration, no session files, one decision per turn with a reply menu on the last line.
3. If invoked directly (not from `/workflow-main`), run the silent `prompt-enhancer` ⇄ `validate-prompt-intent` loop first and present the enhanced prompt for approval.
4. If `.cursor/context/PROJECT_CONTEXT.md` is missing, stale, or thin for the target area, invoke `get-project-context` in `focused` or `refresh` mode before brainstorming.
5. Pause at every mandatory gate: intent, context, approach, design, schema (if applicable), implementation, review, test, automation (for new Playwright-style suites), docs, destructive (any).

## Inputs

Use any text after `/workflow-feature` as the feature brief. If empty, ask the user for the brief.

## Must Not

- Narrate bookkeeping or create session files.
- Implement before both the approach-gate AND the design-gate have been approved.
- Merge without the review-gate AND the test-gate.
- Skip the schema-gate when DB shapes change.
