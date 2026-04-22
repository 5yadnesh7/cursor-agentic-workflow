# Subagent Dispatch Map

Canonical mapping from skills (source of truth, under `.cursor/skills/`) to their subagent wrappers (under `.cursor/agents/`).

**Rule of thumb for workflows:** when a step says "Invoke `<skill>`", dispatch via the `Task` tool to the corresponding `subagent-<name>` if one exists in the table below. Skills remain the authoritative procedure — each subagent loads its skill file at start of run.

## Front Door: `subagent-super`

`subagent-super` is the zero-config entry subagent. Use it when the caller (or user) does not know which skill, subagent, or workflow to invoke. It:

1. Runs `subagent-prompt-intake` internally to enhance + validate the raw request.
2. Classifies intent using the `workflow-main` routing table.
3. Either dispatches a single specialist subagent, parallel-dispatches several (e.g. all three auditors), runs a trivial inline skill, or emits a `HANDOFF` block naming the workflow to run.
4. Never approves destructive gates — it relays them back to the caller.

It is the ONLY subagent that orchestrates other subagents. All others coordinate by returning structured output to their caller; the caller (workflow, main agent, or `subagent-super`) decides the next dispatch.

## Mapping

| Skill (source of truth) | Subagent wrapper | Readonly | Primary callers |
|---|---|---|---|
| _(meta — orchestrates all others)_ | `subagent-super` | no | User / main agent when the route is unknown |
| `prompt-enhancer` + `validate-prompt-intent` | `subagent-prompt-intake` | yes | Every workflow's intent gate (Phase 0 / Step 1), and Phase 1 of `subagent-super` |
| `get-project-context` | `subagent-context-scout` | yes | `workflow-context`, and context-gate of every other workflow |
| `researcher` | `subagent-researcher` | yes | `workflow-research`, plus any workflow that hits an unknown |
| `reviewer` | `subagent-code-reviewer` | yes | `workflow-review`, and review-gate of feature/fix/refactor/new-project |
| `debugging` | `subagent-debug-investigator` | no | `workflow-fix`, and anywhere a failure needs root cause |
| `security-auditor` | `subagent-security-auditor` | yes | `workflow-audit` security mode, quality-bar phase of new-project/feature/release |
| `performance-engineer` | `subagent-performance-auditor` | yes | `workflow-audit` performance mode, release rollout watch |
| `accessibility-auditor` | `subagent-accessibility-auditor` | yes | `workflow-audit` a11y mode, UI feature quality bar |
| `dev-engineer` | `subagent-dev-engineer` | no | Any workflow that builds application code |
| `db-engineer` | `subagent-db-engineer` | no | Schema / migration work in feature/fix/refactor/migration/new-project |
| `devops` | `subagent-devops-engineer` | no | `workflow-infra`, release rollout, IaC/CI remediations |
| `tester` | `subagent-qa-tester` | no | test-gate in feature/fix/refactor/new-project/release |
| `doc-writer` | `subagent-doc-writer` | no | `workflow-docs`, docs-gate in every builder workflow, release notes |

## Skills without a subagent wrapper

These remain pure skills (invoked inline by the workflow or by a subagent that needs them):

- `brainstorm` — conversational by nature (gates, approach picks, design iteration)
- `discussion` — conversational; user is the co-author
- `data-engineer` — add if project needs it
- `ml-engineer` — add if project needs it
- `release-manager` — orchestrator skill, invoked by `workflow-release`
- `incident-commander` — orchestrator skill, invoked by `workflow-incident`

## Dispatch contract

1. **Inputs**: when dispatching to a subagent, pass the exact inputs listed in that subagent's `## When Invoked` section. The subagent starts with a clean context window — include everything it needs.
2. **Output contract**: every subagent returns one of a small set of named blocks (`APPROVED`, `CONTEXT_REQUIRED`, `REVIEW_REPORT`, `DESTRUCTIVE_GATE_REQUIRED`, etc.). The workflow branches on the block type.
3. **Parallelism**: workflows MAY dispatch multiple independent subagents in parallel (e.g. audit mode with security + perf + a11y) by emitting multiple `Task` calls in a single assistant turn.
4. **Context escalation**: if a subagent returns `CONTEXT_REQUIRED`, the workflow dispatches `subagent-context-scout` first, then re-invokes the original subagent.
5. **Destructive gate**: if a builder subagent returns `DESTRUCTIVE_GATE_REQUIRED`, the workflow holds its `destructive-gate` before re-invoking with `approved_destructive: true`.
6. **User-facing turns remain the workflow's job.** Subagents return structured output; the workflow owns the chat turn and formats the one-decision-per-turn reply menu.
