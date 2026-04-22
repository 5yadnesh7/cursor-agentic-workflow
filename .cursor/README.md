# Cursor Agentic Workflow

An opinionated, batteries-included workspace for driving real engineering work through Cursor. It wires together **workflows**, **slash commands**, **subagents**, **skills**, and **rules** into a single, gate-driven system so the agent does the right thing, in the right order, with the right specialist — every time.

> **TL;DR** — Type `/workflow-main` with any task. It enhances your prompt, classifies intent, and dispatches to the correct end-to-end workflow. Every workflow pulls in the specialist *skills* it needs and runs long/parallel tasks through *subagents*.

---

## Table of Contents

- [Mental Model](#mental-model)
- [Quick Start](#quick-start)
- [Slash Commands](#slash-commands)
- [Workflows](#workflows)
- [Skills](#skills)
- [Subagents](#subagents)
- [Rules](#rules)
- [Directory Layout](#directory-layout)
- [Decision Matrix — Which Thing Do I Use?](#decision-matrix--which-thing-do-i-use)

---

## Mental Model

Four layers, each with a single responsibility:

| Layer | Lives In | Role | Invoked By |
|---|---|---|---|
| **Slash Commands** | `.cursor/commands/` | Thin, user-facing entry points (what you type in chat). | **You** |
| **Workflows** | `.cursor/workflows/` | Orchestrated, gate-driven procedures that coordinate skills and subagents across phases. | Commands |
| **Skills** | `.cursor/skills/` | Reusable specialist personas (QA, DBA, DevOps, Reviewer…) with a concrete method. | Workflows |
| **Subagents** | `.cursor/agents/` | Scoped, often read-only helper agents launched via the Task tool for heavy or parallelizable work. | Skills / Workflows |
| **Rules** | `.cursor/rules/` | Stack-specific coding standards & conventions the agent must follow. | Loaded on demand |

**Flow:** `Command → Workflow → Gates → Skills (which may spawn Subagents) → Output`.

---

## Quick Start

1. Open a chat in Cursor inside this workspace.
2. Start with the universal entry:

   ```
   /workflow-main add a rate-limiter to the /login endpoint
   ```

3. The agent will:
   - Silently enhance + validate your prompt (via the **prompt-intake** loop).
   - Classify intent (fix / feature / refactor / audit / …).
   - Dispatch to the matching workflow with explicit approval gates.
4. If you already know the shape of the task, skip the router and call the specific command directly (e.g. `/workflow-fix`, `/workflow-release`).
5. Not sure which command? Type `/workflow-help`.

---

## Slash Commands

Slash commands are **thin entry points**. They do nothing except open and follow the matching workflow file in `.cursor/workflows/`. Pass the task description as the command argument.

### Universal Entry

| Command | Purpose | Use When |
|---|---|---|
| `/workflow-main` | Router: enhances, validates, classifies, and dispatches the prompt. | You don't want to think about which workflow fits. Default entry point. |
| `/workflow-help` | Read-only listing of all workflows. | You want to pick a workflow manually. |

### Lifecycle — Build, Change, Ship

| Command | Purpose | Use When |
|---|---|---|
| `/workflow-new-project` | Greenfield zero → first release, all 8 phases. | Starting a brand-new codebase. |
| `/workflow-feature` | End-to-end feature add on an existing project. | "Add X to our app." |
| `/workflow-fix` | Bug/regression with true root cause + regression test. | Something is broken or misbehaving. |
| `/workflow-refactor` | Behavior-preserving cleanup in small green-bar steps. | Renames, restructuring, module extraction, dependency upgrades. |
| `/workflow-review` | Code/PR review + conditional specialist audits. | Before merging, or reviewing a diff/PR. |
| `/workflow-release` | Ship an approved change-set with progressive rollout. | Cut a version, promote to prod. |
| `/workflow-migration` | Large migration using expand → migrate → contract. | Schema change, stack swap, large data move. |

### Specialist / Ops

| Command | Purpose | Use When |
|---|---|---|
| `/workflow-research` | Bounded citation-backed research → memo / ADR draft. | "Compare X vs Y", dependency choice, unknown tech. |
| `/workflow-audit` | Security / perf / a11y audit → remediation plan. | Pre-launch, post-incident, or compliance check. |
| `/workflow-incident` | Live outage response + blameless postmortem. | "Production is on fire." |
| `/workflow-docs` | Documentation-only (README, ADR, runbook, API ref…). | Docs are stale, missing, or explicitly requested. |
| `/workflow-infra` | Infra/IaC-only with plan → security → apply gates. | Cloud resources, CI/CD, observability, networking, clusters. |

### Context

| Command | Purpose | Use When |
|---|---|---|
| `/workflow-context` | Build or refresh `PROJECT_CONTEXT.md` in full / refresh / focused mode. | Onboarding a repo, or other workflows report stale/missing context. |

---

## Workflows

Workflows are the **real procedures** behind each command. They own phase structure, approval gates, and which skills/subagents run at each step. Open the file to see the exact script.

| Workflow | File | One-Liner |
|---|---|---|
| Main (Router) | `.cursor/workflows/workflow-main.md` | Prompt intake → intent classification → dispatch. |
| New Project | `workflow-new-project.md` | 8 phases: intent, discovery, foundations, scaffolding, build, quality, docs, release, close-out. |
| Feature | `workflow-feature.md` | Context → brainstorm → specialist branches → iterative slices → audits → docs. |
| Fix | `workflow-fix.md` | Reproduce → debugging → minimal fix → regression test → review. |
| Refactor | `workflow-refactor.md` | Lock behavior with tests → reviewer-designed plan → small steps → perf sanity. |
| Review | `workflow-review.md` | Reviewer + conditional specialist audits → severity-tagged findings. |
| Release | `workflow-release.md` | Pre-checks → manifest/notes → build → progressive rollout → watch window. |
| Migration | `workflow-migration.md` | Expand → migrate → contract with checkpoint gates and consumer waves. |
| Research | `workflow-research.md` | Capped research loop → synthesis with citations → typed deliverable. |
| Audit | `workflow-audit.md` | Pick modes (sec/perf/a11y) → run auditors → consolidated plan. |
| Incident | `workflow-incident.md` | Stabilize → verify → root cause → durable fix → postmortem. |
| Docs | `workflow-docs.md` | Context check → doc-writer → optional reviewer → docs-gate. |
| Infra | `workflow-infra.md` | Plan → security audit → apply-gate → progressive apply → verification. |
| Context | `workflow-context.md` | Run `get-project-context` → optional polish → docs-gate. |

Plus one shared reference document (not a runnable workflow):

| File | Purpose |
|---|---|
| `workflows/SUBAGENT_DISPATCH.md` | Canonical skill → subagent mapping, dispatch contract, parallelism rules, and context/destructive-gate escalation rules that every workflow obeys. |

Every workflow enforces: **no edits before intent-gate**, **one decision per turn**, **explicit approval at each gate**.

---

## Skills

Skills are **specialist personas** the workflows invoke. Each has a single `SKILL.md` that defines its method, inputs, outputs, and handoff rules. Read the file when a skill becomes relevant.

### Intent & Planning

| Skill | When To Use |
|---|---|
| `prompt-enhancer` | A prompt feels vague, ambiguous, or underspecified. Rewrites into clear, actionable form. |
| `validate-prompt-intent` | After `prompt-enhancer` — scores the prompt against a rubric and approves or loops. |
| `discussion` | The user's idea is sparse or exploratory; drive Q&A until the topic is well understood. |
| `brainstorm` | An approved prompt needs design thinking: requirement coverage, impact, approach comparison, plan. |
| `researcher` | Any unknown blocks progress — iterative, citation-backed web/codebase research. |

### Context

| Skill | When To Use |
|---|---|
| `get-project-context` | Start of any non-trivial task, or when stored `PROJECT_CONTEXT.md` is missing/stale. |

### Build

| Skill | When To Use |
|---|---|
| `dev-engineer` | Implement features end-to-end across backend, frontend, middleware, APIs, queues, caches. |
| `db-engineer` | Design schemas, indexes, migrations, partitioning, replication, query tuning for any engine. |
| `data-engineer` | Non-OLTP data: pipelines, ETL/ELT, streaming, warehouses, lakes, lineage. |
| `ml-engineer` | Classical ML, deep learning, or LLM/RAG/agent systems; training through deployment + monitoring. |
| `devops` | IaC, CI/CD, containers, Kubernetes, cloud, networking, observability, secrets, scaling, cost. |

### Verify & Harden

| Skill | When To Use |
|---|---|
| `reviewer` | Code/PR review with project-context grounding and severity-tagged feedback. |
| `tester` | Plan + execute QA: functional, regression, edge-case, a11y, basic perf/security via real UI. |
| `debugging` | Find root cause of bugs/flakes with evidence-driven investigation + minimal fix. |
| `security-auditor` | STRIDE threat modeling, authn/authz, secrets, dependencies, IaC misconfig. |
| `performance-engineer` | SLOs, load tests, profiling, bottleneck ID with before/after evidence. |
| `accessibility-auditor` | WCAG 2.2 AA audit with remediations. |

### Operate

| Skill | When To Use |
|---|---|
| `incident-commander` | Live incident: severity, comms, mitigation, timeline, postmortem. |
| `release-manager` | Versioning, changelogs, notes, signing, promotion, feature flags, rollback. |
| `doc-writer` | README, architecture, ADR, runbooks, API ref, changelogs, onboarding docs. |

---

## Subagents

Subagents are **scoped helper agents** launched via the Task tool. Each carries its own context window, loads its source-of-truth skill at start of run, and returns a small set of named output blocks (`APPROVED`, `CONTEXT_REQUIRED`, `REVIEW_REPORT`, `DESTRUCTIVE_GATE_REQUIRED`, …). Workflows branch on those blocks.

> The canonical skill → subagent mapping lives in `.cursor/workflows/SUBAGENT_DISPATCH.md`. Read that file when wiring up a workflow.

### Front Door (meta-orchestrator)

| Subagent | Readonly | Use When |
|---|:---:|---|
| `subagent-super` | No | Zero-config entry. The caller has no idea which skill, subagent, or workflow to invoke. Runs prompt-intake → classifies intent → dispatches the right specialist(s), runs a trivial inline skill, or emits a `HANDOFF` block naming the workflow to run. The only subagent that orchestrates other subagents. |

### Intake & Context

| Subagent | Readonly | Use When |
|---|:---:|---|
| `subagent-prompt-intake` | Yes | Silent enhancer ⇄ validator loop at the start of every workflow. |
| `subagent-context-scout` | Yes | Walk the repo and produce/refresh `PROJECT_CONTEXT.md` before review, design, or unfamiliar edits. |
| `subagent-researcher` | Yes | Iterative citation-backed research to unblock a skill that hit an unknown. |

### Builders (can write code/infra)

| Subagent | Readonly | Use When |
|---|:---:|---|
| `subagent-dev-engineer` | No | Implement approved plans end-to-end across backend, frontend, middleware, APIs, queues, caches. |
| `subagent-db-engineer` | No | Schemas, indexes, migrations, partitioning, replication, query tuning for any engine. |
| `subagent-devops-engineer` | No | IaC, CI/CD, containers, Kubernetes, cloud, networking, observability, secrets, scaling, cost. |
| `subagent-debug-investigator` | No | Deep root-cause analysis that may apply a minimal fix + regression guard. |
| `subagent-qa-tester` | No | Plan + execute verification through the real UI; asks before writing automated test code. |
| `subagent-doc-writer` | No | README, architecture, ADR, runbook, API reference, changelog, release notes. |

### Auditors (read-only, severity-tagged findings)

| Subagent | Readonly | Use When |
|---|:---:|---|
| `subagent-code-reviewer` | Yes | Post-implementation review of a diff/PR against correctness, security, perf, tests, docs. |
| `subagent-security-auditor` | Yes | STRIDE threat modeling, authn/authz, secrets, dependencies, IaC misconfig. |
| `subagent-performance-auditor` | Yes | SLO definition, load tests, profiling, bottleneck ID with before/after evidence. |
| `subagent-accessibility-auditor` | Yes | WCAG 2.2 AA audit with concrete remediations. |

> **Rule of thumb:** if the task is a needle query, call tools directly. If it's broad exploration, parallel work, or a long chain — delegate to a subagent. Use `subagent-super` when even the routing is unclear.

---

## Rules

Stack-specific coding standards in `.cursor/rules/`. They load on demand when files matching their globs are touched.

| Area | Files |
|---|---|
| **Databases** | `db-mongodb`, `db-redis`, `db-schema-mysql`, `db-schema-postgres`, `db-schema-sql` |
| **Secrets / Env** | `env-secrets` |
| **Node / Express** | `express-01-core` → `express-06-ops` (core, routes/services, middleware/auth, db-api, quality, ops) |
| **Next.js** | `next-01-core` → `next-05-ops` (core, api-state, components, quality, ops) |
| **React + Vite** | `react-vite-01-core` → `react-vite-05-ops` |
| **Python** | `python-01-core` → `python-05-ops` (core, structure, api-services, quality, ops) |
| **Terraform** | `terraform-01-core` → `terraform-05-ops` (core, modules, state/providers, patterns, ops) |
| **Playwright** | `playwright-01-core`, `playwright-02-ops` |

---

## Directory Layout

```
.cursor/
├── agents/       # Subagent definitions (.md with front-matter + method)
├── commands/     # Slash-command entry points (thin; just open a workflow)
├── context/      # PROJECT_CONTEXT.md + sibling context files
├── rules/        # Stack-specific standards loaded on demand
├── skills/       # Specialist skills, one folder per skill, each with SKILL.md
└── workflows/    # Full orchestrated procedures behind each command
    └── SUBAGENT_DISPATCH.md  # Shared skill → subagent mapping + dispatch contract
```

---

## Decision Matrix — Which Thing Do I Use?

| I want to… | Reach for |
|---|---|
| Just describe my task and have the system route it | `/workflow-main` |
| Pick a workflow manually | `/workflow-help` |
| Build something new from scratch | `/workflow-new-project` |
| Add to an existing project | `/workflow-feature` |
| Fix a bug | `/workflow-fix` |
| Clean up code without changing behavior | `/workflow-refactor` |
| Get a PR/diff reviewed | `/workflow-review` |
| Ship a version | `/workflow-release` |
| Do a large cross-cutting migration | `/workflow-migration` |
| Answer "X vs Y" with citations | `/workflow-research` |
| Audit security / perf / a11y | `/workflow-audit` |
| Respond to an outage | `/workflow-incident` |
| Only touch docs | `/workflow-docs` |
| Only touch infra | `/workflow-infra` |
| Refresh or seed `PROJECT_CONTEXT.md` | `/workflow-context` |
| Have a specialist do one focused thing mid-conversation | Invoke the matching **skill** directly (e.g. ask for `db-engineer` or `reviewer`). |
| Run a heavy read-only side-quest | Delegate to a read-only **subagent** (`subagent-researcher`, `subagent-context-scout`, `subagent-code-reviewer`, `subagent-security-auditor`, `subagent-performance-auditor`, `subagent-accessibility-auditor`). |
| Parallelize several specialists (e.g. all three auditors) | Dispatch multiple subagents in one turn — see rules in `workflows/SUBAGENT_DISPATCH.md`. |
| Hand off the whole routing decision to the agent | Call `subagent-super` — it classifies the request and dispatches the right thing. |

---

## Conventions

- **One decision per turn.** Every workflow ends each turn with a reply menu on the last line.
- **Explicit gates.** Intent-gate, design-gate, review-gate, docs-gate, rollout-gate — nothing advances without approval.
- **Project context is king.** Any non-trivial review, design, or implementation checks `PROJECT_CONTEXT.md` first.
- **No session clutter.** Workflows never spawn `SESSION.md`, `INDEX.md`, `handoffs/`, `decisions.md`, or similar bookkeeping files.
- **Minimal, reversible changes.** Fixes and refactors are small, green-bar, and testable at every step.
