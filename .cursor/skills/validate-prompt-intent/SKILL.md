---
name: validate-prompt-intent
description: Validates that a user or enhanced prompt is complete, unambiguous, and actionable. Scores it against a rubric and either approves it or returns structured feedback that triggers the prompt-enhancer skill to refine it, looping until the prompt passes. Use after prompt-enhancer, before handing work to brainstorm, researcher, dev-engineer, or any downstream skill.
---

# Validate Prompt Intent

Acts as a gatekeeper between prompt enhancement and execution. Either certifies a prompt as ready or sends it back for another enhancement pass with a precise list of what to fix.

## When To Use

- **Immediately after every single `prompt-enhancer` run.** The two skills are tightly coupled: the workflow is required to call this skill every time the enhancer produces or updates an enhanced prompt — including after the user answered blocking questions, after the user replied `refine <feedback>` on a previously-approved prompt, and after any silent auto-refine.
- Before any downstream skill (`brainstorm`, `researcher`, `db-engineer`, `dev-engineer`, `debugging`, `tester`, `devops`) starts real work.
- Whenever a skill detects it cannot proceed because the intent is unclear.

This skill is the ONLY legitimate exit gate for a prompt. If you see an enhanced prompt that has not been validated, treat it as untrusted and run this skill on it first.

## Inputs

- `enhanced_prompt`: the latest draft to validate.
- `raw_prompt` (optional): original user message for drift checks.
- `iteration` (optional): current loop count.
- `max_iterations` (default 3): hard cap to avoid infinite loops.

## Validation Rubric

Evaluate each dimension as `pass`, `weak`, or `fail`.

| Dimension | Pass Criteria |
|-----------|---------------|
| Intent clarity | Single-sentence intent matches the raw prompt with no drift. |
| Intent type | One or more categories assigned. |
| Goals | Specific, testable, and finite. |
| Non-goals | Present if scope is non-trivial. |
| Inputs | Clearly enumerated where relevant. |
| Outputs | Concrete deliverables named. |
| Constraints | Performance, security, compatibility, and style covered where relevant. |
| Success criteria | Observable and verifiable. |
| Assumptions | All gap-fills are explicit. |
| Open questions | Either empty, or only non-blocking meta-items. Any question that would change Goals, Non-Goals, Inputs, Outputs, Constraints, Success Criteria, or choice of stack is **blocking** and fails this dimension. |
| Next skill | A sensible downstream skill is named. |

A prompt passes overall only when every dimension is `pass` and zero blocking open questions remain.

### Blocking vs non-blocking questions

- **Blocking** (always fails the rubric): target market/domain/segment, primary user persona, scope of v1 vs later, tech stack when the user did not say "you choose", deployment target, data sources, integration points, success metrics that drive Goals, timeline when it constrains Non-Goals.
- **Non-blocking** (allowed in Open Questions): nice-to-have polish items, optional integrations, analytics opt-ins, long-tail edge cases, choices that can be deferred past MVP without changing the plan.

If the enhancer left blocking questions in the prompt, the validator must REJECT with `verdict=ask-user` (see Step 5c below). Do **not** approve and let the workflow hand a prompt full of blocking unknowns to the user.

## Workflow

```
Prompt Validation:
- [ ] Step 1: Load enhanced prompt and iteration count
- [ ] Step 2: Check drift vs raw prompt
- [ ] Step 3: Score each rubric dimension
- [ ] Step 4: Decide verdict: approve | refine | ask-user
- [ ] Step 5a: Approve and hand off
- [ ] Step 5b: Emit structured feedback to prompt-enhancer (refine)
- [ ] Step 5c: Emit blocking-questions list to workflow (ask-user)
- [ ] Step 6: Enforce max iteration cap
```

### Step 1: Load Inputs

Read the enhanced prompt and current iteration. If `iteration` is missing, treat it as 1.

### Step 2: Drift Check

Compare the enhanced prompt's Intent section to the raw prompt. If the enhancer added scope the user did not ask for, or dropped something the user did ask for, mark `Intent clarity` as `fail`.

### Step 3: Score The Rubric

For each dimension, assign `pass`, `weak`, or `fail` with a one-line justification. Be strict: `weak` is not a pass.

### Step 4: Decide

Return exactly one of three verdicts:

- **`approve`** — all dimensions `pass` and zero blocking open questions.
- **`ask-user`** — at least one blocking open question exists (even if every other dimension is `pass`). The enhancer cannot fix this; only the user can. Emit the blocking-questions list (Step 5c).
- **`refine`** — any `weak`/`fail` on dimensions other than Open Questions, and no blocking questions remain. The enhancer can fix these on its own. Emit refine feedback (Step 5b).

If both refine-able defects AND blocking questions exist, return `ask-user` first — there is no point asking the enhancer to refine Goals/Constraints when the underlying unknowns haven't been answered yet.

### Step 5a: Approve

Emit an approval block and suggest the next skill the enhanced prompt named. The `Evidence Line` is **required** — the workflow renders it verbatim as the `**Validated** — …` header on the user-facing turn so the user can see that validation actually ran.

