---
name: workflow-fix
description: Bug and regression fix workflow. Reproduces the issue, uses debugging to find root cause with evidence, applies a minimal source-level fix via the correct owner skill, verifies with tests, reviews, and optionally updates runbooks. Use for reported bugs, failing tests, 5xx spikes, flaky behavior, or regressions.
skills_used:
  - prompt-enhancer
  - validate-prompt-intent
  - get-project-context
  - debugging
  - dev-engineer
  - db-engineer
  - devops
  - reviewer
  - tester
  - doc-writer
subagents_used:
  - subagent-prompt-intake
  - subagent-context-scout
  - subagent-debug-investigator
  - subagent-dev-engineer
  - subagent-db-engineer
  - subagent-devops-engineer
  - subagent-code-reviewer
  - subagent-qa-tester
  - subagent-doc-writer
gates:
  - intent-gate
  - context-gate
  - review-gate
  - test-gate
  - automation-gate
  - destructive-gate
  - docs-gate
next_workflows:
  - workflow-incident
  - workflow-release
---

# Workflow: workflow-fix

Find the true root cause, apply the minimal fix, prevent regression.

## Chat Rules

Follow the Chat Rules in `workflow-main.md`: no narration, no session files, one decision per turn with a reply menu on the last line. All state lives in the conversation.

## Step 1 — Intent Gate

If dispatched from `workflow-main` the gate already passed. If invoked directly via `/workflow-fix`, run **Step 1 of `workflow-main.md` verbatim** (the tightly-coupled `prompt-enhancer` ⇄ `validate-prompt-intent` loop: every enhancer run is immediately followed by a validator run; only `approve` exits the loop; user `refine <feedback>` re-enters at 1a; cap on validator-driven refines is 3). Then hold the **intent-gate** with the bug description and scope.

## Step 2 — Context Gate

1. Ensure `PROJECT_CONTEXT.md` covers the failing area; if not, dispatch `subagent-context-scout` in `focused` mode on that area.
2. **context-gate** (mandatory).

## Step 3 — Debugging Loop (dispatched)

Dispatch `subagent-debug-investigator` (wraps the `debugging` skill; see `.cursor/workflows/SUBAGENT_DISPATCH.md`) with:

- `report`: user description, stack trace, logs, failing test, or repro.
- `environment`: OS, runtime, versions, feature flags, data shape.
- `recent_changes`: suspect commits, PRs, deploys, config flips.
- `constraints`: blast radius, rollback window, SLAs.

The subagent runs the 9-step investigation (reproduce → collect evidence → isolate → hypothesize → root-cause → minimal fix → regression guard) and returns a single `Bug Report` block.

If users are impacted (prod outage, data loss in progress), STOP this workflow and escalate to `workflow-incident` first.

If the subagent returns `DESTRUCTIVE_GATE_REQUIRED`, hold the **destructive-gate** in chat before re-dispatching with `approved_destructive: true`.

## Step 4 — Apply The Fix (owner subagent)

`subagent-debug-investigator` applies the minimal fix itself when safe. Re-dispatch to a specialist owner only when it explicitly defers in the Bug Report:

- `subagent-dev-engineer` — application code fix beyond the debugger's minimal scope.
- `subagent-db-engineer` — schema or data fix (triggers **destructive-gate** if data is mutated).
- `subagent-devops-engineer` — infra or pipeline fix.

Regardless of owner, a regression test must be added (deterministic, failing before the fix, passing after) per the `Bug Report`'s Regression Guard section.

## Step 5 — Review And Verify

1. Dispatch `subagent-code-reviewer` on the fix diff → **review-gate** (mandatory).
2. Dispatch `subagent-qa-tester` to replay the original repro and adjacent flows. Before any new persistent automation: **automation-gate**. Then **test-gate** (mandatory).

## Step 6 — Docs (if needed)

If the bug exposed a gap:

- Dispatch `subagent-doc-writer` to update the relevant runbook.
- Add a brief postmortem note if the bug had production impact.
- **docs-gate** (mandatory if any doc was written).

## Step 7 — Sibling Sweep (optional)

`debugging` may flag sibling issues using the same pattern. Offer the user:

- `fix now` — run Step 4–5 for each sibling.
- `file only` — list them in the close-out summary for the user to track externally.

## Step 8 — Hand Off Or Release

- If the fix must ship urgently → invoke `workflow-release`.
- Otherwise hand back with a summary.

## Step 9 — Close Out

Send one final chat turn summarizing the run: root cause, fix applied (with file paths), regression test added, siblings addressed or filed, any docs updated.

## Safety Rules

- Never patch a symptom. The `debugging` root cause must be recorded before a fix is applied.
- Never skip the regression test.
- Live production impact → escalate to `workflow-incident` first.
