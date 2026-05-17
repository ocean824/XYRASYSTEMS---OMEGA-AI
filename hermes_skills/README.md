# Hermes Skills — Additive Capability Plumbing for ØMEGA AI

> **What this is:** Six domain-agnostic *skills* (plumbing — capabilities, tools, and logic) that ØMEGA's canonical 25 agents call. **No new agents.** This module slots into the existing architecture as additional callable capabilities for DOMINION and the 24 subordinate agents.
> **What this is not:** A new pantheon. A trading system. The trading-specific 14-agent oceanic Pantheon lives in [`ocean824/LEVIATHAN-AI-BLACKWEALTHCAPITAL`](https://github.com/ocean824/LEVIATHAN-AI-BLACKWEALTHCAPITAL/tree/main/hermes_pantheon) and is the canonical reference deployment of these same skills, fully specialized to one vertical.
> **Strictly additive.** Replaces, renames, or overrides nothing in `docs/`, `architectures/`, `prompts/`, `security/`, `CODESPRING-MASTER-INSTRUCTIONS.md`, or the canonical 25-agent roster.

---

## How These Skills Plug Into the Canonical 25

The roster of 25 is authoritative (`docs/master-agent-index.md`). These skills are tools those agents use — not replacements, not parallels.

| Skill | Primary Caller(s) Among the 25 | What the Skill Adds to Their Existing Job |
|---|---|---|
| `council_debate` | **Ø — DOMINION** (Supreme), **Κ — MODULUS** (Decision Engine), **Ψ — MIRRORBACK** (Self-Reflection), with **G0DM0D3** filling the red-team seat | Formal Briefing → Position → Rebuttal → Consensus protocol so MODULUS produces *defended* decisions and DOMINION can route on weighted multi-agent consensus instead of single-agent verdicts. The 5 Meta-Agents (LOOM, WARDEN, SIGMA, MAESTRO, PHANTOM) — DOMINION's Inner Council — are the natural seat composition. |
| `validation_gauntlet` | **Γ — SIGMA** (Optimization & Self-Audit), **Ζ — ARCANE** (Engineering — drafts the candidate code/asset), **Θ — QUANTUM** (in trading deployments) | Standardized 8-stage Research → Validate → Implement contract: initial validation, Monte Carlo, walk-forward, regime/segment testing, peer cross-reference, paper/dry-run, semi-auto with operator approval, full-auto with continuous audit. SIGMA's existing "track performance, improve autonomously" mandate gets a formal pipeline. |
| `sovereign_governance_lock` | **Β — WARDEN** (Security & Risk Control), with **Φ — JURIS** (Legal & Compliance) auditing the policy file | OS-level immutable policy file pattern with SHA-256 master hash + heartbeat verification + standalone guardian process. Closes the "agent edits its own safety rules" failure mode by making constraints physically unmodifiable from inside the system. Extends — does not replace — `security/guardrails-framework.md`. |
| `knowledge_vault` | **Α — LOOM** (Memory & Continuity), **Υ — ARCHIVE** (Data Storage), with the open-source `hermes-agent` (Nous Research) writing reusable skills | Watched directory + ingestion pipeline that routes new domain knowledge (PDFs, transcripts, screenshots, Markdown) to the appropriate agent's training corpus. LOOM holds the long-term context; ARCHIVE persists the structured ingest index; `hermes-agent` writes the reusable skill artifacts that emerge from successful tasks. |
| `external_context_adapter` | **Λ — NEXUS** (Integration Hub), with **Ε — PHANTOM** (Competitive Intelligence) and **Χ — SENTINEL** (Monitoring & Alerts) as common downstream consumers | Generic adapter contract for pulling external context (news, threat-intel, regulatory feeds, market data, competitor signals, etc.) into PRIME-orchestrated task envelopes. Includes `stale=true` flag handling and conservative-mode fallback for when upstream connections degrade. |
| `operator_console_overlay` | **Ω — TRIBUNE** (Human Interface Layer), **Ο — ECHO** (Analytics & Reporting), with **Ι — AETHER** for visual production when needed | WebSocket event-bus contract that surfaces internal Council/Validation/Governance events to whatever frontend the deployment uses. TRIBUNE owns the human-agent boundary; ECHO supplies the analytics overlay. The reference instantiation in the LEVIATHAN repo wires these events to a Bloomberg-style React terminal — but any UI works. |

The Tier I/II/III sub-agent layout, the 5 Meta-Agents (DOMINION's Inner Council), the canonical PRIME/DOMINION naming rule from the control-plane addendum, the ULTRAPLAN/Coordinator/Swarm patterns, and every other ØMEGA construct stay exactly as defined in the existing docs.

---

## The Hermes Logic in One Sentence

When DOMINION needs to make a high-stakes decision, **MODULUS** convenes a `council_debate` over the 5 Meta-Agents (with G0DM0D3 in the red-team seat); if consensus emerges, **SIGMA** runs the proposal through a `validation_gauntlet`; **WARDEN** verifies it does not violate the immutable `sovereign_governance_lock`; **NEXUS** supplies fresh `external_context_adapter` snapshots throughout; **LOOM/ARCHIVE** persist everything the system learns via the `knowledge_vault`; and **TRIBUNE/ECHO** surface the entire chain to the operator through the `operator_console_overlay`.

---

## Folder Layout

```
hermes_skills/
├── README.md                          ← this file
├── council_debate/
│   └── README.md                      ← Briefing → Position → Rebuttal → Consensus contract
├── validation_gauntlet/
│   └── README.md                      ← 8-stage Research → Validate → Implement contract
├── sovereign_governance_lock/
│   └── README.md                      ← immutable policy file + guardian daemon pattern
├── knowledge_vault/
│   └── README.md                      ← watched directory + LOOM/ARCHIVE ingest workflow
├── external_context_adapter/
│   └── README.md                      ← adapter contract for upstream feeds
└── operator_console_overlay/
    └── README.md                      ← WebSocket event-bus contract for frontends
```

---

## How DOMINION Calls a Skill

PRIME / DOMINION invokes a Hermes skill the same way it invokes any other capability — through the existing task envelope, MCP tool registration, or harness call. Skills are pure functions of their inputs and write only to their own audit logs and the canonical policy/audit store. They never spawn new persistent agents, never claim a slot in the canonical 25.

```
DOMINION → opens task envelope { skill: "council_debate", briefing: {...}, agents: [LOOM, WARDEN, SIGMA, MAESTRO, PHANTOM] }
         → routes to skill handler
         → handler returns { consensus: true|false, signal: {...}, transcript: [...] }
         → DOMINION records to canonical audit store, then routes downstream (e.g., to validation_gauntlet)
```

---

## Anti-Pattern Discipline (inherited from ØMEGA)

Every skill honors ØMEGA's existing engineering anti-patterns: atomic writes for shared state, no silent skips (every failure logs a named reason), no speculative writes (entities never marked validated without the artifact already on disk), diagnostic-first on repeated failures, and failure-cache tiering for adapters whose upstream context goes stale.

---

## TLDR

Six new skills the canonical 25 agents can call. They formalize debate, validation, immutable governance, knowledge ingestion, external context, and operator overlay as reusable plumbing. Each skill explicitly maps to existing ØMEGA agents — no parallel pantheon is introduced. The trading-specific instantiation lives in a separate repo. ØMEGA's existing architecture is untouched.
