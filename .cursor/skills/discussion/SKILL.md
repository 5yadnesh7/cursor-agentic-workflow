---
name: discussion
description: Runs an interactive, multi-turn discussion with the user to flesh out a raw problem statement or feature topic. Drives the conversation with targeted Q&A, expands sparse ideas into concrete details (users, flows, data, constraints, edge cases), and proactively contributes the agent's own analysis — suggested angles, trade-offs, risks, and analogous patterns — until the topic is well understood. Use when the user offers a vague idea, asks to "talk through" or "discuss" something, or when prompt-enhancer / brainstorm needs a richer shared understanding before they can run.
---

# Discussion

Turns a fuzzy problem statement or feature topic into a shared, well-structured understanding through conversational Q&A enriched with the agent's own analysis. This skill is about *thinking together* with the user — not producing a final plan, not picking an approach, not writing code.

## When To Use

- The user opens with a vague topic: "I've been thinking about X", "let's discuss Y", "what about a feature that does Z".
- The prompt is too thin for `prompt-enhancer` to safely fill with assumptions alone.
- The user explicitly asks to talk something through before committing to scope.
- `brainstorm` or `researcher` needs richer context before they can be effective.
- Early-stage product/feature exploration where both user intent and solution space are open.

## When Not To Use

- The prompt is already concrete and actionable → go straight to `prompt-enhancer` or `brainstorm`.
- The user wants a decision or plan *now* → use `brainstorm`.
- The unknown is factual/technical, not intent-shaped → use `researcher`.
- The user is debugging a live issue → use `debugging`.

## Core Principles

1. **Listen first, enhance second.** Never overwrite the user's intent; expand and reflect it back.
2. **Ask to unblock, not to interrogate.** Every question must change what the final summary looks like.
3. **Contribute, don't hijack.** The agent offers insights as clearly-labeled additions the user can accept, reject, or ignore.
4. **Progress each turn.** Every round should make the topic noticeably more concrete.
5. **Converge, don't sprawl.** The goal is a crisp shared understanding, not an essay.
6. **Stay decision-free.** Note trade-offs, don't resolve them. Hand resolution to `brainstorm`.

## Inputs

- `raw_topic`: the user's initial problem statement or feature idea.
- `context` (optional): open files, workspace rules, prior session artifacts, previous conversations.
- `prior_discussion` (optional): earlier rounds in the same session.

## Workflow

Track progress with this checklist:

```
Discussion:
- [ ] Step 1: Restate the topic
- [ ] Step 2: Map the known vs unknown
- [ ] Step 3: Ask a focused question round
- [ ] Step 4: Add the agent's perspective
- [ ] Step 5: Integrate answers and iterate
- [ ] Step 6: Converge to a shared understanding
- [ ] Step 7: Hand off to the next skill
```

### Step 1: Restate The Topic

In one or two sentences, mirror back what you heard, using the user's own vocabulary. This anchors the conversation and surfaces misreads early.

```
## Restatement
<one- or two-sentence restatement of the topic>
```

Ask the user to confirm or correct the restatement implicitly — if they jump straight into answers, take that as confirmation.

### Step 2: Map Known vs Unknown

Produce a compact map of what you already know and what is missing. This is the agent's working notes, not a wall of text.

```
## What We Know
- <fact from the user or context>
- <fact from the user or context>

## What Is Unclear
- <dimension that needs clarification>
- <dimension that needs clarification>
```

Cover these dimensions when relevant:

- Who the users / consumers / stakeholders are
- Problem being solved and why now
- Current workaround or baseline
- Desired outcome and how success is measured
- Scope boundaries (in / out)
- Data involved (shape, source, volume, sensitivity)
- Key flows and states
- Constraints (performance, security, compliance, cost, deadlines)
- Environment and existing systems it must fit into

### Step 3: Ask A Focused Question Round

Ask **2–4 questions per round**, chosen to maximize information gain. Prefer open questions for exploration and multiple-choice questions when the option space is small and bounded.

- Use the `AskQuestion` tool when available for structured multi-select / single-select.
- Put free-form or deeply open questions in chat.
- Number the questions so the user can answer by reference.
- Include a sensible default in parentheses for any question the user might want to skip.

Good shapes:

- "Who is the primary user of this, and what are they doing right before they hit this problem?"
- "When you say 'real-time', do you mean sub-second, a few seconds, or 'same session'? (default: a few seconds)"
- "Is this meant to replace the current X, sit alongside it, or feed into it?"

