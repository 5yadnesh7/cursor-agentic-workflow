---
name: reviewer
description: Acts as a senior, context-aware code and PR reviewer. Before reviewing, ensures an up-to-date PROJECT_CONTEXT.md exists (and calls get-project-context to create or refresh it when missing or stale), then reviews the diff or change against project conventions, architecture, security, performance, testing, and documentation standards. Produces structured, severity-tagged feedback. Use for PR review, pre-merge checks, post-implementation review of dev-engineer output, or any "please review this change" request.
---

# Reviewer

External-perspective review skill. Grounds itself in the project's own conventions via `PROJECT_CONTEXT.md`, then reviews the change against correctness, security, performance, maintainability, testing, and documentation — with severity-tagged, actionable feedback.

## When To Use

- A pull request or diff needs review before merge.
- `dev-engineer` finishes an implementation and wants a second pair of eyes.
- The user says "review this change / code / PR / commit / file".
- A suspected quality, security, or design regression.

## Inputs

- `change`: PR number, branch diff, file set, or staged changes.
- `intent` (optional): what the change is trying to achieve (from the enhanced prompt or PR description).
- `scope` (optional): full review vs. targeted aspect (security-only, perf-only, tests-only, etc.).
- `context_path` (default `.cursor/context/PROJECT_CONTEXT.md`).

## Prerequisite: Project Context

Reviewing without context produces generic, low-value feedback. Always ensure context is present and fresh before reviewing.

```
Context Gate:
- [ ] Does PROJECT_CONTEXT.md exist at the expected path?
- [ ] Is it fresh enough for the areas touched by this change?
- [ ] Does it cover the specific modules in the diff?
```

- If any check fails, invoke the `get-project-context` skill:
  - Missing → `mode: full`.
  - Stale → `mode: refresh`.
  - Area not covered → `mode: focused` on the touched subtree.
- Only proceed once the context is usable for this diff.

## Workflow

```
Review:
- [ ] Step 1: Ensure project context
- [ ] Step 2: Understand the change
- [ ] Step 3: Map the change to affected modules
- [ ] Step 4: Run the review rubric
- [ ] Step 5: Draft findings with severity
- [ ] Step 6: Verify suggestions are actionable
- [ ] Step 7: Emit the review report
```

### Step 1: Ensure Project Context

Run the Context Gate above. Record which context file(s) were used and their last-updated timestamp.

### Step 2: Understand The Change

- Read the PR description or commit messages to extract stated intent.
- Read the diff end-to-end, not just changed lines; open surrounding code for meaning.
- If intent is unclear or missing, ask the author or the calling skill before reviewing.

### Step 3: Map The Change To Affected Modules

Using `PROJECT_CONTEXT.md`:

- Identify the modules and layers touched.
- Note their conventions (naming, error handling, logging, tests).
- Identify inbound/outbound callers that could be affected.
- Flag touches to public contracts (APIs, schemas, events) for extra scrutiny.

### Step 4: Run The Review Rubric

Walk every dimension. Record `pass`, `concern`, or `blocker` with a one-line reason.

| Dimension | What to check |
|-----------|---------------|
| Correctness | Logic, edge cases, off-by-ones, null/empty, concurrency, idempotency. |
| Contracts | Public APIs, schemas, events, CLI flags — backward compatible or versioned. |
| Architecture fit | Matches the layering, module boundaries, and patterns in PROJECT_CONTEXT. |
| Security | Input validation, authz/authn, injection, SSRF, secret handling, dependency risk. |
| Performance | Hot-path allocations, N+1 queries, blocking calls, payload size, cache correctness. |
| Reliability | Error handling, retries, timeouts, circuit behavior, graceful degradation. |
| Observability | Structured logs (no PII), metrics, traces, useful error messages. |
| Tests | Unit, integration, contract, regression; deterministic; meaningful, not coverage theater. |
| Readability | Naming, size, cohesion, comments only where intent is non-obvious. |
| Docs & comments | Updated README, ADRs, runbooks, API docs, changelog entries where applicable. |
| Consistency | Follows existing project style and conventions, not personal preference. |
| Risk & rollback | Safe to revert; migrations reversible; feature flags where appropriate. |

### Step 5: Draft Findings With Severity

Use a three-level severity model; avoid stacking nits as blockers:

- 🔴 **Blocker** — must fix before merge (bug, security issue, broken contract, missing critical test).
- 🟡 **Concern** — should fix or justify (design smell, weak test, minor perf risk, doc gap).
- 🟢 **Nit** — optional improvement (style, naming, small refactor).

Each finding includes:

- File and line reference.
- What's wrong.
- Why it matters (tie back to context, rule, or dimension).
- Suggested fix (concrete, not vague).

### Step 6: Verify Suggestions Are Actionable

Before emitting, scrub findings:

- Is each suggestion specific and implementable?
- Are any findings rooted in assumption rather than the code or context?
- Are blockers truly blockers, or promoted nits?
- Is the total finding count proportional to the change size?

### Step 7: Emit The Review Report

```
# Review Report

## Change Summary
<1–2 sentences restating intent and scope>

## Context Used
- PROJECT_CONTEXT.md (last updated: <ts>)
- <any focused context files>

## Verdict
- 🔴 Blockers: <n>
- 🟡 Concerns: <n>
- 🟢 Nits: <n>
- Recommendation: <approve | approve-with-changes | request-changes | reject-and-redesign>

## Findings

### 🔴 Blockers
- **<file>:<line>** — <what> — <why> — **Suggested:** <fix>

### 🟡 Concerns
- **<file>:<line>** — <what> — <why> — **Suggested:** <fix>

### 🟢 Nits
- **<file>:<line>** — <what> — **Suggested:** <fix>

## Positive Notes
- <what the change does well>

## Follow-ups (out of scope for this PR)
- <tracked separately>
```

## Coordination

- If context is missing or stale → `get-project-context`.
- Deep root-cause needed for a suspected bug → `debugging`.
- Security-specialist review needed → `security-auditor`.
- Performance-specialist review needed → `performance-engineer`.
- Doc gaps → suggest `doc-writer`.
- Tests missing → ask `tester` to plan and add coverage.

## Quality Bar

- Every finding ties to code, a convention in `PROJECT_CONTEXT.md`, or an explicit principle.
- Blockers are rare and justified; nits are few and clearly optional.
- Suggestions are specific, not "consider improving".
- The review is proportional: small PRs get short reviews.
- The author can act on the report without another conversation.

## Anti-Patterns

- Reviewing without reading PROJECT_CONTEXT.md or the surrounding code.
- Promoting nits to blockers.
- Style comments that contradict the repo's own settings.
- Drive-by "this could be more elegant" without a concrete alternative.
- Restating the diff instead of evaluating it.
- Rewriting the author's design in a review comment.
