---
name: workflow-main
description: Universal entry workflow. Enhances and validates the raw prompt, asks only the blocking questions the user must answer, classifies intent, holds the intent-gate, and dispatches to the correct sub-workflow.
skills_used:
  - prompt-enhancer
  - validate-prompt-intent
subagents_used:
  - subagent-prompt-intake
  - subagent-super
gates:
  - intent-gate
next_workflows:
  - workflow-new-project
  - workflow-feature
  - workflow-fix
  - workflow-refactor
  - workflow-review
  - workflow-release
  - workflow-research
  - workflow-audit
  - workflow-incident
  - workflow-migration
  - workflow-docs
  - workflow-infra
  - workflow-context
---

# Workflow: workflow-main

Universal router. Every prompt should enter here unless the user explicitly invokes another workflow command.

## Zero-config escape hatch: `subagent-super`

If the user asks for help with a vague request ("just handle it", "help me", "do the right thing") OR if you are invoked from outside a known workflow context and cannot confidently classify intent, you MAY short-circuit Steps 1-3 by dispatching `subagent-super` with `request = raw user message`. The super subagent runs its own intake, classifies, and returns one of:

- `DONE` — work is complete; surface the result to the user and stop.
- `HANDOFF` — super picked a workflow; treat its returned `Enhanced Prompt` as if Step 1 approved it, skip to Step 3 (Intent Gate) with the workflow it named, and proceed.
- `ASK-USER` / `ESCALATE` — surface its blocking questions or gaps to the user using the 1c / 1d templates below.
- `BLOCKED` / `DESTRUCTIVE_GATE_RELAY` — surface to the user as a one-line escalation; do not proceed without approval.

Prefer the full Steps 1-4 path when you can confidently classify intent yourself. `subagent-super` is the fallback, not the default.

## Chat Rules (apply to every step)

1. **Never narrate bookkeeping or internal reasoning.** Don't say "I'll silently bootstrap…", "Planning next moves…", "Let me first…". Just produce the next user-facing artifact.
2. **No session files. No SESSION.md, INDEX.md, ACTIVE, prompt.md, handoffs/, decisions.md, progress.md.** Keep state in the conversation.
3. **One decision per turn.** Each chat turn surfaces exactly one artifact and ends with a reply menu on the last line (e.g. `Reply: approve | refine <what> | abort`).
4. **First user-facing turn = first decision.** Do not precede the first decision with a progress update, greeting, or announcement.
5. **No file edits until the user has approved the intent-gate** and a sub-workflow begins real work. Up to that point, everything lives in chat.

## Step 1 — Enhance ⇄ Validate Loop (dispatched to `subagent-prompt-intake`)

This step delegates the tightly-coupled `prompt-enhancer` ⇄ `validate-prompt-intent` loop to the `subagent-prompt-intake` subagent (see `.cursor/workflows/SUBAGENT_DISPATCH.md`). The subagent runs the loop silently (cap 3 rounds) and returns exactly one of three verdicts:

- `APPROVED` — the workflow proceeds to 1e (presentation).
- `ASK-USER` — the workflow proceeds to 1c (blocking questions).
- `ESCALATE` — the iteration cap hit; the workflow proceeds to 1d.

The user-facing chat templates (1c, 1d, 1e) below are unchanged — the workflow still owns every user turn. What moved into the subagent is ONLY the silent enhancer⇄validator bookkeeping, not the presentation.

You may not present, store, classify, or act on an enhanced prompt that did not come back as `APPROVED` from `subagent-prompt-intake`.

### Hard rules (enforced by the subagent)

1. **Pair rule**: `subagent-prompt-intake` runs `prompt-enhancer` → `validate-prompt-intent` as a single atomic unit. There is no "the prompt is clearly fine, skip validation" path.
2. **Exit rule**: the ONLY verdict that produces a user presentation (1e) is `APPROVED` with zero `BLOCKING:` items.
3. **User-input rule**: whenever the user replies inside this step (at 1c or 1e), the workflow re-dispatches to `subagent-prompt-intake` with the new `user_answers` or `user_feedback` before any other action.
4. **Evidence rule**: the 1e presentation turn MUST include a visible validator-evidence line so the user can see validation happened. The line is populated from the subagent's returned `Iterations Used` and rubric state.

