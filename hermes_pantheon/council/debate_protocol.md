# The Council Debate Protocol

> **Inspired by:** Godmod3 / Mirrorfish multi-agent consensus frameworks
> **Purpose:** Convert independent, paradigm-specific evidence from the Sensory Network into a single weighted, defensible trade decision.
> **Status:** Additive layer above the existing LEVIATHAN Decision Domain. Does not replace it; supplies its primary input.

---

## 1. When a Council Session is Convened

A Council Session is triggered by any of the following events:

1. **Setup Detection.** SIREN AI flags a chart-based candidate setup (e.g., Market Cipher buy signal with bullish divergence + LuxAlgo trend confirmation).
2. **State Inflection.** CHARYBDIS AI detects a value-area shift, single print, or auction failure.
3. **Liquidity Event.** SCYLLA AI detects a stop run, liquidity grab, or large dark-pool print.
4. **Macro Event Window.** AEGIR AI flags a Tier-2 event window where opportunistic positioning is allowed.
5. **User Inquiry.** POSEIDON AI receives a direct user question (e.g., "should I be long ES here?") and dispatches the request to the Council.

---

## 2. Session Structure

Each session is a finite-state machine with the following phases:

### Phase 1 — Briefing (5 seconds)

- The trigger event is packaged into a `CouncilBriefing` JSON payload containing: instrument, timeframe, current price, macro context (from AEGIR), session context (from MERMAID), state snapshot (from CHARYBDIS), live order-book snapshot (from SCYLLA), and the originating signal payload (from SIREN).
- TRITON AI attaches any relevant Obsidian knowledge tags that match the instrument, methodology, or regime.

### Phase 2 — Independent Analysis (10–30 seconds)

Each Sensory Network agent receives the `CouncilBriefing` and produces a `Position` payload **independently** (no agent sees another agent's vote at this stage). A Position contains:

```json
{
  "agent": "scylla",
  "verdict": "long" | "short" | "abstain" | "veto",
  "confidence": 0.0–1.0,
  "primary_evidence": "string (one sentence)",
  "secondary_evidence": ["string", "..."],
  "methodology_lens": "bank_protocol_stop_run",
  "regime_dependency": "high_vol_only" | "any" | "low_vol_only",
  "invalidation_level": "<price>",
  "expected_horizon_minutes": <int>
}
```

The Titans (NAUTILUS, AEGIR) and Servants (TRITON, PROTEUS) do **not** vote in Phase 2 — they provide context only. Voting agents are: SCYLLA, CHARYBDIS, SIREN, MERMAID. Optional advisory votes from LOCHNESS (if a candidate genome already exists for this pattern).

### Phase 3 — Debate (15–60 seconds)

Positions are revealed to all voting agents. Each agent may issue **rebuttals** to other Positions. A rebuttal must cite a specific piece of evidence that contradicts another agent's reasoning.

- Example: SCYLLA votes **long** based on absorption at the lows; CHARYBDIS rebuts that the absorption is occurring inside a low-volume node and is therefore unstable. SCYLLA may revise its confidence downward or hold its Position with a counter-rebuttal.
- A maximum of **2 debate rounds** is allowed to prevent infinite loops.

### Phase 4 — Weighted Aggregation (LEVIATHAN AI)

LEVIATHAN AI consumes the final Positions and computes a single weighted consensus. Each agent's vote is weighted by:

1. **Methodology fit for current regime.** (NAUTILUS provides the regime classification: trending / ranging / high-vol / low-vol / news-driven.)
2. **Historical accuracy in this regime.** (Drawn from ORCA AI's memory of past Council sessions.)
3. **Confidence score** declared in the agent's Position.
4. **Recency of training corpus.** (Knowledge from the last 30 days carries a slight uplift; knowledge older than 12 months is decayed.)

Default weights (configurable, but bounded — TIAMAT vetoes any weight set that would let one agent dominate beyond 50%):

| Agent | Default Weight |
|---|---|
| SCYLLA AI | 25% |
| CHARYBDIS AI | 25% |
| SIREN AI | 20% |
| MERMAID AI | 15% |
| LOCHNESS AI (advisory) | 10% |
| AEGIR AI (gating only) | 5% (used as a multiplier on the total, not as an additive vote) |

### Phase 5 — Consensus Decision

- If the weighted sum of "long" or "short" verdicts ≥ **75%**, the Council reaches consensus.
- If consensus is below 75%, the setup is discarded with a `CouncilRejection` recorded to ORCA AI's memory.
- If any agent issued a `veto` verdict (reserved for hard contradictions: e.g., a confirmed news embargo, a confirmed bank-protocol manipulation in the opposite direction), the session terminates immediately with `CouncilVeto`.

A successful consensus produces a `ConsensusSignal`:

```json
{
  "session_id": "uuid",
  "instrument": "ES",
  "direction": "long",
  "weighted_confidence": 0.83,
  "regime": "high_vol_trending",
  "supporting_positions": [...],
  "rebuttals": [...],
  "invalidation_level": 4985.25,
  "expected_horizon_minutes": 240,
  "leviathan_allocation": { "kelly_fraction": 0.35, "size_units": 2 }
}
```

---

## 3. From Consensus to Execution

The `ConsensusSignal` is then handed to the existing LEVIATHAN pipeline:

1. **NAUTILUS AI** validates that an existing, unstaled Strategy Genome covers this pattern. If yes → proceed. If no → spawns LOCHNESS to draft a new genome and runs the MoonDev RBI gauntlet (see `../rbi/rbi_pipeline.md`).
2. **TIAMAT AI** verifies the immutable risk file hash and applies sovereign vetoes.
3. **MEGALODON AI** routes the order.
4. **ORCA AI** manages the position and records the outcome back into the Memory Domain — which feeds the next Council session's weighting algorithm.
5. **PROTEUS AI** mirrors the entire session transcript into the BB-Terminal operator console.
6. **POSEIDON AI** delivers a plain-language explanation to the user.

---

## 4. Council Transparency

Every Council Session is fully auditable:

- The `CouncilBriefing`, all `Positions`, all rebuttals, and the final `ConsensusSignal` (or rejection) are persisted to PostgreSQL/TimescaleDB with a `session_id`.
- POSEIDON AI can replay any session for the user in plain language ("Show me why we passed on the 09:47 ES setup").
- The session log is immutable once written.

---

## 5. Why a Council and Not a Single Model?

A single LLM is brittle: it overfits to its prompt, anchors to the loudest evidence, and has no real concept of regime-specific accuracy. A Council:

- **Forces independent reasoning** before any cross-agent contamination.
- **Captures methodology orthogonality** — an order-flow read and an auction-theory read are genuinely different lenses, and their disagreement is meaningful information.
- **Produces an audit trail** that a human supervisor (and POSEIDON) can interrogate.
- **Self-improves** because each agent's weight is tuned by historical accuracy in the current regime, not by static priors.

This is the same insight that drove the Godmod3 / Mirrorfish frameworks: consensus from independent specialists is a stronger signal than any single oracle.
