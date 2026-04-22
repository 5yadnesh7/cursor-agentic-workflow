---
name: workflow-incident
description: Live incident response workflow. incident-commander runs the bridge, coordinates communication and mitigation, then drives a blameless postmortem. debugging finds root cause, owner skills ship the fix, release-manager ships the resolution, doc-writer publishes the post. Use for active outages, SLO breaches, or production anomalies.
skills_used:
  - prompt-enhancer
  - validate-prompt-intent
  - incident-commander
  - debugging
  - dev-engineer
  - db-engineer
  - devops
  - tester
  - release-manager
  - doc-writer
gates:
  - mitigation-gate
  - rollback-gate
  - resolution-gate
  - postmortem-gate
  - destructive-gate
next_workflows:
  - workflow-fix
  - workflow-release
  - workflow-audit
---

# Workflow: workflow-incident

Stop the bleeding, then learn from it.

## Chat Rules

Follow the Chat Rules in `workflow-main.md`: no narration, no session files, one decision per turn with a reply menu on the last line. All state lives in the conversation. Incident decisions are captured in the final postmortem artifact at Phase E; in-flight, the chat transcript IS the record.

## Step 0 — Severity + Prompt Normalization

Ask (or infer from prompt): `SEV1 | SEV2 | SEV3`.

- **SEV1 / SEV2 (live outage)**: **skip** `prompt-enhancer` / `validate-prompt-intent`. Do not slow down mitigation. `incident-commander` captures a one-line incident statement in chat (what's broken, since when, who's affected) and moves to Phase A.
- **SEV3 or drills / tabletop / postmortem-only runs**: if dispatched from `workflow-main` the router already enhanced and validated. If invoked directly via `/workflow-incident`, run **Step 1 of `workflow-main.md` verbatim** (the tightly-coupled `prompt-enhancer` ⇄ `validate-prompt-intent` loop: every enhancer run is immediately followed by a validator run; only `approve` exits the loop; user `refine <feedback>` re-enters at 1a) before Phase A. Use a cap of 2 on validator-driven refines instead of 3 because incident triage is time-sensitive.

## Phase A — Stabilize

Driven by `incident-commander`:

1. Acknowledge and timestamp the incident.
2. Assemble context: what broke, since when, who's affected, known recent changes.
3. Open a communication channel (user-defined); announce status.
4. Propose mitigation options:
   - Rollback to last known good.
   - Feature flag off.
   - Traffic shift / rate limit.
   - Hotfix path.

**mitigation-gate** (mandatory in SEV3; in SEV1/SEV2 the commander may act and log retroactively with user sign-off as soon as possible).

If mitigation = rollback:
- `devops` executes rollback.
- **rollback-gate** (mandatory before triggering in SEV3; logged immediately in SEV1/SEV2).

If mitigation = hotfix:
- `debugging` finds fastest safe fix.
- Owner skill (`dev-engineer` / `db-engineer` / `devops`) applies.
- `tester` runs smoke only.
- `release-manager` + `devops` ship via the hotfix path; **resolution-gate** before going live.

Any destructive op on data → **destructive-gate** even in SEV1.

## Phase B — Verify Resolution

1. Confirm metrics recovered; SLOs returning to baseline.
2. `tester` verifies affected user journeys.
3. `incident-commander` announces resolution timestamp and declares the incident over.
4. **resolution-gate** (mandatory).

## Phase C — Root Cause

1. `debugging` completes full root-cause analysis (no shortcuts; now that pressure is off).
2. Evidence, timeline, contributing factors, and latent risks returned to the workflow for inclusion in the postmortem at Phase E.

## Phase D — Durable Fix

If Phase A was a temporary mitigation, open the true fix as a follow-up:

- Route to `workflow-fix` carrying the RCA forward.
- Afterwards, route to `workflow-release` to ship.

## Phase E — Postmortem

`incident-commander` + `doc-writer` produce a blameless postmortem:

- Summary, impact, timeline, detection, response, root cause, contributing factors, what went well, what didn't, action items with owners + due dates.

**postmortem-gate** (mandatory).

Stored at the repo's postmortem convention path (ask the user at the gate; typical: `docs/postmortems/<date>-<slug>.md`).

## Phase F — Hardening Plan

Derive action items:

- Monitoring / alerting gaps → `workflow-infra`.
- Test coverage gaps → `workflow-fix` / `workflow-feature`.
- Systemic risks → `workflow-audit`.

## Phase G — Close Out

Send one final chat turn summarizing the incident: severity, timeline, mitigation used, resolution time, root cause, postmortem path, and hardening follow-ups dispatched.

## Safety Rules

- Mitigation first, root cause second.
- In SEV1/SEV2, the commander may take emergency action before the mitigation-gate; they must state the action in chat immediately and seek user sign-off as soon as the user is available. This is the only mandated gate that can be deferred — never skip stating it.
- Destructive ops always hold their gate even in SEV1.
- Never skip the postmortem, even for "quick" incidents.
- Postmortems are blameless. `doc-writer` rewrites personalizing language.