### Procedure (dispatch-based)

```
# Initialization (runs ONCE per entry into Step 1)
state = {
  raw_prompt: <user message>,
  user_answers: [],
  user_feedback: "",
}

# Main loop
loop:
  # ---- 1a+1b. DISPATCH: silent enhance+validate loop ----
  # The subagent runs prompt-enhancer ⇄ validate-prompt-intent up to 3 rounds,
  # then returns one of APPROVED / ASK-USER / ESCALATE.
  result = Task(
    subagent_type: "subagent-prompt-intake",
    inputs: {
      raw_prompt: state.raw_prompt,
      user_answers: state.user_answers,
      user_feedback: state.user_feedback,
    },
  )

  # ---- Branch on returned verdict ----
  if result.kind == "APPROVED":
    goto 1e_present

  if result.kind == "ASK-USER":
    goto 1c_ask_blocking

  if result.kind == "ESCALATE":
    goto 1d_escalation

# ---- 1c. CHAT: ask blocking questions ----
1c_ask_blocking:
  send_chat(blocking_questions_template(result.blocking_questions))
  answer = await_user_reply()
  if answer == "abort": stop
  if answer == "use defaults":
    state.user_answers += result.blocking_defaults
  else:
    state.user_answers += parse_answers(answer)
  state.user_feedback = ""
  continue loop   # re-dispatch subagent with updated user_answers

# ---- 1d. CHAT: escalation (iteration cap hit inside subagent) ----
1d_escalation:
  send_chat(escalation_template(result.remaining_gaps))
  answer = await_user_reply()
  if answer == "abort": stop
  state.user_answers += parse_answers(answer)
  state.user_feedback = ""
  continue loop   # re-dispatch subagent; iteration count resets

# ---- 1e. CHAT: present enhanced prompt (the ONLY exit) ----
1e_present:
  # Pre-presentation checklist — if any is false, GOTO continue loop
  assert result.kind == "APPROVED"
  assert not has_blocking_items(result.enhanced_prompt)

  send_chat(presentation_template(result.enhanced_prompt, result.iterations_used, result.delta))
  reply = await_user_reply()
  if reply == "approve":
    return result.enhanced_prompt   # EXIT to Step 2
  if reply.startswith("refine "):
    state.user_feedback = reply.feedback
    continue loop   # re-dispatch subagent; it runs validator after every re-enhance
  if reply == "abort":
    stop
```

### 1c chat template (validator verdict = ask-user)

Send one chat turn. **Do not include the full enhanced prompt here** — the user is filling gaps, not reviewing the whole artifact.

```
Before I write up the plan, I need a few answers. Defaults in parentheses; reply with picks or your own values.

- <question 1> (default: <default>)
- <question 2> (default: <default>)
- <question 3> (default: <default>)

Reply: answer each | `use defaults` | `abort`
```

Rules:

- Max 5 questions per turn.
- Every question has a sensible default in parentheses.
- Never re-ask a question already in `state.user_answers` or already answered in the chat transcript.

### 1d chat template (validator-refine cap hit)

```
I've tightened the prompt three times and these gaps still remain. Could you answer briefly so I can proceed?

- <plain-language question from latest validator feedback>
- <plain-language question>
- <plain-language question>

Reply with the answers, or `abort`.
```

### 1e chat template (subagent returned APPROVED — the only exit gate)

Send exactly one chat turn, no preamble. The **Validated** line at the top is mandatory — it is the visible evidence that `subagent-prompt-intake` actually ran the validator.

```
**Validated** — rubric: all pass · blocking questions: 0 · iterations: <result.iterations_used>/3

Here's how I read your request. Review and reply.

# Enhanced Prompt

## Intent
<one-sentence intent>

## Goals
- <testable goal>
- <testable goal>

## Non-Goals
- <out-of-scope item>

## Inputs
- <data / trigger>

## Outputs
- <deliverable>

## Constraints
- <constraint>

## Success Criteria
- <observable condition>

## Assumptions
- <assumption>

## Open Questions (optional, non-blocking)
- <polish item the user can ignore>

---

Reply: `approve` | `refine <what to change>` | `abort`
```

