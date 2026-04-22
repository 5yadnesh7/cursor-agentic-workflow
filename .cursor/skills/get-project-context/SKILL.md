---
name: get-project-context
description: Walks the entire current working directory end-to-end, maps the folder structure, dependencies, entry points, configs, services, data layer, tests, and how modules link together, then writes a durable PROJECT_CONTEXT.md that other skills reuse to avoid re-discovery. Use at the start of any non-trivial task, before reviewing or editing unfamiliar code, when reviewer / dev-engineer / brainstorm / debugging needs grounded context, or whenever the stored context is missing, stale, or insufficient.
---

# Get Project Context

Builds and maintains a single, reusable map of the project so other skills stop re-exploring the repo on every turn. Produces a living `PROJECT_CONTEXT.md` (plus optional indices) that is cheap to read and accurate enough to act on.

## When To Use

- First run in a new repo or workspace.
- `reviewer`, `dev-engineer`, `brainstorm`, `debugging`, or `tester` signals missing context.
- Stored `PROJECT_CONTEXT.md` is missing, older than the last significant change, or doesn't cover the area in question.
- The user asks "what does this project do", "how is it structured", or similar.

## Inputs

- `root_path` (default: current working directory).
- `mode`: `full` (first-time scan), `refresh` (update existing), `focused` (specific subtree or topic).
- `existing_context_path` (default `.cursor/context/PROJECT_CONTEXT.md`).
- `exclude_patterns` (default: `node_modules`, `.git`, `dist`, `build`, `.next`, `.venv`, `target`, `coverage`, binary/media, lockfiles unless explicitly requested).

## Output Artifacts

Write under `.cursor/context/` (create if missing):

- `PROJECT_CONTEXT.md` — the primary, human-readable map (required).
- `modules.md` — per-module summaries (optional, for large repos).
- `dependencies.md` — package and service dependencies (optional).
- `glossary.md` — domain terms and acronyms used in the codebase (optional).

Keep `PROJECT_CONTEXT.md` under ~500 lines; push detail into the optional files.

## Workflow

```
Project Context:
- [ ] Step 1: Decide mode and check existing context
- [ ] Step 2: Scan top-level layout
- [ ] Step 3: Detect stack and tooling
- [ ] Step 4: Identify entry points and boundaries
- [ ] Step 5: Map modules and their links
- [ ] Step 6: Capture data, config, and secrets layout
- [ ] Step 7: Capture build, test, deploy surface
- [ ] Step 8: Summarize conventions and workspace rules
- [ ] Step 9: Write PROJECT_CONTEXT.md
- [ ] Step 10: Freshness metadata and handoff
```

### Step 1: Decide Mode And Check Existing Context

- If `PROJECT_CONTEXT.md` exists and `mode=refresh`, load it and record the last-updated timestamp.
- If it exists but is stale or insufficient for the asked area, switch to `focused` mode for that subtree.
- If missing, run `full`.

### Step 2: Scan Top-Level Layout

List top-level directories and files. Respect `exclude_patterns`. Note workspaces/monorepo roots (`package.json` workspaces, `pnpm-workspace.yaml`, `nx.json`, `turbo.json`, `pyproject.toml` with subpackages, `go.work`, Cargo workspaces, Gradle multi-project, etc.).

### Step 3: Detect Stack And Tooling

From manifest and config files, detect:

- Languages and runtimes (versions from `.nvmrc`, `.tool-versions`, `pyproject`, `go.mod`, etc.).
- Frameworks (Next.js, Express, FastAPI, Django, React+Vite, NestJS, etc.).
- Package managers and lockfiles.
- Linters, formatters, type checkers, test runners.
- CI configuration (`.github/workflows`, `.gitlab-ci.yml`, `azure-pipelines.yml`, etc.).
- Container and infra files (`Dockerfile`, `docker-compose`, `k8s/`, `terraform/`, `pulumi/`, `helm/`).
- Environment files (`.env.example`, config schemas).

Never read actual secret values; only their presence and shape.

### Step 4: Identify Entry Points And Boundaries

- Executable entry points (`main`, `index`, `server`, `app`, CLI bins).
- HTTP/gRPC/GraphQL routers and their mount points.
- Background workers, cron, queue consumers.
- Frontend app shells and routing.
- Public packages or libraries exported from the repo.