Avoid:

- Yes/no questions that don't fork the design.
- Stacking more than 4 questions in a single round.
- Asking things you can infer from context or rules.

### Step 4: Add The Agent's Perspective

This is the *enhancement* step. After each question round (or alongside it), contribute clearly-labeled additions drawn from your own analysis. Keep each item short and tied to the topic.

Use these buckets, skipping any that aren't relevant:

```
## Agent Notes

### Angles Worth Considering
- <angle the user may not have named yet>

### Likely Edge Cases
- <edge case that commonly bites this kind of feature>

### Trade-offs On The Horizon
- <tension the user will eventually have to resolve (do not resolve it here)>

### Analogous Patterns
- <pattern / product / technique that maps to this problem>

### Risks & Unknowns
- <risk, assumption-to-validate, or dependency>
```

Rules for this section:

- Mark everything as a suggestion, not a conclusion. Phrasing like "worth considering", "often bites", "one pattern is".
- Never assert facts you haven't verified; if you're unsure, say so and recommend `researcher`.
- Never smuggle in a recommended solution — this skill is pre-decision.
- Prefer three sharp notes over ten generic ones.

### Step 5: Integrate Answers And Iterate

When the user responds:

1. Update "What We Know" and shrink "What Is Unclear".
2. Drop resolved questions; promote follow-ups that were unlocked by the answer.
3. Run another question round (Step 3) + agent-notes pass (Step 4) if meaningful unclarity remains.
4. Stop adding new dimensions once the picture is coherent — pile-on is an anti-pattern.

Cap the discussion at **3 question rounds** by default. If more are needed, check with the user: "Happy to keep digging, or shall we lock in what we have and move on?"

### Step 6: Converge To A Shared Understanding

When the topic is concrete enough to act on, produce a single **Discussion Summary** using this template:

```
# Discussion Summary

## Topic
<one-sentence framing of the problem or feature>

## Context & Motivation
<why this matters, who it affects, what triggered it>

## Users & Stakeholders
- <role> — <what they care about>

## Problem Statement
<plain-language description of the problem in its current form>

## Desired Outcome
- <observable, testable outcome>
- <observable, testable outcome>

## Scope
- In: <what is explicitly in scope>
- Out: <what is explicitly out of scope>

## Known Facts
- <fact / constraint confirmed with the user>

## Assumptions
- <assumption that still needs validation>

## Open Questions
- <question left unanswered, worth carrying forward>

## Agent-Contributed Notes
- <distilled angles, edge cases, trade-offs, risks the user accepted as worth tracking>

## Suggested Next Skill
<prompt-enhancer | brainstorm | researcher | dev-engineer | db-engineer | devops | tester>
```

Keep the summary under ~150 lines. If it bloats, drop low-signal detail rather than persisting separate notes.

### Step 7: Hand Off

Recommend the next skill based on where the discussion landed:

- Shared understanding is crisp and actionable → `prompt-enhancer` (to lock the intent) then `validate-prompt-intent`.
- Multiple viable solution shapes emerged → `brainstorm` (to compare approaches).
- Open questions are factual/technical → `researcher`.
- Unknown codebase context blocks further thinking → `get-project-context`.
- Discussion revealed a live bug or regression → `debugging`.

## Conversational Style

- Be direct. No "great question", no filler praise.
- One restatement, one map, one question round, one notes block per turn — not five.
- Prefer bullets to prose when listing dimensions.
- Quote the user's phrasing when restating; paraphrase when summarizing.
- If the user's answer contradicts an earlier assumption, name it and move on — don't re-litigate.

## Quality Bar

A discussion is done well when:

- The user recognizes the summary as *their* problem, sharpened.
- Every major dimension (users, outcome, scope, constraints, data, flows) has at least one concrete line.
- Agent-contributed notes are labeled as suggestions, not decisions.
- Open questions are answerable, not rhetorical.
- The handoff recommendation is obvious from the summary.

## Anti-Patterns

- Asking 10 questions up front and calling it a discussion.
- Turning the notes section into a hidden recommendation.
- Silently changing the topic to match the agent's preferred framing.
- Re-asking things the user already answered in earlier turns.
- Producing the Discussion Summary while major dimensions are still blank.
- Skipping the summary and handing the user raw chat scrollback.
- Letting the discussion loop forever — always converge or check in by round 3.
