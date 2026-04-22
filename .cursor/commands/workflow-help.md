# Workflow: Help
Show the available workflow commands and when to use each. Read-only help listing. Use when you're unsure which workflow to pick.

## What This Does

Read-only help. Does not start any workflow.

## Steps

1. Emit this block verbatim:

```
UNIVERSAL ENTRY
  /workflow-main          Route any prompt. Enhances, validates, classifies, dispatches.

LIFECYCLE
  /workflow-new-project   Greenfield, zero to first release, all phases.
  /workflow-feature       Add a feature to an existing project end-to-end.
  /workflow-fix           Bug / regression fix with true root cause.
  /workflow-refactor      Behavior-preserving refactor, small green-bar steps.
  /workflow-review        Code or PR review with conditional specialist audits.
  /workflow-release       Ship an approved change-set with gated rollout.
  /workflow-migration     Large migration using expand -> migrate -> contract.

SPECIALIST / OPS
  /workflow-research      Bounded web research with citations.
  /workflow-audit         Security, performance, or accessibility audit + remediation plan.
  /workflow-incident      Live incident: stabilize, root-cause, postmortem.
  /workflow-docs          Documentation-only workflow.
  /workflow-infra         Infrastructure-only workflow with plan / security / apply gates.

CONTEXT
  /workflow-context       Build or refresh PROJECT_CONTEXT.md.

HELP
  /workflow-help          This message.
```

2. Then ask: "Which workflow fits your intent?"
3. If the user's message includes a search term (e.g. `release`, `db`, `fix`), narrow the list to matching entries.

## Inputs

Any text after `/workflow-help` may optionally be a search term.

## Must Not

- Invoke any workflow on its own.
- Modify any files.
