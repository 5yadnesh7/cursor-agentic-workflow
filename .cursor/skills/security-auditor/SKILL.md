---
name: security-auditor
description: Performs application and infrastructure security reviews using threat modeling (STRIDE), authn/authz analysis, input/output validation checks, secret and key management audit, dependency and supply-chain scanning, and cloud/IaC misconfiguration review. Produces severity-tagged findings with concrete remediations. Use when the user asks for a security review/audit, before releases touching auth or PII, after a suspected incident, or when reviewer / dev-engineer / devops flags a security concern.
---

# Security Auditor

Specialist security review skill. Grounded in project context, produces a structured audit covering threat model, application code, data handling, infra, and supply chain, with severity-tagged findings and fixes.

## When To Use

- Explicit request for a security review or audit.
- Pre-release checks on changes to auth, payments, PII, or public endpoints.
- After a suspected security incident, alongside `incident-commander`.
- When `reviewer` flags security `concern` or `blocker` findings.

## Inputs

- `scope`: feature, service, repo, or infra stack.
- `change` (optional): PR or diff under review.
- `compliance` (optional): SOC2, ISO, PCI, HIPAA, GDPR, DPDP, etc.
- `context_path` (default `.cursor/context/PROJECT_CONTEXT.md`).

## Prerequisite: Project Context

Run the Context Gate; invoke `get-project-context` if missing or stale (especially for auth, data flow, and secret handling sections).

## Core Principles

1. Least privilege, always.
2. Validate at trust boundaries; sanitize on output, too.
3. Secrets never live in code, logs, images, or tickets.
4. Defense in depth; no single control is relied upon alone.
5. Fail closed on auth and authz paths.
6. Log security-relevant events; never log secrets or PII.
7. Depend on vetted, updated third parties; pin and scan.

## Workflow

```
Security Audit:
- [ ] Step 1: Ensure project context
- [ ] Step 2: Define scope and assets
- [ ] Step 3: Threat model (STRIDE)
- [ ] Step 4: Application review
- [ ] Step 5: Data and secrets review
- [ ] Step 6: Infra and cloud review
- [ ] Step 7: Dependency and supply-chain review
- [ ] Step 8: Logging, monitoring, and response review
- [ ] Step 9: Compliance crosswalk (if applicable)
- [ ] Step 10: Report with severities and fixes
```

### Step 1: Ensure Project Context

Confirm `PROJECT_CONTEXT.md` covers the target area; refresh if not.

### Step 2: Scope And Assets

List:

- Assets (data classes, keys, tokens, endpoints, services).
- Actors (end users, admins, services, attackers, third parties).
- Trust boundaries (public → edge → app → data → third party).

### Step 3: Threat Model (STRIDE)

For each asset and boundary, enumerate:

| Category | Ask |
|----------|-----|
| Spoofing | Can an attacker impersonate a user, service, or host? |
| Tampering | Can data in transit or at rest be modified? |
| Repudiation | Can actions be denied without audit evidence? |
| Information disclosure | Can secrets or PII leak via logs, errors, side channels? |
| Denial of service | Can a component be overwhelmed or starved? |
| Elevation of privilege | Can a user escape their role or tenant? |

Record threats worth tracking. Skip theoretical ones that don't apply.

### Step 4: Application Review

Check for:

- Authentication: strong password handling, MFA support, session lifetime, token rotation, replay protection.
- Authorization: centralized checks, tenant isolation, object-level authz, no "security through obscurity".
- Input validation: at every trust boundary; safe deserialization; file upload limits; strong schema.
- Output handling: proper encoding for HTML, SQL, shell, URLs; content-type correctness.
- Injection classes: SQLi, NoSQLi, command injection, SSRF, XXE, template injection, prototype pollution.
- XSS defenses: CSP, sanitization, framework escaping; no `innerHTML` on untrusted input.
- CSRF: token or SameSite cookie strategy; safe defaults for mutating endpoints.
- Crypto: no home-grown crypto; strong algorithms; key rotation; proper randomness; constant-time comparisons for secrets.
- Error handling: no stack traces or internal details leaked to users.
- Rate limiting and abuse protection on sensitive endpoints.

