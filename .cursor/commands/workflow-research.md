# Workflow: Research
Bounded web research with citations. Runs the researcher skill in a capped loop and produces a summary, recommendation, decision memo, or ADR draft. Use for "compare X vs Y", dependency or stack choices, or as an excursion when another workflow hits an unknown.

## What This Does

Runs `.cursor/workflows/workflow-research.md`: bounded research loop (cap 3 iterations) → synthesis with confidence and citations → research-gate → typed deliverable (summary / recommendation / memo / ADR draft) → optional hand-off to a downstream workflow.

## Steps

1. Open `.cursor/workflows/workflow-research.md` and follow every step.
2. Obey the **Chat Rules** referenced at the top of that workflow: no narration, no session files, one decision per turn with a reply menu on the last line.
3. If invoked directly (not from another workflow), run the silent `prompt-enhancer` ⇄ `validate-prompt-intent` loop first and present the enhanced prompt for approval.
4. Pause at mandatory gates: intent (precise questions, depth, deliverable), research (verdict on findings).
5. If invoked from another workflow, return the synthesis to the caller instead of closing.

## Inputs

Use any text after `/workflow-research` to describe the question(s). If empty, ask the user for precise questions and desired deliverable type.

## Must Not

- Narrate bookkeeping or create session files.
- Loop past the cap. Surface unresolved items as "open questions" instead.
- Make non-trivial claims without citations.
- Rely on stale or non-primary sources without flagging them.