```
# Prompt Validation: APPROVED

## Evidence Line
rubric: all pass · blocking questions: 0 · dimensions checked: <N>

## Intent
<copied from enhanced prompt>

## Next Skill
<downstream skill>

## Notes
- <optional guidance for the downstream skill>
```

### Step 5b: Refine (enhancer can fix without user input)

Return structured feedback that the `prompt-enhancer` can consume directly.

```
# Prompt Validation: REFINE

## Iteration
<current>/<max>

## Failing Dimensions
- <dimension>: <pass|weak|fail> — <why>
- <dimension>: <pass|weak|fail> — <why>

## Required Fixes
- <specific, minimal change needed>
- <specific, minimal change needed>

## Do Not Change
- <sections that are already strong, to prevent thrash>

## Next Action
Re-run `prompt-enhancer` with this feedback.
```

### Step 5c: Ask-user (blocking questions only the user can answer)

Return a tightly-scoped list of blocking questions the workflow should surface directly to the user, plus the current working draft of the enhanced prompt. The workflow will ask the user, collect answers, and loop back into `prompt-enhancer` with the answers as additional context — not as `feedback` (answers fill gaps, they do not correct defects).

```
# Prompt Validation: ASK-USER

## Iteration
<current>/<max>

## Why Blocked
<one sentence naming the blocking dimension(s)>

## Blocking Questions
1. <plain-language question that, once answered, unblocks Goals / Non-Goals / Constraints / Inputs / Outputs / Stack>
2. <question>
3. <question>

Rules for questions:
- Maximum 5 questions per verdict. Fewer is better.
- Each question must include a sensible default in parentheses the workflow can offer as a quick-pick.
- Never repeat a question the user already answered earlier in this conversation (check the chat transcript including the raw prompt and any prior user answers).
- Never ask for information that belongs in a downstream skill (`brainstorm` handles design trade-offs; `researcher` handles external facts).

## Next Action
Workflow surfaces the questions to the user, collects answers, then re-runs `prompt-enhancer` with `user_answers` appended to the raw prompt.
```

### Step 6: Iteration Cap

If `iteration >= max_iterations` and the prompt still fails:

1. Stop the loop.
2. Surface the remaining gaps to the user in plain language.
3. Ask the user to clarify directly, then resume validation with `iteration` reset.

## Loop Contract With Prompt Enhancer (tight coupling)

- **Enhancer and validator are a single atomic unit.** Every enhancer run must be followed by a validator run before anything else happens. The workflow never presents, stores, or acts on an enhanced prompt that this skill has not approved.
- **Only `approve` exits the loop.** `refine` and `ask-user` both send the prompt back into the enhancer (directly in the case of `refine`; via the user's answers in the case of `ask-user`).
- **User-driven refines re-enter the loop too.** If the user replies `refine <feedback>` at the presentation gate, the workflow calls the enhancer with that feedback and then calls THIS skill again. The user is not authoritative over rubric defects; the workflow does not skip validation just because the user touched the prompt.
- Validator sends only minimal, targeted feedback.
- Enhancer must change only what the validator flagged or what the user's feedback/answers touched; untouched sections stay stable.
- Each validator-driven loop must improve at least one failing dimension, otherwise escalate to the user via `refine` cap or `ask-user`.

## Chat Presentation

This skill runs **silently by default**. It does not own a chat turn.

- **Do not narrate validation.** Do not write "I'll now score the enhanced prompt against the rubric…". Just produce the verdict.
- **Do not emit chat directly.** Return the verdict body (`approve` | `refine` | `ask-user` + structured feedback) to the caller workflow. The workflow decides whether the user ever sees it.
- **No file writes.** No session files, no handoffs, no logs. State is returned in-memory to the caller.
- **Never ask the user a question.** If the iteration cap is hit, emit the remaining gaps as plain-language questions inside the verdict body; the workflow surfaces them at the escalation gate.
- **One-line delta on rejection** is required in addition to the structured feedback — a compact reason the enhancer (and optionally the workflow) can quote when auto-refinement is disclosed to the user, e.g. "Goals were not testable; Success Criteria missing observability."
- **Never self-approve to end the loop faster.** `approve` is issued only when every rubric dimension passes AND there are zero `BLOCKING:` items. If in doubt between `approve` and `refine`, return `refine`.

## Quality Bar

Validation is doing its job when:

- Approved prompts are acted on without further clarification.
- Rejected prompts come back strictly better, not just different.
- The loop terminates quickly (usually 1–2 iterations).
- The user is never asked redundant questions they already answered.

## Anti-Patterns

- Rubber-stamping vague prompts to move on.
- **Approving a prompt that still has blocking Open Questions.** If the user would have to answer a question before work can start, it is a blocking question and the verdict is `ask-user`, never `approve`.
- Treating "surfaced in the `## Open Questions` section" as equivalent to "answered". A surfaced question is still unanswered.
- Rejecting prompts for stylistic nits rather than missing substance.
- Allowing the enhancer to rewrite approved sections and introduce drift.
- Running the loop forever instead of escalating to the user.
- Asking the user questions the downstream skills (`brainstorm`, `researcher`, `dev-engineer`, `db-engineer`) are supposed to decide.