### Step 5: Data And Secrets

- PII and sensitive data inventory; encryption at rest and in transit.
- Access control on datastores; least-privilege DB users per service.
- Backups: encrypted, access-controlled, restore-tested.
- Data retention and deletion; subject-access and deletion handling.
- Secrets: stored in a secret manager; never in repo, env files committed, images, or logs.
- Key management: rotation policy, separation of duties, access audit.

### Step 6: Infra And Cloud

- IAM policies: no `*:*`, no wildcard principals where not required.
- Network: private-by-default; public exposure justified and minimized.
- TLS everywhere; HSTS; modern ciphers; cert rotation automated.
- Containers: non-root, minimal base images, pinned, scanned; read-only filesystems where possible.
- Kubernetes: namespaces with network policies; limited RBAC; pod security standards.
- CI/CD: OIDC for cloud access (no static keys); required reviews; artifact signing; SBOM.
- IaC: policy checks (OPA/Conftest, Sentinel); plan review required for prod.

### Step 7: Dependency And Supply Chain

- Lockfiles present; versions pinned.
- Known-vulnerability scanning (OSV, GHSA, provider scanners).
- License review where applicable.
- Provenance: trusted sources, signed artifacts, SLSA-style build integrity where available.
- Typosquat / malicious package awareness for new dependencies.

### Step 8: Logging, Monitoring, And Response

- Authn/authz failures logged with correlation IDs.
- Alerts on abuse patterns (brute force, token misuse, egress anomalies).
- No secrets or PII in logs.
- Audit trails for sensitive admin actions.
- Runbook exists for common security alerts; ties to `incident-commander`.

### Step 9: Compliance Crosswalk (If Applicable)

For the named framework, list the controls touched and evidence pointers. Do not fabricate compliance claims. Flag gaps honestly.

### Step 10: Report

```
# Security Audit: <scope>

## Context Used
- PROJECT_CONTEXT.md (last updated: <ts>)

## Threat Model Highlights
- <threat> → <current control> → <gap / risk>

## Findings
### 🔴 Critical
- **<area>** — <issue> — <impact> — **Fix:** <concrete remediation>

### 🟠 High
- **<area>** — <issue> — <impact> — **Fix:** ...

### 🟡 Medium
- ...

### 🟢 Low / Informational
- ...

## Positive Controls
- <what the system does well>

## Compliance Notes (if applicable)
- <control> — <status> — <evidence>

## Recommended Next Steps
1. <prioritized action>
2. <prioritized action>
```

## Coordination

- Code-level remediations → `dev-engineer`.
- Infra remediations → `devops`.
- Data / schema remediations → `db-engineer`.
- Live incidents → `incident-commander`.
- Regression tests for security fixes → `tester`.
- Policy / ADR capture → `doc-writer`.

## Severity Guidance

| Severity | Meaning |
|----------|---------|
| 🔴 Critical | Active or near-term exploitability with high impact (RCE, auth bypass, secret exposure). Fix now. |
| 🟠 High | Serious weakness likely to be exploited; fix before next release. |
| 🟡 Medium | Meaningful risk under realistic scenarios; fix in a planned window. |
| 🟢 Low / Info | Hardening or hygiene; fix opportunistically. |

## Quality Bar

- Every finding names the asset, the threat, the impact, and a concrete fix.
- No theoretical findings without a realistic attacker model.
- Severity reflects real-world risk, not aspirational standards.
- Compliance statements are accurate and evidenced.
- Remediations name the skill or owner who should implement them.

## Anti-Patterns

- Scanner-output dumps without triage.
- "Use HTTPS" style non-findings.
- Hardcoded secrets "just for testing".
- Relying on obscurity (hidden endpoints, custom encodings) as security.
- Blocking releases on low-severity findings while ignoring criticals.
- Writing your own crypto.
