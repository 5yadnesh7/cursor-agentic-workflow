---
name: incident-commander
description: Runs live incident response end-to-end: declare and classify severity, coordinate communications, drive mitigation before fix, track actions and timeline, hand off to debugging for root cause, and produce a blameless postmortem with follow-ups. Use during outages, severe degradations, security incidents, data issues, or any "production is on fire" moment, and immediately after to produce the postmortem.
---

# Incident Commander

Coordinates live incidents with clear roles, crisp communication, mitigation-first bias, and a structured postmortem afterward. Optimizes for stopping user pain fast, then permanent fix, then learning.

## When To Use

- Production outage, severe latency/error spike, or broken core flow.
- Security incident (suspected breach, exposed secret, malicious traffic).
- Data incident (corruption, loss, leakage, bad migration).
- Major dependency failure impacting users.
- Immediately after any of the above to run the postmortem.

## Core Principles

1. Mitigate before you fully understand. Stop the bleeding first.
2. One incident commander at a time. No competing authority.
3. Communication is a first-class task, not an afterthought.
4. Write everything down as it happens; memory is not a source of truth.
5. Blameless. Focus on systems, signals, and defaults, not people.
6. A fix is not the end; follow-ups and prevention are.

## Roles

- **Incident Commander (IC)** — runs the incident, owns decisions, keeps focus.
- **Ops Lead** — drives technical mitigation and investigation.
- **Comms Lead** — owns internal and external updates.
- **Scribe** — maintains the live timeline and action log.
- **Subject-matter experts** — pulled in as needed (db, infra, security, app).

In small teams, one person may hold multiple roles, but IC and Ops Lead should be different people when possible.

## Severity Scale (Default; adapt to your org)

| Sev | Criteria | Target response |
|-----|----------|-----------------|
| SEV-1 | Major outage or data loss; user-visible broken flow. | Immediate, 24x7 page. |
| SEV-2 | Significant degradation; partial outage; elevated errors. | Fast response within business SLOs. |
| SEV-3 | Minor degradation; single-feature impact; workaround exists. | Best-effort response. |
| SEV-4 | Informational / near-miss. | Triage on next business day. |

## Workflow

```
Incident:
- [ ] Step 1: Detect and declare
- [ ] Step 2: Classify severity and assemble roles
- [ ] Step 3: Establish comms channel and cadence
- [ ] Step 4: Mitigate
- [ ] Step 5: Investigate and fix
- [ ] Step 6: Verify and resolve
- [ ] Step 7: Close the incident
- [ ] Step 8: Postmortem and follow-ups
```

### Step 1: Detect And Declare

- Confirm the signal is real (dashboard, alert, multiple reports).
- Declare an incident explicitly ("Declaring SEV-<X>: <one-line summary>").
- Open the incident record (ticket, channel, or doc) with: start time, detector, summary, initial severity.

### Step 2: Classify Severity And Assemble Roles

- Pick severity against the scale; escalate proactively if unsure.
- Name IC, Ops Lead, Comms Lead, Scribe.
- Invite the minimum set of SMEs needed; avoid crowding.

### Step 3: Establish Comms Channel And Cadence

- Single incident channel (chat room or bridge).
- Decide update cadence (SEV-1: every 15–30 min; SEV-2: every 30–60 min; adjust for rate of change).
- Draft the first internal status post using the template below.
- For customer-impacting incidents, align with the status-page owner.

### Step 4: Mitigate

Mitigation takes priority over root-cause analysis.

Common levers, in roughly decreasing preference:

- Roll back the most recent deploy or config change.
- Toggle a feature flag off.
- Shift traffic away from an unhealthy region, version, or tenant.
- Scale up a resource that is saturated.
- Failover to a healthy replica or cache.
- Apply a narrow hotfix patch.
- Reduce load (rate-limit, shed non-critical traffic, disable heavy features).

Record every action, who did it, exact time, and observed effect.

### Step 5: Investigate And Fix

Once users are safe, drive the root-cause investigation. Hand off to `debugging`:

- Provide IC timeline, symptoms, mitigations attempted, and evidence collected.
- `debugging` owns hypothesis testing and root cause.
- Keep the incident open until the fix is planned and tracked.

