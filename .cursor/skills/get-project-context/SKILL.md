---
name: get-project-context
description: "Performs a deep, incremental, phase-by-phase discovery of ANY project (small or monorepo-scale) and writes PROJECT_CONTEXT.md plus sibling files every other skill can reuse. Use at start of non-trivial tasks, before editing unfamiliar code, or when reviewer / dev-engineer / brainstorm / debugging signals missing or stale context."
---

# Get Project Context

## When To Use

| Situation | Mode |
|-----------|------|
| First run in a repo / workspace | `full` |
| `PROJECT_CONTEXT.md` missing | `full` |
| Existing context is stale or incomplete | `refresh` |
| Targeting one area before editing | `focused:<subtree>` |
| Another skill (reviewer, dev-engineer, brainstorm, debugging) asks for context | whichever fits |

---

## Inputs

| Parameter | Default | Notes |
|-----------|---------|-------|
| `root_path` | CWD | Absolute or relative path to project root |
| `mode` | `full` | `full` \| `refresh` \| `focused:<subtree-or-topic>` |
| `existing_context_path` | `.cursor/context/PROJECT_CONTEXT.md` | Used in `refresh` / `focused` modes |
| `exclude_patterns` | see below | Comma-separated glob patterns to skip |
| `size_hint` | `auto` | `small` (<200 files) \| `medium` (<2k files) \| `large` (2k+) \| `auto` |

**Default `exclude_patterns`:**  
`node_modules, .git, dist, build, .next, .venv, __pycache__, target, coverage,  
.turbo, .cache, *.lock, *.log, *.min.js, *.min.css, *.map, *.wasm,  
*.png, *.jpg, *.jpeg, *.gif, *.svg, *.ico, *.ttf, *.woff, *.woff2, *.mp4, *.mp3`

---

## Output Artifacts

All files live under `.cursor/context/` (create directory if missing).

| File | Required | Purpose |
|------|----------|---------|
| `PROJECT_CONTEXT.md` | ✅ | Primary human-readable project map — **no length limit; completeness always wins over brevity** |
| `TREE_SNAPSHOT.md` | ✅ | Full working-tree snapshot taken at scan start; never regenerated unless explicitly requested |
| `modules.md` | large projects | One section per significant module |
| `dependencies.md` | when deps are complex | All package + service deps |
| `data_layer.md` | when DB/queue details are deep | Schema, migrations, caching topology |
| `infra.md` | when IaC / CI details are deep | Cloud resources, CI jobs, environment topology |
| `glossary.md` | domain-rich codebases | Domain terms and acronyms from the code |
| `PROJECT_CONTEXT.<area>.md` | focused mode | Scoped appendix; does not bloat the main doc |

---

## Phased Workflow

```
Phase 0 — Guard & Mode Selection
Phase 1 — Working Tree Snapshot          [flush to TREE_SNAPSHOT.md]
Phase 2 — Stack & Tooling Detection      [flush partial context]
Phase 3 — Entry Points & Boundaries      [flush partial context]
Phase 4 — Incremental Module Walk        [flush after every module]
Phase 5 — Data, Config & Secrets Layout  [flush partial context]
Phase 6 — Build / Test / Deploy Surface  [flush partial context]
Phase 7 — Conventions & Workspace Rules  [flush partial context]
Phase 8 — Reframe & Final Assembly       [write final PROJECT_CONTEXT.md]
Phase 9 — Freshness Stamp & Handoff
```

> **Flushing rule:** After every phase and every module in Phase 4, append findings to
> `.cursor/context/.wip_context.md`. Interrupted runs resume from the last phase marker.

---

### Phase 0 — Guard & Mode Selection

