# Canonical `PROJECT_CONTEXT.md` Template

> Referenced by `SKILL.md` Phase 8.3.  
> This is the exact structure the skill must produce. All 17 sections are required.
> Omit only clearly inapplicable subsections and mark them `N/A — not present in
> this repo` so readers know the check was done, not skipped.

---

```markdown
# Project Context

> **Skill:** get-project-context
> **Mode:** <full | refresh | focused: subtree/topic>
> **Generated:** <ISO 8601 timestamp>
> **Git SHA:** <sha or N/A>
> **Size class:** <small | medium | large>
> **Sibling files:** [TREE_SNAPSHOT.md] [modules.md?] [dependencies.md?] [data_layer.md?] [infra.md?] [glossary.md?]

---

## 1. Summary

<3–6 sentences. Be specific — not "a web app" but "a multi-tenant SaaS invoicing
platform built with Next.js 14 App Router and a Postgres-backed REST API, currently
serving ~50k users." Include: what it does, who uses it, primary tech, scale/maturity.>

---

## 2. Project State

- **Current version:** <from CHANGELOG / package.json / pyproject.toml, or "not versioned">
- **Last release:** <date or "N/A">
- **Recent activity:** <one-line from top of CHANGELOG — e.g. "v2.4.1 — breaking auth change, active work on payments module">
- **CHANGELOG location:** <path or "not present">

---

## 3. Stack

| Layer            | Technology | Version |
|------------------|------------|---------|
| Language         |            |         |
| Runtime          |            |         |
| Framework        |            |         |
| Database         |            |         |
| Cache / Queue    |            |         |
| Package manager  |            |         |
| Test runner      |            |         |
| Linter/Formatter |            |         |
| CI               |            |         |
| Deploy target    |            |         |
| License          |            |         |

---

## 4. Top-Level Layout

> Full annotated tree → [TREE_SNAPSHOT.md](.cursor/context/TREE_SNAPSHOT.md)

    <root>/
    ├── <dir1>/          # <one-line purpose>
    │   ├── <subdir>/    # <purpose>
    │   └── <file>       # <purpose>
    ├── <dir2>/          # <purpose>
    └── <file>           # <purpose>

Annotate every directory and significant file at all levels.
Do not truncate — write the full annotated tree, however long it needs to be.

---

## 5. Entry Points

| Path | Type | Role |
|------|------|------|
| <path> | HTTP server / CLI / worker / WS server / SSE / frontend shell / library | <runtime role> |

---

## 6. API Contracts

| File | Format | Version | Top-level resources / operations |
|------|--------|---------|----------------------------------|
| <path> | OpenAPI / GraphQL / gRPC / AsyncAPI | <v> | <summary> |

N/A — no contract files found in this repo.   ← remove line if contracts exist

---

## 7. Module Map

> Detailed breakdown → [modules.md](.cursor/context/modules.md) *(medium/large projects)*

| Module | Path | Purpose | Depends On (internal) | Depends On (external) | Used By |
|--------|------|---------|-----------------------|-----------------------|---------|
|        |      |         |                       |                       |         |

---

## 8. Data Layer

> Full schema/migration details → [data_layer.md](.cursor/context/data_layer.md) *(if created)*

- **Engines:** <PostgreSQL 15, Redis 7, etc.>
- **ORM / query layer:** <Prisma 5 — schema at `prisma/schema.prisma`>
- **Migrations:** <location and command to run>
- **Caches:** <Redis — sessions + rate limiting>
- **Queues:** <BullMQ — queue names: emailQueue, exportQueue>

---

## 9. Configuration & Secrets

- **Config sources (priority order):** `.env.local` > `.env.<NODE_ENV>` > `.env`
- **.env.example present:** <yes | no — if no, required keys are undocumented>
- **All required keys (names only — NEVER values):**
  `DATABASE_URL`, `JWT_SECRET`, `STRIPE_SECRET_KEY`, `SENDGRID_API_KEY`
- **Optional keys:** <list or "not distinguished in .env.example">
- **Secret manager:** <none | AWS Secrets Manager | Vault | Doppler — at `src/config/secrets.ts`>
- **Feature flags:** <none | LaunchDarkly — client at `src/lib/flags.ts`>
- **Auth mechanism:** <NextAuth.js at `src/lib/auth.ts` | Passport JWT | session | OAuth2/OIDC | none>
- **Permission model:** <RBAC at `src/lib/permissions.ts` | ABAC | row-level security | none>

---

## 10. Build / Test / Deploy

### Commands

    # Install
    <cmd>

    # Local dev (required env: DATABASE_URL, ...)
    <cmd>

    # Unit tests
    <cmd>

    # Integration / e2e tests
    <cmd>

    # Production build
    <cmd>

    # Deploy
    <cmd or description>

- **Environments:** dev → staging → production
  Defined in: <.env files / GitHub Actions secrets / Vercel project settings>
- **CD trigger:** <push to main / manual / tag>

### Local Service Map (Docker Compose)

| Service | Role | Ports (host:container) | Volumes | Depends On |
|---------|------|------------------------|---------|------------|
| <name> | db / cache / queue / app / worker | <mapping> | <named vol> | <service> |

N/A — no docker-compose.yml in this repo.   ← remove line if compose exists

---

## 11. Infrastructure Map

> Full IaC / CI details → [infra.md](.cursor/context/infra.md) *(if created)*

- **Cloud provider(s):** <AWS / GCP / Azure / Cloudflare / self-hosted / none>
- **IaC tool:** <Terraform 1.6 — root at `terraform/` | Pulumi | CDK | none>
- **Environments in IaC:** <dev, staging, prod — at `terraform/environments/`>

### Cloud Resources

| Resource type | Name / ID | Environment | Provisioned by |
|---------------|-----------|-------------|----------------|
| <RDS PostgreSQL> | <app-db> | <prod, staging> | <terraform/modules/db/main.tf> |
| <S3 bucket> | <app-uploads> | <all> | <terraform/modules/storage/main.tf> |
| <ECS service> | <api> | <prod> | <terraform/modules/ecs/main.tf> |

### CI/CD Job Map

| Job | Trigger | Deploys to | Required secrets |
|-----|---------|------------|-----------------|
| <build> | <push to any branch> | <—> | <—> |
| <test> | <push to any branch> | <—> | <DATABASE_URL> |
| <deploy-staging> | <merge to main> | <staging> | <AWS_ACCESS_KEY_ID, ...> |
| <deploy-prod> | <manual / tag> | <production> | <AWS_ACCESS_KEY_ID, ...> |

### Serverless / Edge

| Service / Function | Runtime | Trigger | Region |
|--------------------|---------|---------|--------|
| <name> | <Node 20 / Python 3.12> | <HTTP / SQS / cron> | <us-east-1> |

N/A — no IaC or CI/CD pipeline files found.   ← remove line if infra exists

---

## 12. Architecture Decisions (ADRs)

- **ADR directory:** <path or "not present">

| #  | Title | Status | Decision summary |
|----|-------|--------|-----------------|
| 01 | <title> | accepted / deprecated / superseded | <one-line summary> |

N/A — no ADR directory found.   ← remove line if ADRs exist

---

## 13. Agent & Contributor Rules

| File | Key rules |
|------|----------|
| `CONTRIBUTING.md` | <branch naming, PR process, commit style, review expectations> |
| `AGENTS.md` | <agent task rules, tool permissions, workflow constraints for this repo> |
| `.cursorrules` | <key constraints from legacy Cursor rules file> |
| `CODEOWNERS` | <path patterns → owning teams, e.g. `src/payments/ @payments-team`> |
| `SECURITY.md` | <vulnerability disclosure process — note existence> |
| `.husky/` / `.pre-commit-config.yaml` | <pre-commit hooks enforced on this repo> |

Omit any row where the file does not exist in the repo.

---

## 14. Conventions

| Area | Convention |
|------|-----------|
| Naming | <kebab-case files, PascalCase components, camelCase variables> |
| Layering | <routes → controllers → services → repositories> |
| Error handling | <custom AppError class; HTTP errors via `src/lib/errors.ts`> |
| Logging | <Pino at `src/lib/logger.ts`; JSON in prod, pretty in dev> |
| Imports | <absolute from `src/`; no barrel re-exports in tests> |
| Testing | <unit tests co-located as `*.test.ts`; e2e in `tests/`> |
| Ownership | <CODEOWNERS present — `src/payments/ @payments-team`, etc. — or "not defined"> |
| Pre-commit | <hooks enforced: lint-staged, commitlint, etc. — or "none"> |

---

## 15. Workspace Rules In Effect

| Rule file | Applies when | Key constraints |
|-----------|-------------|----------------|
| `.cursor/rules/<n>.mdc` | <trigger condition> | <what it enforces> |

---

## 16. Known Unknowns

- <be honest and specific — never leave this section empty>
- Example: "Auth flow not fully traced — session store and token refresh location unknown"
- Example: "no .env.example — required config keys undocumented"
- Example: "ADR directory present but unpopulated — architectural decisions unknown"
- Example: "No IaC files found — cloud resource provisioning method unknown"

---

## 17. Suggested Deep Dives

- `<module or area>` — <reason, e.g. "payment service has 4 FIXMEs and zero tests">
- `<...>`

<!-- Files written this run: PROJECT_CONTEXT.md, TREE_SNAPSHOT.md, modules.md -->
```