### Step 6: Verify And Resolve

- Confirm impact metrics are back to normal for at least one full monitoring window.
- Confirm error budgets and alerts are green.
- Check for secondary fallout (queues, backlogs, retries, stale caches).
- Communicate "mitigation holding; monitoring" before declaring resolved.

### Step 7: Close The Incident

- Announce resolution with time, impact summary, and what was done.
- Schedule the postmortem (SEV-1/2: within 3–5 business days).
- Archive channel logs, dashboards, and the timeline.
- Open follow-up tickets for every deferred action.

### Step 8: Postmortem And Follow-ups

Use the blameless template below. Coordinate with `doc-writer` and `debugging`.

## Templates

### Internal Status Update

```
[SEV-<X>] <short title>

- Started: <HH:MM TZ>
- Status: <investigating | mitigating | monitoring | resolved>
- Impact: <who / what is affected, quantified if possible>
- Current action: <what the team is doing now>
- Next update: <time>
- IC: <name>  Ops: <name>  Comms: <name>
```

### External / Status Page Update

Write plainly. No speculation. No internal jargon.

```
<Time> — We are investigating <observed impact>. Some users may experience <symptom>.
<Time> — We have identified a likely cause and are applying a mitigation.
<Time> — Services have recovered. We are monitoring and will share a full postmortem.
<Time> — The incident is fully resolved. A postmortem will be published by <date>.
```

### Live Timeline (Scribe)

```
Incident: <title>  |  Severity: SEV-<X>  |  IC: <name>

HH:MM — Detected via <signal>
HH:MM — Declared SEV-<X>; IC: <name>; Ops: <name>
HH:MM — Hypothesis: <X>; Action: <action>; Owner: <name>
HH:MM — Observed: <result>
HH:MM — Mitigation applied: <change>
HH:MM — Impact back to normal
HH:MM — Monitoring
HH:MM — Resolved
```

### Postmortem (Blameless)

```
# Postmortem: <incident title>

## Summary
- Severity: SEV-<X>
- Duration: <start> – <end> (<total>)
- User impact: <quantified>

## Timeline
<key events from the live timeline>

## Root Cause
<what actually went wrong, at the system level>

## Contributing Factors
- <missing guardrail, weak signal, unclear ownership, etc.>

## What Went Well
- <detections, decisions, mitigations that worked>

## What Went Poorly
- <gaps in detection, comms, process, knowledge>

## Action Items
| # | Action | Owner | Type (prevent / detect / respond) | Due |
|---|--------|-------|-----------------------------------|-----|
| 1 | ...    | ...   | prevent                           | ... |

## Lessons
- <generalizable learning, written without blame>
```

## Security And Data Incidents

- Treat as SEV-1 until proven otherwise.
- Preserve evidence (logs, images, memory, DB snapshots) before remediating.
- Limit communications to a need-to-know channel initially.
- Engage security, legal, and privacy owners per org policy.
- Do not speculate publicly about cause or attribution.

## Coordination

- Root-cause investigation → `debugging`.
- Infra mitigation (rollback, traffic shift, scaling, rollout fixes) → `devops`.
- Data-level fixes (bad migration, corruption, backfill) → `db-engineer`.
- Release-gating changes following the incident → `release-manager`.
- Postmortem write-up polish → `doc-writer`.
- Test coverage that would have caught it → `tester`.

## Quality Bar

- Severity was declared promptly and matched impact.
- Users saw a mitigation before a full fix.
- A written timeline exists and matches reality.
- Internal and external comms were honest, specific, and on cadence.
- Every action item has an owner and due date.
- The postmortem is blameless and produces systemic changes, not individual blame.

## Anti-Patterns

- Multiple people acting as IC at once.
- Debating root cause while users are still impacted.
- Silent war rooms with no updates for 45+ minutes.
- Declaring resolved before monitoring a full window.
- Postmortems that list only "be more careful" as actions.
- Attributing cause to a single person rather than the system that allowed it.
- Skipping postmortems for SEV-1/2 "because everyone already knows what happened".
