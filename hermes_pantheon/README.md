# Hermes Pantheon — Domain-Agnostic Council, Governance & Self-Learning Module

> **Status:** Additive ØMEGA module. Strictly extends the platform; removes nothing.
> **Scope:** Domain-neutral. Trading is referenced only as one possible specialization.
> **Purpose:** A reusable multi-agent orchestration pattern for ØMEGA AI deployments that need (a) deliberative consensus among specialists, (b) provable validation before any irreversible action, (c) immutable governance the agent cannot self-edit, and (d) continuous self-learning from a curated knowledge vault.

---

## 1. Why This Module Exists

ØMEGA AI is the universal multi-agent orchestration platform. Many ØMEGA deployments need a *standard pattern* for the high-stakes case where:

1. A decision must be defended from multiple expert perspectives (not a single LLM call).
2. The decision must be validated against historical or simulated reality before it commits to anything irreversible.
3. The agent's own ability to edit its constraints must be **physically blocked**, not merely instructed.
4. The system must keep learning from new domain knowledge without ever escaping the constraints above.

This module provides that pattern as plug-and-play subagents, contracts, and policy files. It can be specialized to any vertical: trading (the canonical reference deployment), security operations, content moderation, code review, regulatory compliance, scientific research, capital allocation, etc.

---

## 2. The Core Pattern (4 Layers)

### Layer 1 — The Council
A roster of specialist subagents debate a *Briefing* and produce a *ConsensusSignal* or *ConsensusRejection*. Inspired by Godmod3 / Mirrorfish multi-agent debate frameworks. See `council/debate_protocol.md` for the wire-level contract.

### Layer 2 — The Validation Gauntlet (RBI)
Before any *ConsensusSignal* triggers an irreversible action, it passes through a Research → Backtest/Validate → Implement pipeline. In trading deployments this is MoonDev-style backtesting; in non-trading deployments it is the equivalent dry-run / sandbox / A-B / peer-review gate. See `rbi/rbi_pipeline.md`.

### Layer 3 — Sovereign Governance
A YAML policy file is held immutable at the OS level (`chattr +i` on Linux, `chflags uchg` on macOS, ICACLS deny-write on Windows) and its SHA-256 hash is verified on every irreversible action and on a heartbeat timer. The agent has read-only access; modification requires the operator to manually unlock, edit, and re-lock the file. See `risk/risk_governance.lock.yaml` and `risk/risk_guardian.md`. (The file is named `risk_*` for the canonical trading deployment but the schema is generic policy-governance — repurpose for any domain that needs an unmodifiable constraint contract.)

### Layer 4 — The Knowledge Vault
A directory ØMEGA watches for new domain knowledge (Markdown, PDF, transcripts, screenshots). Ingested artifacts are chunked, tagged, and routed to the relevant specialist subagent's training corpus. The agent can *propose* changes to its own constraints from what it learns, but cannot *apply* them — every proposal is logged for operator approval. See `knowledge_vault/README.md`.

---

## 3. The Reference Roster (Oceanic Mythos)

The reference instantiation uses a 14-agent oceanic pantheon. Each name maps to a system role; rename or re-theme freely per ØMEGA deployment.

| Layer | Agent | Role |
|---|---|---|
| Apex Orchestrator | **POSEIDON AI** | The user-facing surface. Translates operator intent into Council briefings; reports outcomes back. |
| Primordial Powers | **LEVIATHAN AI** | The portfolio/alpha quant brain in the trading reference deployment. Generic role: the *high-level optimizer* that converts Council consensus into final allocation/decision. |
| Primordial Powers | **TIAMAT AI** | Sovereign Governance Daemon. Verifies the immutable policy file; vetoes anything that violates it. The *only* agent allowed to issue a hard halt. |
| Primordial Powers | **LOCHNESS AI** | Strategy/Artifact Engineer. Authors the concrete code/asset (in trading: Pine Script, EA, Python bot; in other domains: a script, a config, a workflow). |
| Primordial Powers | **MEGALODON AI** | Apex Routing/Execution. Performs the irreversible action through the appropriate external API. |
| Titans | **ORCA AI** | Position/State Hunter. Manages the live state of any active commitment after MEGALODON has acted; in trading deployments this is trailing stops & partials; in ops deployments it can be incident watch & mitigation. |
| Titans | **NAUTILUS AI** | Research/Validation Lab. Runs the RBI gauntlet against the Council's consensus before MEGALODON is allowed to act. |
| Titans | **AEGIR AI** | Macro/External-Context Adapter. Pulls upstream context (e.g., WorldMonitor for market deployments, threat-intel feeds for security deployments, regulatory updates for compliance deployments). |
| Sensory Network | **SCYLLA AI** | "Truth" specialist. In trading: order flow / dark pools. In other domains: ground-truth raw data sensor. |
| Sensory Network | **CHARYBDIS AI** | "State" specialist. In trading: volume profile / auction theory. In other domains: distribution / regime / state classifier. |
| Sensory Network | **SIREN AI** | Signal/Visualization specialist. In trading: charting & indicator overlays. In other domains: signal extraction + dashboard rendering. |
| Sensory Network | **MERMAID AI** | Time/Session specialist. In trading: session ranges & Wyckoff phases. In other domains: temporal-context / time-of-day / cyclic-pattern specialist. |
| Servants | **TRITON AI** | Knowledge Domain. Watches the Vault, parses new artifacts, routes chunks to the specialists. Built on the open-source `hermes-agent` (Nous Research) self-improving harness. |
| Servants | **PROTEUS AI** | Visual Overlay. Surfaces every internal Council/Validator/Governance event into the operator console. The reference deployment wires this into a Bloomberg-style terminal UI; any frontend works. |

