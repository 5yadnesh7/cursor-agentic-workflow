---
name: workflow-review
description: Standalone code or PR review workflow. Ensures PROJECT_CONTEXT.md is fresh, runs the reviewer skill over the target diff, and optionally pulls in specialist auditors (security, performance, accessibility) based on what changed. Use when the user asks for a PR review, a diff review, or a pre-merge sanity check without necessarily shipping.
skills_used:
  - prompt-enhancer
  - validate-prompt-intent
  - get-project-context
  - reviewer
  - security-auditor
  - performance-engineer
  - accessibility-auditor
  - doc-writer
subagents_used:
  - subagent-prompt-intake
  - subagent-context-scout
  - subagent-code-reviewer
  - subagent-security-auditor
  - subagent-performance-auditor
  - subagent-accessibility-auditor
  - subagent-doc-writer
gates:
  - intent-gate
  - context-gate
  - review-gate
  - audit-gate
next_workflows:
  - workflow-fix
  - workflow-feature
---

# Workflow: workflow-review

Pre-merge quality gate.

## Chat Rules

Follow the Chat Rules in `workflow-main.md`: no narration, no session files, one decision per turn with a reply menu on the last line. All state lives in the conversation.

## Step 1 — Intent Gate

If dispatched from `workflow-main` the gate already passed. If invoked directly via `/workflow-review`, run **Step 1 of `workflow-main.md` verbatim** (the tightly-coupled `prompt-enhancer` ⇄ `validate-prompt-intent` loop: every enhancer run is immediately followed by a validator run; only `approve` exits the loop; user `refine <feedback>` re-enters at 1a; cap on validator-driven refines is 3).

Capture the target:

- A PR number / URL, or
- A branch / diff range, or
- A patch file.

**intent-gate** (mandatory): confirm scope and severity policy (blockers / majors / minors).

## Step 2 — Context Gate

Dispatch `subagent-code-reviewer` with the diff first. If it returns `CONTEXT_REQUIRED`, dispatch `subagent-context-scout` with the requested mode+area, then re-dispatch `subagent-code-reviewer`. **context-gate** passes when `subagent-code-reviewer` returns a `REVIEW_REPORT` block.

## Step 3 — Reviewer Pass (dispatched)

`subagent-code-reviewer` (wraps the `reviewer` skill; see `.cursor/workflows/SUBAGENT_DISPATCH.md`) loads the diff, applies repo rules (from `.cursor/rules/`), runs the 12-dimension rubric, and returns severity-tagged feedback (🔴 Blocker / 🟡 Concern / 🟢 Nit) in a `REVIEW_REPORT` block.

## Step 4 — Specialist Audits (conditional, parallelizable)

Based on diff analysis, dispatch in a single turn (the Task tool runs them concurrently):

- Auth / session / input / output / secrets / PII → `subagent-security-auditor` → **audit-gate** on remediations.
- Hot path / N+1 risk / large loops / DB-heavy → `subagent-performance-auditor` → **audit-gate**.
- UI / component / interaction → `subagent-accessibility-auditor` → **audit-gate**.

Each auditor may itself return `CONTEXT_REQUIRED`; handle by routing to `subagent-context-scout` and re-dispatching.

## Step 5 — Consolidate Findings

1. The workflow merges the `REVIEW_REPORT` and any auditor reports into a single chat summary.
2. Classify each finding with severity and suggested owner subagent (`subagent-dev-engineer` / `subagent-db-engineer` / `subagent-devops-engineer` / `subagent-doc-writer`).

## Step 6 — Review Gate

**review-gate** (mandatory). The user chooses:

- `approve` — reviewer writes a merge-ready comment/summary.
- `fix now` — dispatch to `workflow-fix` or `workflow-feature`.
- `reject` — reviewer drafts the decline comment.

## Step 7 — Docs Sync (optional)

If the diff changes public behavior that isn't documented, dispatch `subagent-doc-writer` and add the gap to follow-ups.

## Step 8 — Close Out

Send one final chat turn summarizing the review: blockers/majors/minors count, specialist audits run, user decision (approve / fix now / reject), and the merge-ready or decline comment text.

## Safety Rules

- No auto-merge. The user owns the final merge decision.
- Never claim "no issues" without citing which rules and patterns were checked.
- When a specialist audit is triggered, its findings are part of the review gate, not optional polish.
