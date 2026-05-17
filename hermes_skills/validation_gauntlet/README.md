# Skill: `validation_gauntlet`

> **Primary callers:** Γ — SIGMA (Optimization & Self-Audit), Ζ — ARCANE (Engineering & Code Generation), Θ — QUANTUM (in trading deployments).
> **Pattern lineage:** MoonDev's *Research → Backtest → Implement* (RBI) framework, generalized for any irreversible-action vertical.
> **Purpose:** No `ConsensusSignal` becomes an irreversible action without surviving an explicit, named, auditable validation pipeline.

## When SIGMA / ARCANE Should Call This Skill

- After `council_debate` returns a `ConsensusSignal` for any action that costs money, mutates external state, publishes content, modifies user accounts, or otherwise cannot be cheaply reversed.
- Whenever ARCANE drafts a new artifact (trading bot, automation script, n8n workflow, deployable SaaS module) and SIGMA needs proof it improves on the current baseline before promotion.

## The 8-Stage Gauntlet

Each stage either passes (and proceeds) or fails (with a named reason recorded). No stage is silently skipped.

| Stage | Owner | Purpose |
|---|---|---|
| **R1 — Initial Validation** | SIGMA | Static checks: schema, type, sanity, regression-safety. No execution. |
| **R2 — Sandboxed Dry-Run** | ARCANE | Execute against fixtures or replay data in an isolated sandbox. |
| **R3 — Monte Carlo / Stochastic** | SIGMA | Run N stochastic simulations to bound expected outcome distribution. |
| **R4 — Walk-Forward / Out-of-Sample** | SIGMA | Train on early window, test on later window; repeat across rolling segments. |
| **R5 — Regime / Segment Robustness** | SIGMA + Ε — PHANTOM | Re-run across distinct regimes/segments (bull/bear, high/low traffic, weekday/weekend, etc.) to ensure performance is not regime-specific. |
| **R6 — Peer Cross-Reference** | Α — LOOM | Compare against historical similar artifacts in the knowledge vault for known failure modes. |
| **R7 — Paper / Shadow Deployment** | Θ — QUANTUM (trading) or Ν — VANGUARD (outreach) or relevant operations agent | Run live but with no external commitment for a defined trial period. |
| **R8 — Live with Continuous Audit** | Ω — TRIBUNE owns operator hand-off; Χ — SENTINEL monitors | Promote to live; SENTINEL alerts on any breach of the gauntlet's success criteria. |

### Pass Thresholds (defaults; deployment-tunable)

| Vertical | R3/R4 metrics | R5 floor | R7 trial duration |
|---|---|---|---|
| Trading (QUANTUM/LEVIATHAN) | Sharpe ≥ 1.5; MaxDD ≤ 15%; Win-rate ≥ 50% | All regimes positive | ≥ 30 sessions |
| Marketing (MAESTRO/OMNIPOST) | Lift ≥ 5%; CTR ≥ baseline | A/B passes in ≥ 2 segments | ≥ 7 days |
| Engineering (ARCANE) | Tests ≥ 95% pass; no perf regression | No segment regression | ≥ 1 release cycle |
| Outreach (VANGUARD/SIREN) | Reply-rate ≥ baseline; spam ≤ 0.1% | Passes in ≥ 2 list segments | ≥ 7 days |

## Contract

### Input
```yaml
artifact_id: <uuid>
artifact_kind: trading_strategy | marketing_campaign | code_module | outreach_sequence | other
artifact_payload: <ref to LOOM/ARCHIVE>
consensus_signal: <ref to council_debate output>
deployment_thresholds: <override defaults>
```

### Output
```yaml
gauntlet_id: <uuid>
artifact_id: <uuid>
final_status: passed | failed | quarantined
stage_results:
  - stage: R1
    status: pass | fail
    reason: "<named reason>"
    evidence_ref: <path>
  ...
final_recommendation: promote_to_live | extend_paper | discard
```

## Hard Rules

- A `failed` stage halts the gauntlet immediately. Failed artifacts are quarantined under LOOM/ARCHIVE for post-mortem; they are never silently retried.
- No artifact reaches R8 (live) without WARDEN's `sovereign_governance_lock` heartbeat verification confirming the policy file has not been tampered with since R1 began.
- Every stage transition is an atomic write; the gauntlet's state file uses temp-then-rename semantics.

## Reference Instantiation
The trading-specialized RBI gauntlet — with QUANTUM/LEVIATHAN-specific metrics (Sharpe, MaxDD, win-rate floors), backtesting via `pandas` and `backtesting.py`, and Monte Carlo / walk-forward stages tuned for OHLCV data — lives in [`ocean824/LEVIATHAN-AI-BLACKWEALTHCAPITAL/hermes_pantheon/rbi`](https://github.com/ocean824/LEVIATHAN-AI-BLACKWEALTHCAPITAL/tree/main/hermes_pantheon/rbi).
