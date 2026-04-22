---
name: workflow-docs
description: Documentation-only workflow. Ensures project context is fresh, runs doc-writer to produce or update READMEs, architecture overviews, ADRs, runbooks, API references, onboarding guides, or CHANGELOG entries, and finishes with a docs review. Use when the user asks for docs without code changes.
skills_used:
  - prompt-enhancer
  - validate-prompt-intent
  - get-project-context
  - doc-writer
  - reviewer
  - researcher
gates:
  - intent-gate
  - context-gate
  - docs-gate
next_workflows: []
---

# Workflow: workflow-docs

Produce documentation that stays accurate.

## Chat Rules

Follow the Chat Rules in `workflow-main.md`: no narration, no session files, one decision per turn with a reply menu on the last line. All state lives in the conversation.

## Step 1 — Intent Gate

If dispatched from `workflow-main` the gate already passed. If invoked directly via `/workflow-docs`, run **Step 1 of `workflow-main.md` verbatim** (the tightly-coupled `prompt-enhancer` ⇄ `validate-prompt-intent` loop: every enhancer run is immediately followed by a validator run; only `approve` exits the loop; user `refine <feedback>` re-enters at 1a; cap on validator-driven refines is 3).

Capture:

- Doc type(s): `readme | architecture | adr | runbook | api-reference | onboarding | changelog | postmortem | release-notes | user-guide`.
- Target audience.
- Scope (files / subsystems / endpoints).
- Deliverable location.

**intent-gate** (mandatory).

## Step 2 — Context Gate

Ensure `PROJECT_CONTEXT.md` covers the target surface. If not → `get-project-context: focused` (or `refresh` if whole repo).

**context-gate** (mandatory).

## Step 3 — Evidence Gathering

Depending on doc type, `doc-writer` pulls from:

- Source code, schemas, configs.
- Existing docs (avoid duplication; update in place when possible).
- Recent chat decisions (for release notes / ADRs tied to a recent call).
- `researcher` output if external references are needed.

## Step 4 — Draft

`doc-writer` produces the draft under the appropriate path:

- `README.md` at root.
- `docs/architecture.md`.
- `docs/adr/ADR-<nn>-<slug>.md` (or the project's ADR convention).
- `docs/runbooks/<name>.md`.
- `docs/api/<area>.md`.
- `docs/onboarding.md`.
- `CHANGELOG.md` update.
- Release notes: `docs/releases/<version>.md`.

Present the draft in chat for the user to preview before writing to the final path.

## Step 5 — Reviewer Pass

`reviewer` (optional but recommended for public-facing docs):

- Accuracy vs code and PROJECT_CONTEXT.
- Structure, voice, and consistency.
- Links, code samples, and examples.

## Step 6 — Docs Gate

**docs-gate** (mandatory). User options:

- `approve` — doc-writer writes to the final path.
- `modify <feedback>` — revise and re-draft.
- `split` — break into multiple docs.
- `abort`.

## Step 7 — Context Refresh (if applicable)

If the new doc changes what `PROJECT_CONTEXT.md` should say, run `get-project-context: refresh`.

## Step 8 — Close Out

Send one final chat turn listing the doc type(s) produced or updated and the final repo path(s).

## Safety Rules

- No docs without context — never invent facts.
- ADRs are additive and append-only; never rewrite history.
- Public docs must not leak secrets, internal URLs, or private runbook contents.
- Prefer updating an existing doc over creating a new one.