---

## TREE_SNAPSHOT Template

> Written to `.cursor/context/TREE_SNAPSHOT.md` in Phase 1.7.

```markdown
# Working Tree Snapshot

> Generated: <ISO timestamp>  Git SHA: <sha or N/A>
> Tool: get-project-context  Mode: <mode>

## Root: <root_path>

<full tree output, indented>

## Monorepo Workspaces (if any)

| Workspace | Path | Type |
|-----------|------|------|
| ...       | ...  | ...  |

## Size Stats

- Total files (excluding patterns): <N>
- Total directories: <N>
- Estimated LOC: <N or "not computed">

## Project State (from CHANGELOG / release notes)

- Latest version: <version or N/A>
- Last release: <date or N/A>
- Recent activity: <one-line summary of last notable changes>
```

---

## Module Summary Block

> Written to `.cursor/context/.wip_context.md` during Phase 4 for each app module M.

```markdown
### Module: <name>
- **Path:** <relative path>
- **Purpose:** <one sentence>
- **Key Files:** <file1>, <file2>, ...
- **Public API:** <exported symbols or route prefixes>
- **Depends On (internal):** <module names>
- **Depends On (external):** <package names>
- **Used By:** <module names>
- **Integrations:** <third-party APIs / DBs / queues>
- **Known Risks:** <TODOs / FIXMEs / HACKs found>
```