```
[ ] 0.1  Check if .cursor/context/PROJECT_CONTEXT.md exists.
         - Exists + mode=full  → warn user, confirm overwrite or switch to refresh.
         - Exists + mode=refresh → load it; record last-updated timestamp and commit SHA.
         - Missing + mode=refresh or focused → downgrade to full, log the downgrade.
[ ] 0.2  Check for .cursor/context/.wip_context.md (interrupted previous run).
         - Found → ask user: resume from last phase marker OR start fresh.
[ ] 0.3  Detect size_hint if set to auto:
         - Count files in root (excluding exclude_patterns).
         - < 200  → small   (single-pass, all phases inline)
         - 200–2000 → medium  (phases sequential, flush after each)
         - 2000+  → large   (phases sequential, flush after every module in Phase 4)
[ ] 0.4  Log: "Mode=<X>  Size=<Y>  Root=<Z>  Resuming=<yes|no>"
```

---

### Phase 1 — Working Tree Snapshot

> Captures the full directory tree once as ground truth for all later phases.
> Never regenerated unless `TREE_SNAPSHOT.md` is deleted or `mode=full` re-run.

```
[ ] 1.1  Run: tree -L 6 --dirsfirst -I "<exclude_patterns>" <root_path>
         (or equivalent: find / ls -R with depth limit if tree is unavailable)
[ ] 1.2  Annotate top-level entries with a one-line purpose inferred from name + content.
[ ] 1.3  For monorepos: identify workspace roots and list each package/app separately.
[ ] 1.4  Read README files (root + each monorepo workspace root):
         Extract: purpose, target users, setup, architecture overview, external links.
         If absent, record "README: absent" as a Known Unknown.
[ ] 1.5  Read CHANGELOG.md / HISTORY.md / RELEASES.md if present:
         Extract most recent 20 lines: current version, last release, active change areas.
         Record a one-line "Current state" note.
[ ] 1.6  Note LICENSE type (MIT, Apache, GPL, proprietary) if present.
         Read CODEOWNERS if present — maps path patterns to owning teams.
[ ] 1.7  Write .cursor/context/TREE_SNAPSHOT.md → template in reference.md.
[ ] 1.8  Flush marker: append "## PHASE_1_COMPLETE <ISO>" to .wip_context.md
```