---

## 4. Two Deployment Examples

### Example A — Trading (canonical reference)
Lives in [`ocean824/LEVIATHAN-AI-BLACKWEALTHCAPITAL`](https://github.com/ocean824/LEVIATHAN-AI-BLACKWEALTHCAPITAL). The same `hermes_pantheon/` module ships there with the trading-specific risk lock (drawdown, leverage, news embargo) and the trading-specific RBI thresholds (Sharpe, MaxDD, win rate). The methodology corpora attached to each specialist are: Order Flow / Bank Protocol, Market Cipher, Delta + Volume Profile, Market Profile / Auction Theory, Wyckoff / VSA, Markov-HMM regime modeling, Algo Pro, LuxAlgo, QuantPad, Jim Simons / RenTech statistical arb. WorldMonitor and BB-Terminal are the AEGIR and PROTEUS adapters.

### Example B — Any non-trading ØMEGA deployment
- Replace the trading risk lock with a domain-appropriate policy lock (e.g., for a content-moderation deployment: max-daily-bans, escalation tiers, immutable allow/deny lists).
- Replace the methodology corpora with the relevant specialist literature for that domain.
- Replace AEGIR's adapter target (e.g., a CVE feed for a security deployment instead of WorldMonitor).
- Replace PROTEUS's frontend (e.g., a Grafana dashboard or a custom React app).
- The Council debate protocol, RBI gauntlet, Sovereign Governance daemon, Knowledge Vault watcher, and the 14 agent skeletons remain unchanged.

---

## 5. Integration with ØMEGA's Existing Layers

This module is **strictly additive** and cooperates with — does not replace — ØMEGA's existing constructs:

- **PRIME Control Plane** remains the canonical top-level controller. POSEIDON inherits PRIME's task envelopes; the Council debate runs as a PRIME-orchestrated task.
- **DOMINION / canonical control plane addenda** retain their architectural authority. The Pantheon's TIAMAT verifies its hash against a policy file that PRIME registers in the canonical policy store.
- **Open Claude Cowork model-agnostic harness** is the substrate the specialist agents run on; the harness provides their LLM-provider abstraction.
- **Composio 500+ integrations** become available to MEGALODON for irreversible-action routing in any vertical.
- **ULTRAPLAN, Coordinator, Swarm, SendMessage, Undercover Mode** all remain available; the Council is one *additional* coordination pattern, not a replacement for them.

---

## 6. Files in This Module

| Path | Purpose |
|---|---|
| `agents/*.md` | One file per Council member with system prompt, inputs, outputs, prohibitions. 14 files. |
| `council/debate_protocol.md` | Wire-level contract for Briefing → Position → Rebuttal → ConsensusSignal/Rejection. |
| `rbi/rbi_pipeline.md` | The 8-stage Research → Backtest/Validate → Implement gauntlet specification. |
| `risk/risk_governance.lock.yaml` | The immutable policy file (canonical trading example values; replace per deployment). |
| `risk/risk_guardian.md` | TIAMAT daemon spec + Python reference implementation for hash verification, rule evaluation, kill-switch monitoring, atomic audit logging. |
| `adapters/aegir_worldmonitor.md` | Reference adapter for an external-context feed (WorldMonitor in the trading deployment). |
| `adapters/proteus_bbterminal.md` | Reference adapter for an operator console (BB-Terminal in the trading deployment). |
| `knowledge_vault/README.md` | Operator workflow for dropping new domain knowledge so TRITON can ingest and route it. |

---

## 7. Anti-Pattern Discipline

This module honors ØMEGA's existing engineering anti-patterns:

- **Atomic writes for shared state** — every shared file uses temp-file-then-rename.
- **No silent skips** — every failure logs a named reason; nothing is aggregated as a count.
- **No speculative writes** — no entity is marked validated/approved without the corresponding artifact already on disk.
- **Diagnostic-first** — repeated failures spawn a diagnostic skill before retry escalation.
- **Failure-cache tiering** — adapters that lose upstream connectivity cache last-good with a `stale=true` flag and TIAMAT enforces conservative-mode under staleness.

---

## 8. Quick Start (Domain-Agnostic)

1. Copy `hermes_pantheon/` into your ØMEGA deployment.
2. Edit `risk/risk_governance.lock.yaml` to your domain's actual constraints.
3. `chmod 444` the file, then `chattr +i` (Linux) or `chflags uchg` (macOS) or ICACLS deny-write (Windows).
4. `sha256sum risk_governance.lock.yaml > risk_master.hash`.
5. Start the guardian: `python hermes_pantheon/risk/risk_guardian.py &`.
6. Drop your domain knowledge into `knowledge_vault/`.
7. From PRIME, register a `council_session` task type that maps to this module's debate protocol.
8. From POSEIDON, issue your first briefing.

The Council debates. NAUTILUS validates. TIAMAT verifies the policy hash. MEGALODON acts only after all three.
