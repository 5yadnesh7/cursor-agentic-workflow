# Workflow: Docs
Documentation-only workflow. Ensures project context is fresh, then runs doc-writer to produce or update READMEs, architecture overviews, ADRs, runbooks, API references, onboarding guides, release notes, or CHANGELOG entries, with an optional reviewer pass and a docs-gate.

## What This Does

Runs `.cursor/workflows/workflow-docs.md`: pick doc type(s) and audience → context check → evidence gathering → draft via `doc-writer` → optional `reviewer` pass → docs-gate → refresh PROJECT_CONTEXT.md if needed.

## Steps

1. Open `.cursor/workflows/workflow-docs.md` and follow every step.
2. Obey the **Chat Rules** referenced at the top of that workflow: no narration, no session files, one decision per turn with a reply menu on the last line.
3. If invoked directly (not from `/workflow-main`), run the silent `prompt-enhancer` ⇄ `validate-prompt-intent` loop first and present the enhanced prompt for approval.
4. If `PROJECT_CONTEXT.md` is missing or stale for the target, run `get-project-context: focused` (or `refresh` if repo-wide).
5. Pause at mandatory gates: intent (doc type, audience, scope, location), context, docs (approve the draft).

## Inputs

Use any text after `/workflow-docs` to name the doc type and target surface. If empty, ask the user.

## Must Not

- Narrate bookkeeping or create session files.
- Invent facts. Every doc statement must trace to code, config, PROJECT_CONTEXT.md, or a cited source.
- Rewrite existing ADRs. They are append-only.
- Leak secrets, internal URLs, or private runbook contents into public docs.
