# NAUTILUS AI — MoonDev RBI Pipeline (Research → Backtest → Implement)

> **Origin:** Adapted from MoonDev's `moondevonyt/Harvard-Algorithmic-Trading-with-AI` repository.
> **Owner:** NAUTILUS AI (Research / RBI Lab — Titan tier).
> **Hard Rule:** No Strategy Genome may be marked `validated` without a verifiable backtest record on disk. (See `feedback_speculative_write`.)

---

## 1. Research (R)

When the Council emits a `ConsensusSignal` AND no existing unstaled Strategy Genome covers the pattern, NAUTILUS triggers Research mode and tasks **LOCHNESS AI** with drafting the genome.

**Inputs**
- The Council session transcript (briefing, all Positions, rebuttals, weighted aggregation).
- The current regime classification (NAUTILUS HMM module).
- Any matching Obsidian notes ingested by TRITON.
- The methodology lineage (e.g., "Wyckoff Spring + Bank Protocol stop-run absorption").

**Output: a Strategy Genome** — a structured payload with these required fields:

```yaml
genome_id: <uuid>
created: <iso8601>
methodology: ["wyckoff_spring", "bank_protocol_stop_run"]
instrument_universe: ["ES", "NQ"]
timeframe: "5m"
regime_fit: ["high_vol_trending"]
hypothesis: "<one sentence>"
entry:
  conditions: [...]
  invalidation_level: "<formula>"
exit:
  take_profit: "<formula>"
  stop_loss: "<formula>"
  trailing: "<formula>"
sizing:
  basis: "kelly_fraction"
  max_pct: 1.0
backtest_window:
  start: "2020-01-01"
  end: "2025-12-31"
  bars_minimum: 50000
notes: "<rationale, lineage, council transcript ref>"
```

LOCHNESS produces both a Pine Script and a Python implementation suitable for `backtesting.py`.

---

## 2. Backtest (B)

NAUTILUS runs the genome through the gauntlet using MoonDev's stack: `pandas`, `backtesting.py`, `yfinance` (or live cached OHLCV), `talib`, `ccxt`.

### Stage 1 — Initial Backtest

Run the strategy against the configured `backtest_window`. Compute:

- Total return
- Sharpe ratio
- Sortino ratio
- Maximum drawdown
- Win rate
- Profit factor
- Average MAE / MFE

**Pass criteria** (configurable via `config.json` → `hermes_pantheon.rbi_thresholds`):

| Metric | Default Threshold |
|---|---|
| Sharpe | > 1.5 |
| Max Drawdown | < 15% |
| Win Rate | > 55% |
| Profit Factor | > 1.6 |
| Trades in window | > 100 |

If the genome fails any threshold, NAUTILUS sends it back to LOCHNESS with the specific failure reason. LOCHNESS revises and resubmits (max 3 iterations per genome before escalating to operator).

### Stage 2 — Monte Carlo (50 parallel runs)

Fork 50 isolated subprocesses, each running the strategy with bootstrap-resampled trade orders. For each run, compute the same metric set. Pass criteria:

- 90th-percentile MaxDD < 1.5x of single-run MaxDD
- 10th-percentile Sharpe > 1.0
- No run shows ruin (equity curve never goes negative)

This guards against survivorship bias.

### Stage 3 — Walk-Forward Validation

Split the backtest window into 5 folds. Optimize parameters on each in-sample fold; measure out-of-sample. Pass criteria:

- Out-of-sample Sharpe within 80% of in-sample Sharpe (averaged across folds)
- No single fold shows a negative Sharpe

### Stage 4 — Regime Testing

Use NAUTILUS's HMM regime classifier to label every bar in the test set with one of: `low_vol_trending`, `low_vol_ranging`, `high_vol_trending`, `high_vol_ranging`, `news_driven`. Run the strategy in each regime separately. Pass criteria:

- Profitable in ≥ 3 of 5 regimes
- The regime(s) the genome claims to fit (see `regime_fit`) must be the strongest performers

### Stage 5 — Council Cross-Reference

Query ORCA AI's memory for any prior Council Sessions involving this methodology. If past sessions show contradicting lessons (e.g., "Bank Protocol stop-runs in HFT-driven indices have been failing since 2024"), NAUTILUS appends a *staleness flag* to the genome and may downgrade the deployment ladder.

---

## 3. Implement (I)

A genome that passes all backtest stages graduates to Implement.

### Stage 6 — Paper Trade (14 days, Signal-Only Mode)

NAUTILUS deploys the strategy in **Signal-Only Mode**. SIREN generates the signals; LOCHNESS's code publishes paper-trade fills; ORCA paper-manages the positions. No real capital is risked.

**Pass criteria for graduation to Live:**

- Live Sharpe within 70% of backtest Sharpe
- Live MaxDD within 1.2x of backtest MaxDD
- At least 10 trades in the paper-trade window
- No tail event greater than 2σ outside backtest expectation

### Stage 7 — Semi-Auto Live (40 trades)

The genome is promoted to Semi-Auto Mode. SIREN generates signals; the operator approves each entry. ORCA AI handles the trade. After 40 successful Semi-Auto trades that match the backtest envelope (P&L within 1.5σ, hit rate within ±5%), the genome is promoted to Full Auto.

### Stage 8 — Full Auto + Continuous Audit

The genome runs autonomously. Ongoing checks:

- ORCA AI compares every live trade outcome against backtest expectation; deviations are logged.
- After every 10 trades, NAUTILUS recomputes live metrics. Drift > 2σ → automatic demotion to Semi-Auto.
- Every 14 days, the staleness check (see `feedback_council_consensus_required` for the inherited discipline) re-runs Stage 1 against the most recent rolling window. If thresholds fail, the genome is retired.

---

## 4. Records & Auditability

Every stage writes a record to disk:

```
hermes_pantheon/rbi/runs/<genome_id>/
  ├── 01_research.json
  ├── 02_backtest_initial.json
  ├── 03_montecarlo_runs.jsonl
  ├── 04_walk_forward.json
  ├── 05_regime_test.json
  ├── 06_paper_trade.jsonl
  ├── 07_semi_auto.jsonl
  └── 08_live.jsonl
```

The genome's `status` field (in the master ledger `hermes_pantheon/rbi/genomes.jsonl`) is updated after each stage. **NAUTILUS may never set status to `validated` or `live` without the corresponding stage's run-record file existing on disk.** This is enforced by `feedback_speculative_write`.

---

## 5. Self-Improvement Loop

After a genome reaches Full Auto and accumulates 100+ live trades, NAUTILUS feeds the live trade journal back to TRITON. TRITON tags the recurring success/failure patterns and proposes:

1. New methodology nodes for the relevant agent's training corpus.
2. Regime-specific weight adjustments (proposals only — operator must approve via the `pantheon_evolve` ØMEGA domain).
3. Candidate refinements to LOCHNESS's code (e.g., a tighter trailing logic).

These proposals are persisted to `hermes_pantheon/rbi/proposals.jsonl` and surfaced to the operator via PROTEUS in the next session.

The Pantheon evolves in *strategy* but never in *risk*. TIAMAT's lock is absolute.
