---
name: workflow-research
description: Standalone or escape-hatch research workflow. Runs the researcher skill in a bounded loop, produces an evidence-backed summary with citations, and optionally hands findings into another workflow. Use for "compare X vs Y", "how do people solve Z", dependency choice, or when another workflow hits an unknown.
skills_used:
  - prompt-enhancer
  - validate-prompt-intent
  - researcher
  - doc-writer
subagents_used:
  - subagent-prompt-intake
  - subagent-researcher
  - subagent-doc-writer
gates:
  - intent-gate
  - research-gate
next_workflows:
  - workflow-feature
  - workflow-infra
  - workflow-migration
  - workflow-audit
---

# Workflow: workflow-research

Bounded research with receipts.

## Chat Rules

Follow the Chat Rules in `workflow-main.md`: no narration, no session files, one decision per turn with a reply menu on the last line. All state lives in the conversation.

## Step 1 — Intent Gate

If dispatched from `workflow-main` or another workflow, the gate already passed. If invoked directly via `/workflow-research`, run **Step 1 of `workflow-main.md` verbatim** (the tightly-coupled `prompt-enhancer` ⇄ `validate-prompt-intent` loop: every enhancer run is immediately followed by a validator run; only `approve` exits the loop; user `refine <feedback>` re-enters at 1a; cap on validator-driven refines is 3).

Capture:

- The question(s) to answer (precise, testable).
- Depth: `quick | standard | deep`.
- Deliverable: `summary | recommendation | decision memo | ADR draft`.
- Scope limits: must-have / out-of-scope.

**intent-gate** (mandatory).

## Step 2 — Research Loop (dispatched)

Dispatch `subagent-researcher` (wraps the `researcher` skill; see `.cursor/workflows/SUBAGENT_DISPATCH.md`) with:

- `research_question`: from the approved prompt.
- `depth`: from the intent gate (`quick` | `standard` | `deep`).
- `constraints`: version pins, license, allowed sources.
- `known_sources`: docs the user already trusts.

The subagent runs the 8-step research loop internally (plan → search → extract → self-critique → iterate, capped by depth) and returns a single consolidated `Research` report with inline citations and a `Loop Metadata` block showing rubric state.

If the subagent returns an `Open Gaps` section at the depth cap, surface those gaps at the research-gate rather than silently approving.

## Step 3 — Synthesis

The report returned by `subagent-researcher` already includes:

- Answer(s) to the sub-questions.
- Recommendation (if requested) with trade-offs.
- Open questions / unknowns.
- Confidence level per claim.
- Citations inline.

## Step 4 — Research Gate

**research-gate** (mandatory). User options:

- `approve` — finalize deliverable.
- `modify <feedback>` — re-enter the loop.
- `abort` — archive session.
- `deepen <subtopic>` — targeted follow-up loop.

## Step 5 — Deliverable

Based on deliverable type:

- `summary` — the synthesis itself is the deliverable; render it in chat.
- `recommendation` — add a "Chosen Option + Reasons" section to the synthesis.
- `decision memo` → dispatch `subagent-doc-writer` with `doc_type=explainer` to produce a memo at a user-chosen repo path (ask at the gate).
- `ADR draft` → dispatch `subagent-doc-writer` with `doc_type=adr` at a user-chosen repo path (typically `docs/adr/` or the project's ADR convention).

## Step 6 — Hand Off

Suggest the next workflow if the research answers a choice blocking real work:

- Stack / framework / library choice → `workflow-feature` or `workflow-new-project`.
- Cloud / infra / CI / observability choice → `workflow-infra`.
- Engine / migration strategy → `workflow-migration`.
- Security / performance / a11y deep-dive → `workflow-audit`.

## Step 7 — Close Out

Send one final chat turn with the synthesis, citations, and suggested next workflow. If invoked from another workflow, hand control back to the caller with the synthesis instead of closing.

## Safety Rules

- Every non-trivial claim must have a source. No dressed-up guesses.
- Stale sources must be flagged; prefer primary docs.
- Loop cap is firm — if still unresolved, surface as "open questions" instead of looping forever.
