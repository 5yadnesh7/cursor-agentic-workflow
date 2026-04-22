# Workflow: New Project
Greenfield end-to-end, zero to first release. Covers prompt, research, brainstorm, context seeding, DB and infra foundations, scaffolding, iterative feature build, quality audits, documentation, and release, with an approval gate at every phase.

## What This Does

Runs `.cursor/workflows/workflow-new-project.md` end-to-end: Phase 0 (intent) through Phase 8 (close-out) — discovery, foundations, scaffolding, iterative build, quality bar, documentation, first release, and close-out.

## Steps

1. Open `.cursor/workflows/workflow-new-project.md` and follow every phase in order.
2. Obey the **Chat Rules** referenced at the top of that workflow: no narration, no session files, one decision per turn with a reply menu on the last line.
3. If invoked directly (not from `/workflow-main`), run the silent `prompt-enhancer` ⇄ `validate-prompt-intent` loop first and present the enhanced prompt for approval.
4. Pause at every mandatory gate: intent, research, approach, design, schema, infra-plan, implementation, review, test, automation, security, docs, release, rollout, destructive.
5. At every phase boundary, post a compact phase-summary chat turn before asking for the next phase's gate.

## Inputs

Use any text after `/workflow-new-project` as the greenfield brief (audience, goal, constraints). If empty, ask the user for the brief before starting.

## Must Not

- Narrate bookkeeping or create session files.
- Skip any mandatory gate — especially security-gate, release-gate, rollout-gate, destructive-gate.
- Run prod rollout without a green staging smoke from the `tester` skill.