Presentation rules:

- The `**Validated** — ...` line is REQUIRED on every 1e turn. It proves the subagent ran the validator. If you are about to send a 1e turn without it, stop — you skipped the dispatch.
- If `result.iterations_used > 1`, optionally add a second line: `Auto-tightened: <result.delta>` so the user sees what changed and why.

### User reply behaviour at 1e

- `approve` → exit Step 1. Carry `result.enhanced_prompt` forward to Step 2.
- `refine <feedback>` → set `state.user_feedback = feedback`, **do not** reset `user_answers`, re-dispatch `subagent-prompt-intake`. The next user-facing turn is again 1e — but only after the subagent returns `APPROVED`. If the subagent returns `ASK-USER`, detour through 1c first.
- `abort` → one-line acknowledgement, stop.

### Self-check before every user-facing turn in Step 1

Before you type a single character to the user inside Step 1, silently answer:

1. Did `subagent-prompt-intake` run as the most recent non-chat action? (yes/no)
2. For 1e only: was `result.kind == "APPROVED"` with zero `BLOCKING:` items? (yes/no)

## Step 2 — Classify Intent (silent)

Using the approved enhanced prompt, classify and pick one best-matching workflow:

| Signal in enhanced prompt | Route to |
|---|---|
| Greenfield, new project, from scratch, empty repo | `workflow-new-project` |
| Bug, regression, error, 500, flaky, broken | `workflow-fix` |
| Refactor, cleanup, rename, restructure (no behavior change) | `workflow-refactor` |
| Review, PR, diff, code review | `workflow-review` |
| Release, version, publish, ship, tag | `workflow-release` |
| Research, compare, investigate, find docs | `workflow-research` |
| Security / performance / accessibility audit | `workflow-audit` (with mode) |
| Incident, outage, on fire, users impacted | `workflow-incident` |
| Migration, upgrade, cutover, rewrite | `workflow-migration` |
| Docs, README, ADR, runbook, onboarding | `workflow-docs` |
| Infra, IaC, Terraform, K8s, pipeline | `workflow-infra` |
| Map repo, what is this project, build context | `workflow-context` |
| Anything else involving new feature code | `workflow-feature` |

Do not message the user yet. Carry the chosen workflow into Step 3.

## Step 3 — Intent Gate (chat, mandatory)

Send one chat turn:

```
Routing this to **<workflow-name>** — <one-sentence reason tying the enhanced prompt to the chosen workflow>.

- Signals that drove the choice: <keyword1>, <keyword2>, <keyword3>
- Key assumptions carried forward: <bullet or "none">
- What this workflow will do first: <short sentence>

Reply: `approve` | `modify <feedback>` | `abort` | `use workflow-<name>`
```

On reply:

- `approve` → proceed to Step 4.
- `modify <feedback>` → loop back to Step 1 (start at 1a) with the feedback.
- `abort` → one-line acknowledgement, stop.
- `use workflow-<name>` → proceed to Step 4 using the user-specified workflow. If the name is not in `next_workflows`, reject and re-present the gate.

## Step 4 — Dispatch

Send one short chat line (nothing more), e.g. `Starting **workflow-new-project**.`, then invoke the chosen workflow file at `.cursor/workflows/<workflow-name>.md`. The sub-workflow inherits the approved enhanced prompt from chat and owns all subsequent turns.

## Safety Rules

- Never skip the intent-gate.
- Never dispatch to a workflow that is not in `next_workflows`.
- Never write files under `.cursor/context/sessions/` — that system is removed.
- `.cursor/context/PROJECT_CONTEXT.md` is owned by `get-project-context` and only written inside workflows that explicitly need it (`workflow-context`, `workflow-new-project` closeout, and on-demand refreshes from other workflows).
- On any prod-impacting anomaly mid-dispatch, escalate to `workflow-incident`.
