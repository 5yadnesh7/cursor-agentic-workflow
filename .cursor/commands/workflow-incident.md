# Workflow: Incident
Live incident response. incident-commander runs the bridge, coordinates comms and mitigation, then drives a blameless postmortem. debugging finds root cause, owner skills ship the fix, release-manager ships the resolution, doc-writer publishes the post.

## What This Does

Runs `.cursor/workflows/workflow-incident.md`: announce INCIDENT MODE → stabilize (mitigate / rollback / hotfix) → verify resolution → root cause → durable fix via `/workflow-fix` + `/workflow-release` → blameless postmortem → hardening plan.

## Steps

1. Open `.cursor/workflows/workflow-incident.md` and follow every phase.
2. Obey the **Chat Rules** referenced at the top of that workflow: no narration, no session files, one decision per turn with a reply menu on the last line. The chat transcript IS the incident record until the postmortem is written.
3. Set severity: `SEV1 | SEV2 | SEV3`. In SEV1 / SEV2, skip prompt-enhance/validate and go straight to mitigation.
4. Pause at mandatory gates: mitigation, rollback, resolution, postmortem, destructive.
5. In SEV1 / SEV2, emergency action may precede the gate; state it in chat immediately and seek retroactive sign-off as soon as the user is available.

## Inputs

Use any text after `/workflow-incident` to state severity, symptoms, known recent changes, and anything already tried. Ask for missing info — but do not block mitigation on it.

## Must Not

- Narrate bookkeeping or create session files.
- Skip the postmortem, even for "quick" incidents.
- Waive the destructive-gate at any severity.
- Let personalizing language into the postmortem — blameless only.