> `TREE_SNAPSHOT.md` template → [reference.md](reference.md#tree-snapshot-template)

---

### Phase 2 — Stack & Tooling Detection

```
[ ] 2.1  Scan manifest / config files (never read secret values):
         JS/TS    → package.json, tsconfig.json, .nvmrc, .node-version
         Python   → pyproject.toml, setup.cfg, setup.py, requirements*.txt, .python-version
         Go       → go.mod, go.sum
         Rust     → Cargo.toml
         Java     → pom.xml, build.gradle, .sdkmanrc
         Ruby     → Gemfile, .ruby-version
         .NET     → *.csproj, global.json
         Misc     → .tool-versions, mise.toml
         Monorepo → turbo.json, nx.json, pnpm-workspace.yaml (pipeline / task config)
         Bundler  → vite.config.ts, webpack.config.js, next.config.js, rollup.config.js
                    (note aliases, output targets, env injection, proxies)
[ ] 2.2  Detect frameworks from deps list:
         Web    → Next.js, Nuxt, Remix, SvelteKit, Express, Fastify, NestJS,
                  Django, FastAPI, Flask, Rails, Spring, Laravel, etc.
         Mobile → React Native, Expo (app.json / app.config.js), Flutter
                  (pubspec.yaml), Swift (Package.swift), Kotlin (build.gradle.kts)
[ ] 2.3  Detect test runners (Jest, Vitest, Pytest, Go test, RSpec, Playwright, Cypress, etc.);
         read their config files (jest.config.js, vitest.config.ts, playwright.config.ts, etc.)
         — note coverage thresholds, setup files, excluded directories.
[ ] 2.4  Detect linters / formatters: ESLint, Prettier, Ruff, Black, Golint, Rubocop, etc.
[ ] 2.5  Detect CI: .github/workflows, .gitlab-ci.yml, .circleci, Jenkinsfile,
         azure-pipelines.yml, bitbucket-pipelines.yml, etc.
[ ] 2.6  Detect container / infra: Dockerfile, docker-compose.yml, k8s/, helm/,
         terraform/, pulumi/, cdk/, serverless.yml, fly.toml, railway.json, etc.
[ ] 2.7  Detect package managers: npm/yarn/pnpm/bun (from lockfile presence),
         pip/poetry/pdm/uv, cargo, go modules, maven/gradle.
[ ] 2.8  Flush findings to .wip_context.md with marker "## PHASE_2_COMPLETE <ISO>"
```

---

### Phase 3 — Entry Points & Boundaries

```
[ ] 3.1  Find executable entry points:
         main.*, index.*, server.*, app.*, cmd/ (Go), __main__.py, bin/
         package.json "main"/"bin"/"scripts.start"; pyproject.toml [project.scripts]
[ ] 3.2  Map HTTP / gRPC / GraphQL routers:
         - Scan for router definitions (express Router, @Controller, APIRouter, etc.)
         - Record mount paths and HTTP methods if discoverable.
         - WebSocket (ws, Socket.IO, ActionCable): record path and event namespaces.
         - SSE: scan for `text/event-stream` handlers — record path and event names.
[ ] 3.3  API contract files — read explicitly (they define the full API surface):
         OpenAPI/Swagger: openapi.yaml, swagger.json, api.yaml, docs/api.*, etc.
         GraphQL: schema.graphql, *.gql — record types, queries, mutations, subscriptions.
         AsyncAPI / gRPC: asyncapi.yaml, *.proto — record services and methods.
         Record: file path, API version, top-level resources / operations.
[ ] 3.4  Find background workers, cron jobs, queue consumers, event-driven patterns:
         BullMQ, Celery, Sidekiq, Temporal, Kafka consumers, cron expressions;
         custom event buses / domain event emitters (EventEmitter, MediatR, etc.).
[ ] 3.5  Find frontend app shells and routing:
         pages/, app/, routes/ (Next.js, Remix, SvelteKit, etc.) — root layout + route tree.
[ ] 3.6  Find public packages / libraries exported:
         "exports" in package.json, __init__.py exports, lib/ or pkg/ dirs.
[ ] 3.7  Flush findings to .wip_context.md with marker "## PHASE_3_COMPLETE <ISO>"
```

---

### Phase 4 — Incremental Module Walk

> **small**: walk all modules in one pass.  
> **medium/large**: one module at a time, flush after each, resume on interrupt.

```
[ ] 4.0  Build the walk list from TREE_SNAPSHOT.md. Tag each entry as app or infra:
         App modules  — top-level code dirs + workspace packages
                        (src/, lib/, packages/, apps/, services/, shared/, etc.)
         Infra modules — terraform/, tofu/, infra/, k8s/, helm/, pulumi/, cdk/,
                         .github/workflows/, serverless.yml, fly.toml, railway.json,
                         render.yaml, ansible/, .circleci/
         Walk app modules first (steps 4.M below), then infra modules (steps 4.I below).

         For each app module M:

  [ ] 4.M.1  Read key files in M — priority order:
             1. README.md / README.rst (most reliable source of module purpose)
             2. Entry file(s): index.*, main.*, mod.rs, __init__.py, etc.
             3. Types / interfaces file
             4. Core logic files (largest non-test files)
             5. Test file(s) — describe/it/test headers only
             No README → proceed to step 2, note "no module README".
  [ ] 4.M.2  Determine: purpose (one sentence), public API, inbound deps (grep imports
             across repo), outbound deps, external integrations (APIs / SDKs / DB clients).
  [ ] 4.M.3  Note TODOs, FIXMEs, HACKs in key files — record as "known risks".
  [ ] 4.M.4  Flush: append M summary block to .wip_context.md with marker
             "## MODULE_<M>_COMPLETE <ISO>"

[ ] 4.FINAL  Assemble all module summaries into modules.md (for medium/large projects).
             Flush marker "## PHASE_4_COMPLETE <ISO>"

         For each infra module I:

  [ ] 4.I.1  IaC (Terraform/Pulumi/CDK): read main.tf / Pulumi.yaml / bin+lib stacks.
             Record: provider(s), resource types, environment dirs, output values.
             Config mgmt (Ansible/Chef/Puppet): read site.yml / playbooks/ — record roles, hosts.
  [ ] 4.I.2  K8s / Helm: read Chart.yaml, values.yaml, values-<env>.yaml.
             Record: Deployments, Services, Ingresses, namespaces, ingress hosts, limits.
  [ ] 4.I.3  CI/CD pipelines: read every workflow file (.github/workflows/, .gitlab-ci.yml…).
             Record: job names, triggers, deploy targets, required secrets, promo chain.
  [ ] 4.I.4  Serverless / Edge (serverless.yml, fly.toml, railway.json, render.yaml):
             Record: function names, runtime, event triggers, regions, health-check paths.
  [ ] 4.I.5  Flush: append infra summary to .wip_context.md with marker
             "## INFRA_<I>_COMPLETE <ISO>"

[ ] 4.I.FINAL  Assemble infra summaries into infra.md (for medium/large projects).
               Flush marker "## PHASE_4I_COMPLETE <ISO>"
```

> Module summary block format → [reference.md](reference.md#module-summary-block)

---

### Phase 5 — Data, Config & Secrets Layout

```
[ ] 5.1  Databases:
         - Detect ORM / query builder (Prisma, TypeORM, Drizzle, SQLAlchemy, GORM,
           ActiveRecord, Hibernate, etc.)
         - Locate schema files (schema.prisma, models.py, *.entity.ts, migrations/)
         - Record engines: PostgreSQL, MySQL, SQLite, MongoDB, DynamoDB,
           search (Elasticsearch, OpenSearch), vector (pgvector, Pinecone, Weaviate), etc.
[ ] 5.2  Caches & Queues:
         Redis, Memcached, BullMQ, RabbitMQ, Kafka, SQS, Pub/Sub, NATS — where configured.
[ ] 5.3  Config loading order:
         .env, .env.local, .env.<environment>, config.yaml, config.ts — record priority order.
[ ] 5.4  .env.example / .env.sample — read explicitly (authoritative key list):
         Record every key name (never values); note required vs optional.
         If absent but .env.* files exist → Known Unknown: "required keys undocumented".
[ ] 5.5  Secrets (names only, NEVER values):
         List secret env var names (API_KEY, DATABASE_URL, etc.); note secret manager
         integration: AWS Secrets Manager, Vault, Doppler, Infisical.
[ ] 5.6  Feature flags: LaunchDarkly, Unleash, Flagsmith, custom DB table, env var booleans.
[ ] 5.7  Auth / Authz:
         - Auth provider: NextAuth/Auth.js, Passport.js, Django-allauth, Devise,
           OAuth2/OIDC (Auth0, Keycloak, Okta), Supabase Auth, custom JWT middleware.
         - Locate auth entry point (middleware, guards, decorators) — record file paths.
         - Permission model: RBAC, ABAC, row-level security, or none.
[ ] 5.8  Flush findings to .wip_context.md with marker "## PHASE_5_COMPLETE <ISO>"
```

---

### Phase 6 — Build / Test / Deploy Surface

```
[ ] 6.1  Install commands (from package.json scripts, Makefile, justfile, README):
         How to install dependencies; how to seed / migrate the database locally.
[ ] 6.2  Dev run:
         - Primary dev server command + required env vars (cross-reference 5.4).
         - If docker-compose.yml / compose.yaml exists, read it and record:
           service names + roles, ports (host:container), named volumes, depends_on
           relationships. Write as "Local service map" in the output.
[ ] 6.3  Test: unit cmd, integration/e2e cmd, coverage cmd, single-file cmd.
[ ] 6.4  Build: production cmd, output directory, artifacts produced.
[ ] 6.5  Deploy: how and where (Vercel, AWS, GCP, Railway, Fly.io, self-hosted, etc.);
         environments defined; CD pipeline file and trigger.
[ ] 6.6  Flush findings to .wip_context.md with marker "## PHASE_6_COMPLETE <ISO>"
```

---

### Phase 7 — Conventions & Workspace Rules

```
[ ] 7.1  Folder conventions: naming patterns (kebab-case, PascalCase, etc.);
         layering (controllers → services → repositories, etc.)
[ ] 7.2  Error handling: custom error classes, Result type, try/catch conventions,
         HTTP error response shape.
[ ] 7.3  Logging & observability: logger library (Winston, Pino, structlog, zerolog, etc.);
         tracing (OpenTelemetry, Datadog, Sentry); log level conventions.
[ ] 7.4  Code style (from config, not assumption): significant ESLint / Ruff / Golint rules,
         import order; read .editorconfig if present (indent style/size, charset, line endings).
[ ] 7.5  Human-facing convention documents — read each if present:
         - CONTRIBUTING.md: branch naming, PR process, commit style, release process.
         - AGENTS.md: Cursor agent task rules, tool permissions, workflow constraints.
         - .cursorrules: legacy Cursor rules file — read fully, extract all constraints.
         - CODEOWNERS: maps path patterns to owning teams — record key ownership entries.
         - SECURITY.md: note existence (vulnerability disclosure / response process).
         - .husky/, .pre-commit-config.yaml, lefthook.yml: note pre-commit hooks enforced.
         - CODE_OF_CONDUCT.md: note existence only.
         Record: path → key rules it imposes.
[ ] 7.6  Architecture Decision Records (ADRs):
         Check: docs/adr/, docs/decisions/, docs/architecture/, adr/, .adr/, records/
         If found: list every ADR with number, title, status, one-line decision summary.
         If directory exists but empty: Known Unknown "ADR directory unpopulated".
[ ] 7.7  Cursor workspace rules: read all .cursor/rules/*.mdc files;
         record: path → when it applies → key constraints it imposes.
[ ] 7.8  Flush findings to .wip_context.md with marker "## PHASE_7_COMPLETE <ISO>"
```

---

### Phase 8 — Reframe & Final Assembly

> Reads all findings from `.wip_context.md` and assembles final output files —
> clean, structured, no draft artifacts.

```
[ ] 8.1  Read .wip_context.md top to bottom, collecting all phase outputs.
[ ] 8.2  Write sibling files for medium/large repos (source: .wip_context.md):
  [ ] 8.2a  modules.md — one section per module using the module summary blocks
            written during Phase 4. Include all fields: path, purpose, public API,
            internal deps, external deps, used-by, integrations, known risks.
  [ ] 8.2b  dependencies.md — write when the project has non-trivial dependency
            complexity. Two sections:
            "Package Dependencies": name | version | purpose | used by (modules)
            "Service Dependencies": name | endpoint or engine | purpose | used by
  [ ] 8.2c  data_layer.md — write when DB/queue details are deep. Full schema
            model list, migration history summary, index notes, queue topology.
  [ ] 8.2d  infra.md — write when IaC or CI/CD details are deep. Two sections:
            "Cloud Resources": type | name | environment | provisioned by (IaC file)
            "CI/CD Jobs": job | trigger | deploys to | required secrets
  [ ] 8.2e  glossary.md — write for domain-rich codebases or heavy domain jargon.
            Source: README, type defs, schema names, ADRs, core logic comments.
            Format: term | definition | first seen in (file path)
[ ] 8.3  Write PROJECT_CONTEXT.md using the canonical template in reference.md.
         Cross-reference sibling files with links; do NOT truncate for length.
[ ] 8.4  Validate:
         - Every entry point listed in Phase 3 appears in PROJECT_CONTEXT.md.
         - Every app module covered in Phase 4 appears in the Module Map table.
         - Every infra module from Phase 4.I appears in Section 11 (Infrastructure Map).
         - No secret values present anywhere in output files.
         - No invented facts; unknowns are listed under Known Unknowns.
[ ] 8.5  Cleanup — conditional on whether the module walk completed:
         - If Phase 4 and 4.I both completed fully:
           · Delete .cursor/context/.wip_context.md.
         - If Phase 4 or 4.I was INCOMPLETE (large project hit token/time limit):
           · Do NOT delete .wip_context.md — the next run needs it to resume.
           · Rename the file written in 8.3 to PROJECT_CONTEXT.partial.md and prepend:
             "⚠️ INCOMPLETE — module walk stopped at <module N> of <total>.
              Run get-project-context again to continue. Do not rely on this file."
           · Do NOT overwrite any existing complete PROJECT_CONTEXT.md.
[ ] 8.6  Flush marker "## PHASE_8_COMPLETE <ISO>" to a fresh status log.
```

---

## Canonical `PROJECT_CONTEXT.md` Template

> **Full 17-section output template** → [reference.md](reference.md)

Key rules: all 17 sections required; inapplicable → `N/A — not present`; never truncate
for length; no secret values; every claim backed by a file path; unknowns named explicitly.

---

### Phase 9 — Freshness Stamp & Handoff

```
[ ] 9.1  Write git SHA (git rev-parse HEAD) into the header of PROJECT_CONTEXT.md.
         If git is unavailable, write "N/A".
[ ] 9.2  Record the list of files modified in this run in a trailing comment block
         at the bottom of PROJECT_CONTEXT.md:
         <!-- Files written: PROJECT_CONTEXT.md, TREE_SNAPSHOT.md, modules.md -->
[ ] 9.3  Print a handoff summary to the user:
         "✅ Context ready. Files written: <list>.
          Next: ask reviewer / dev-engineer / debugging to read PROJECT_CONTEXT.md
          before acting. For focused work on <module>, re-run in focused:<module> mode."
[ ] 9.4  If Phase 4 was INCOMPLETE (partial .md written):
         "⚠️ MODULE WALK INCOMPLETE — covered <N> of <total> modules.
          A PROJECT_CONTEXT.partial.md was written. Run again to continue;
          the existing .wip_context.md will be resumed automatically."
```

---

## Refresh Strategy

| Scenario | Action |
|----------|--------|
| New top-level module | `refresh` — add module block, update table |
| Stack version bump | `refresh` — update Stack section only |
| Editing one subsystem | `focused:<subsystem>` — write sibling `PROJECT_CONTEXT.<area>.md` |
| Context older than task window | `refresh` — diff and update changed sections only |
| `TREE_SNAPSHOT.md` missing | Regenerate it before refresh |

**Diff first, write only what changed** — rewriting stable sections creates noisy diffs.

---

## Coordination With Other Skills

| Skill | Uses |
|-------|------|
| `reviewer` | Call this skill first if context missing or stale |
| `dev-engineer` | Summary + Module Map before planning; `focused` mode before editing |
| `brainstorm` | Summary + Stack + Module Map |
| `debugging` | Entry Points + Module Map + Data Layer |
| `devops` | Build/Test/Deploy + Infra sections |
| `tester` | Module Map + Conventions |

---

## Quality Bar

- [ ] A new engineer could orient in under 5 minutes using `PROJECT_CONTEXT.md`.
- [ ] Every non-obvious claim is backed by a file path; nothing is invented.
- [ ] No secret values anywhere in any output file.
- [ ] `PROJECT_CONTEXT.md` is as long as it needs to be — never truncated.
- [ ] Sibling files organise detail; they do not hide it from the main file.
- [ ] All phases marked complete in status log; large-project progress always resumable.

---

## Anti-Patterns To Avoid

- Listing files without annotation instead of explaining what they do
- Guessing architecture from folder names without checking actual imports
- Recording secret values or tokens anywhere in output files
- Rewriting the entire doc on every refresh instead of diffing
- Claiming conventions the code does not actually follow
- Walking all modules at once on a large repo; skipping the working tree snapshot
- Mixing `.wip_context.md` draft content with final output files
