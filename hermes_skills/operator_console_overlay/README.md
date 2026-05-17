# Skill: `operator_console_overlay`

> **Primary callers:** Ω — TRIBUNE (Human Interface Layer), Ο — ECHO (Analytics & Reporting). Visual production owned by Ι — AETHER when needed.
> **Purpose:** A WebSocket event-bus contract that surfaces every internal Council/Validation/Governance event into whatever frontend the deployment uses. The 25 agents are not opaque to the operator; they narrate themselves through this overlay.

## Why It Exists

TRIBUNE owns the human-agent boundary; ECHO already produces analytics. What was missing was a *standardized event bus* so the internals (debate transcripts, gauntlet stage results, governance heartbeat status, knowledge-vault ingestion events) flow to the operator console in real time without each frontend having to integrate against each producing agent individually.

## Event Schema

Every event published to the overlay bus has this shape:

```yaml
event_id: <uuid>
ts: <iso8601>
producer: <agent or skill name>     # e.g., "MODULUS", "council_debate", "WARDEN", "validation_gauntlet"
kind: <enum>                         # see below
severity: info | notice | warning | critical
payload: <structured event-specific data>
links:
  briefing_id: <optional>
  artifact_id: <optional>
  audit_ref: <path or DB id>
```

## Event Kinds

| `kind` | Producer | When Fired |
|---|---|---|
| `council.briefing_opened` | DOMINION/MODULUS | A `council_debate` session begins. |
| `council.position` | each seated agent | An agent submits its Position. |
| `council.rebuttal` | each seated agent | An agent submits a Rebuttal. |
| `council.consensus_signal` | council_debate | Consensus reached. |
| `council.consensus_rejection` | council_debate | Consensus failed. |
| `gauntlet.stage_started` | validation_gauntlet | A stage (R1–R8) begins. |
| `gauntlet.stage_passed` | validation_gauntlet | Stage passed. |
| `gauntlet.stage_failed` | validation_gauntlet | Stage failed; gauntlet halted. |
| `gauntlet.promoted` | validation_gauntlet | Artifact reached R8 (live). |
| `governance.heartbeat_ok` | guardian | 60s heartbeat verified policy hash. |
| `governance.hash_mismatch` | guardian | Critical — kill-switch fired. |
| `governance.kill_switch` | guardian | System-wide halt issued. |
| `vault.ingested` | LOOM | New knowledge artifact processed. |
| `vault.skill_proposed` | hermes-agent | New skill draft awaiting gauntlet. |
| `vault.skill_promoted` | validation_gauntlet | Skill reached `promoted_skills/`. |
| `adapter.stale` | NEXUS | An external adapter went stale. |
| `agent.action_executed` | any | An irreversible action committed. |
| `agent.error` | any | An agent emitted a named error. |

## Frontend Reference Wiring

```
overlay_bus (WebSocket on ws://<deployment>/overlay)
   ├── TRIBUNE subscribes (operator notifications)
   ├── ECHO subscribes (analytics/reporting)
   ├── AETHER subscribes (visual production for high-severity events)
   └── any frontend subscribes (React, Grafana, Discord bot, Slack bot, etc.)
```

The overlay does not require a specific UI; the contract is the protocol. The reference deployment in the LEVIATHAN repo wires this bus to a Bloomberg-style React/TypeScript terminal, but any consumer that speaks WebSocket and the schema above works.

## Hard Rules

- Events are append-only; once published they cannot be edited (replay/audit integrity).
- Critical-severity events (`governance.hash_mismatch`, `governance.kill_switch`, `gauntlet.stage_failed` for live artifacts) are mirrored to a durable log SENTINEL monitors regardless of subscriber state.
- Every event has an `audit_ref` pointing at the canonical evidence; the overlay is the *narration*, not the source of truth.

## Reference Instantiation
The trading-specialized BB-Terminal frontend (Bloomberg-style React/TypeScript console with Council debate transcript panel, gauntlet stage tracker, governance heartbeat indicator, and vault ingest log) lives in [`ocean824/LEVIATHAN-AI-BLACKWEALTHCAPITAL/hermes_pantheon/adapters/proteus_bbterminal.md`](https://github.com/ocean824/LEVIATHAN-AI-BLACKWEALTHCAPITAL/blob/main/hermes_pantheon/adapters/proteus_bbterminal.md).