### Step 5: Map Modules And Their Links

For each significant module/package:

- Purpose in one line.
- Key files or subfolders.
- Inbound dependencies (who uses it).
- Outbound dependencies (what it uses).
- External integrations (DBs, queues, third-party APIs).

Use the repo's own import graph as ground truth (imports, `require`, `use`, `go import`, etc.). Do not invent links.

### Step 6: Data, Config, And Secrets Layout

- Databases and engines in use; location of schemas/migrations.
- Caches and queues.
- Config loading order (files, env, secret managers).
- Where secrets live (names only, never values) and how they are injected.
- Feature flags and their store.

### Step 7: Build, Test, Deploy Surface

- How to install, build, run locally (from scripts, Makefile, `package.json` scripts, `justfile`, etc.).
- How tests are organized and run (unit/integration/e2e).
- How the app is packaged and deployed.
- Environments that exist and where they are defined.

### Step 8: Conventions And Workspace Rules

- Folder conventions, naming patterns, layering rules observed in code.
- Error-handling, logging, and observability patterns.
- Any `.cursor/rules/*.mdc` that apply to parts of the repo.
- Linter/formatter/type settings that imply style rules.

Prefer observed conventions over assumed best practices.

### Step 9: Write `PROJECT_CONTEXT.md`

Use this template exactly; trim empty sections:

```
# Project Context

> Generated by the get-project-context skill.
> Last updated: <ISO timestamp>
> Scope: <full | refresh | focused: subtree/topic>

## Summary
<2–5 sentences: what this project is, who uses it, primary capability>

## Stack
- Languages & runtimes: <...>
- Frameworks: <...>
- Package managers: <...>
- Testing: <...>
- Infra & deploy: <...>

## Top-Level Layout
<tree with one-line purpose per entry>

## Entry Points
- <path> — <runtime role>

## Modules
| Module | Purpose | Depends on | Used by |
|--------|---------|------------|---------|
| ...    | ...     | ...        | ...     |

## Data Layer
- Engines: <...>
- Schema / migration locations: <...>
- Caches / queues: <...>

## Configuration & Secrets
- Config sources: <...>
- Secret locations (names only): <...>
- Feature flags: <...>

## Build / Test / Deploy
- Install: <cmd>
- Dev run: <cmd>
- Test: <cmd>
- Build: <cmd>
- Deploy: <how and where>

## Conventions
- <observed rule>
- <observed rule>

## Workspace Rules In Effect
- <path under .cursor/rules> — <when it applies>

## Known Unknowns
- <what could not be confirmed from the repo>

## Suggested Deep Dives
- <module or area that deserves its own focused context pass>
```

### Step 10: Freshness Metadata And Handoff

- Stamp the file with an ISO timestamp and the git commit SHA if available.
- List "Known Unknowns" honestly; do not guess.
- Recommend the caller skill to re-invoke this skill in `focused` mode for any area they plan to change heavily.

## Refresh Strategy

- `refresh` mode compares the stored context against the current repo and updates only changed sections. Preserve stable sections to avoid churn.
- `focused` mode writes a scoped appendix in a sibling file (e.g. `PROJECT_CONTEXT.<area>.md`) rather than bloating the main doc.
- Treat context as stale if: new top-level modules appeared, stack versions changed, or the last update is older than the user's current task window.

## Coordination

- `reviewer` calls this skill first if context is missing or stale.
- `dev-engineer` and `brainstorm` should read `PROJECT_CONTEXT.md` before planning, and request a `focused` pass on the area they will touch.
- `debugging` uses the context to locate modules and dependencies faster.
- `devops` relies on the Build/Test/Deploy and Infra sections.

## Quality Bar

- A new engineer could orient in 5 minutes using `PROJECT_CONTEXT.md`.
- Every non-obvious claim is backed by a file path.
- Nothing in the doc is invented; unknowns are named.
- The doc is stable across refreshes unless the repo actually changed.
- File size stays under ~500 lines; detail lives in sibling files.

## Anti-Patterns

- Pasting long file listings instead of summarizing.
- Guessing architecture from folder names without checking imports.
- Recording secret values, tokens, or credentials.
- Rewriting the whole doc on each refresh instead of diffing.
- Claiming conventions the code does not actually follow.
