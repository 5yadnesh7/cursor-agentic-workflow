---
name: debugging
description: Systematically finds the root cause of bugs, failures, regressions, or unexpected behavior using evidence-driven investigation, then proposes and applies a minimal source-level fix. Reproduces the issue, isolates the failing component, reads the actual code and logs, forms and tests hypotheses, and verifies the fix does not recur. Use when the user reports a bug, a test fails, a system misbehaves, or another skill (dev-engineer, tester, devops) hits an unexplained failure.
---

# Debugging

A disciplined, evidence-first debugging skill. Never guesses a fix. Finds the true root cause, fixes it at the source, and verifies the issue cannot return silently.

## When To Use

- A user-reported bug, regression, incident, or flaky test.
- An error, stack trace, 5xx, timeout, or memory spike in production or dev.
- Unexpected output from a feature just built by `dev-engineer`.
- A `tester` run that fails for unclear reasons.
- A `devops` pipeline that fails inconsistently.

## Core Principles

1. Reproduce before you theorize.
2. Read the actual code and data, not your memory of it.
3. Form falsifiable hypotheses; verify with evidence.
4. Fix the cause, not the symptom.
5. Prefer the smallest safe change.
6. Leave behind a regression test and observability, not just a patch.

## Inputs

- `report`: user description, stack trace, logs, repro steps.
- `environment`: OS, runtime, versions, feature flags, data shape.
- `recent_changes` (optional): suspect commits, PRs, deploys, config flips.
- `constraints` (optional): blast radius, rollback window, SLAs.

## Workflow

```
Debugging:
- [ ] Step 1: Clarify the problem
- [ ] Step 2: Reproduce reliably
- [ ] Step 3: Collect evidence
- [ ] Step 4: Isolate the failing component
- [ ] Step 5: Form and test hypotheses
- [ ] Step 6: Identify the root cause
- [ ] Step 7: Propose the minimal fix
- [ ] Step 8: Apply, verify, and add regression test
- [ ] Step 9: Post-mortem summary
```

### Step 1: Clarify The Problem

Restate the issue in one sentence:

- Observed behavior.
- Expected behavior.
- Scope: who, where, when, how often.

Flag anything unknown. Ask the user only for information that changes the investigation path.

### Step 2: Reproduce Reliably

No fix without a repro, unless the blast radius demands immediate mitigation (then mitigate first and return to reproduce).

- Minimize inputs: smallest request, dataset, or UI path that triggers it.
- Pin versions and config.
- Capture exact commands, URLs, or steps.
- Record rate (always / sometimes / only under load).

If intermittent, treat concurrency, timing, ordering, caching, and shared state as prime suspects.

### Step 3: Collect Evidence

Gather before theorizing:

- Full stack traces and error codes.
- Structured logs from the failing flow.
- Metrics around the failure window.
- Recent deploys, config changes, feature-flag flips, schema changes.
- Diffs of suspect files.
- External dependency status.

Evidence goes into a running notes block. Do not mix it with guesses.

### Step 4: Isolate The Failing Component

Narrow the failure surface:

- Binary-search the call path (where does it still work? where does it first break?).
- Toggle features, caches, or dependencies to see which change makes the symptom disappear.
- Compare working vs broken inputs, environments, or commits (`git bisect` in spirit).
- Inspect actual data, not assumed data.

### Step 5: Form And Test Hypotheses

For each hypothesis, record:

```
Hypothesis: <what is broken and why>
Prediction: <if true, we should see X>
Test: <experiment or inspection>
Result: <evidence>
Verdict: <confirmed | refuted | inconclusive>
```

Stop chasing refuted hypotheses. Keep only confirmed ones.

### Step 6: Identify The Root Cause

The root cause is the earliest point where the system deviated from intended behavior. Apply the "five whys" discipline: keep asking why until the answer is a code, config, data, or design flaw, not a symptom.

Classify the cause:

- Logic error
- Off-by-one or boundary condition
- Race or concurrency issue
- State or cache inconsistency
- Contract mismatch (API, schema, types)
- Configuration or environment
- Data shape or bad input
- External dependency failure
- Performance or resource exhaustion
- Security boundary failure

### Step 7: Propose The Minimal Fix

- Change the fewest lines needed to fix the cause.
- Preserve public contracts unless the contract itself is the bug.
- Avoid fixing unrelated issues in the same change; file them separately.
- Consider whether the bug hides elsewhere with the same pattern; note siblings.

Present the proposed fix with:

- What the change does.
- Why it addresses the root cause.
- What it does not change.
- Risk and rollback plan.

### Step 8: Apply, Verify, And Add Regression Test

- Apply the fix.
- Add a deterministic regression test that fails before the fix and passes after.
- Re-run the original repro and related flows.
- Check for regressions in neighboring functionality.
- Improve observability where the bug hid (logs, metrics, assertions, types).

If a full fix is risky, land a safe mitigation behind a flag first, then the proper fix.

### Step 9: Post-Mortem Summary

```
# Bug Report: <title>

## Symptom
<what users saw>

## Root Cause
<one-paragraph explanation>

## Evidence
- <key evidence item>
- <key evidence item>

## Fix
<what changed and where>

## Regression Guard
<test or monitor added>

## Siblings / Follow-ups
- <related issues to investigate separately>

## Prevention
<process, typing, test, or observability change that would have caught this>
```

## Coordination

- Hand off schema-level causes to `db-engineer` for proper migration.
- Hand off infra or pipeline causes to `devops`.
- Ask `tester` to extend E2E coverage for the failing user flow.
- Feed repeated causes back to `brainstorm` to revisit design.

## Quality Bar

- A failing repro exists and a regression test covers it.
- The fix addresses the cause, not a symptom.
- The change is small and reviewable.
- Observability is strictly better after the fix.
- Siblings and follow-ups are named, not silently left behind.

## Anti-Patterns

- Guessing a fix and shipping it to "see if it helps".
- Wrapping the symptom in try/catch and calling it fixed.
- Rewriting large sections of code while debugging.
- Declaring "cannot reproduce" without a serious reproduction attempt.
- Deleting or disabling the failing test to make CI green.
- Blaming "flakiness" without investigating the race or timing root cause.
