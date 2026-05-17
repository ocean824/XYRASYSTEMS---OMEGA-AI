# Skill: `council_debate`

> **Primary callers:** Ø — DOMINION (Supreme Orchestrator), Κ — MODULUS (Strategic Decision Engine), Ψ — MIRRORBACK (Self-Reflection).
> **Natural seats (the 5 Meta-Agents — DOMINION's Inner Council):** Α — LOOM, Β — WARDEN, Γ — SIGMA, Δ — MAESTRO, Ε — PHANTOM. **G0DM0D3** fills the red-team seat per its canonical "Liberated Cognition" mandate.
> **Pattern lineage:** Godmod3 / MiroFish multi-agent debate (already referenced in `docs/mirofish-swarm-intelligence.md`).
> **Purpose:** Produce a defensible *weighted consensus* from a set of specialist agents instead of a single-call LLM verdict.

## When DOMINION Should Call This Skill

- Any decision that would otherwise commit on a single agent's reasoning where the cost of being wrong is non-trivial (capital deployment, irreversible content publication, configuration changes, escalations to TRIBUNE).
- Any time an external user requests a *justified* recommendation — DOMINION attaches the full debate transcript as audit evidence in the canonical audit store.
- Any time DOMINION's red-team posture would benefit from being a structured debate role rather than a one-shot critique.

## Contract

### Input — `Briefing`
```yaml
session_id: <uuid>
question: "<the decision being debated>"
context:
  current_state: <free-form>
  constraints: <list>
  upstream_context: <attached external_context_adapter snapshots>
agents:                       # which existing OMEGA agents fill the seats
  - role: dominion
    weight: 1.0
  - role: modulus
    weight: 1.2               # core decision logic
  - role: g0dm0d3
    weight: 1.5               # red-team gets extra weight on safety questions
  - role: warden
    weight: 1.3               # safety/risk perspective
  - role: sigma
    weight: 1.1               # optimization/self-audit perspective
  - role: phantom
    weight: 1.0               # competitive-intel perspective
  - role: quantum              # optional, only for trading-vertical questions
    weight: 1.2
consensus_threshold: 0.65     # of Σ weights
```

### Stage 1 — Position
Each seated agent produces an independent `Position`:
```yaml
agent: <role>
verdict: support | reject | abstain
confidence: 0.0–1.0
rationale: "<reasoning>"
evidence: ["<refs>"]
```

### Stage 2 — Rebuttal
Each agent reads every other agent's Position and produces one `Rebuttal` per opposing verdict:
```yaml
agent: <role>
target_agent: <role>
counter: "<reasoning>"
update_to_own_confidence: <delta>
```

### Stage 3 — Weighted Aggregation
The skill computes the weighted-confidence support score:
```
score = Σ (agent.weight × agent.final_confidence × {+1 if support, −1 if reject, 0 if abstain})
```

### Stage 4 — Consensus or Rejection
- If `score ≥ consensus_threshold × Σ weights` → emit `ConsensusSignal{ ... }`.
- Else → emit `ConsensusRejection{ reason, transcript }`.

DOMINION persists the full transcript to the canonical audit store (via Α — LOOM and Υ — ARCHIVE) before acting on the result. If the result is `ConsensusSignal` and the action is irreversible, DOMINION must route it through the `validation_gauntlet` skill before any execution.

## Anti-Patterns

- **No silent skips.** If an agent times out, the skill records `{verdict: abstain, reason: "agent_timeout"}` — never drop the agent silently from the tally.
- **No speculative writes.** The `ConsensusSignal` is not persisted until every Position and Rebuttal is on disk.
- **Atomic transcript writes.** Transcript file uses temp-file-then-rename.
- **Diagnostic-first.** If two agents repeatedly produce `agent_error`, spawn a diagnostic skill before re-running the debate.

## Reference Instantiation
The trading-specialized version (with the 14 oceanic agents serving as Council seats inside the LEVIATHAN trading vertical) lives in [`ocean824/LEVIATHAN-AI-BLACKWEALTHCAPITAL/hermes_pantheon/council`](https://github.com/ocean824/LEVIATHAN-AI-BLACKWEALTHCAPITAL/tree/main/hermes_pantheon/council). That repo is what Θ — QUANTUM uses internally; the skill contract here is identical, only the seated specialists change.
