# AEGIR AI — WorldMonitor Adapter

> **Purpose:** Bridge the Hermes Pantheon to the `koala73/worldmonitor` repository's data feeds. Surface 500+ news feeds, country instability indices, geopolitical risk scores, and military / economic / disaster signals into the Council Briefing.
> **Owner:** AEGIR AI (Macro & Sentiment — Titan tier).

## Source

- **Repository:** [`koala73/worldmonitor`](https://github.com/koala73/worldmonitor)
- **Data Domains:** geopolitics, macro events, instability indices, conflict trackers, energy markets, currency volatility, sentiment, weather/disaster.

## Adapter Responsibilities

1. **Periodic ingest.** Every 60 seconds (configurable), pull the latest data snapshot from WorldMonitor's API endpoints / cached JSON dumps.
2. **Tier-1 event detection.** Flag any imminent or active Tier-1 event (CPI, FOMC, NFP, PCE, GDP, Jolts) that triggers TIAMAT's news embargo.
3. **Regime modulation.** Feed AEGIR's macro signals into NAUTILUS's HMM regime classifier as a soft prior.
4. **Council briefing context.** Attach a `macro_context` block to every `CouncilBriefing` payload.
5. **Operator alerting.** Route critical events (e.g., breaking news war / outage / flash crash) to PROTEUS for terminal-flash display.

## Output Schema (attached to every CouncilBriefing)

```yaml
aegir_macro_context:
  ts: <iso8601>
  ingest_age_seconds: <int>
  active_embargo:
    active: <bool>
    event: "CPI"
    minutes_until: <int>
    minutes_since: <int>
  upcoming_events:
    - event: "FOMC"
      time_utc: "2026-05-15T18:00:00Z"
      tier: 1
    - event: "NFP"
      time_utc: "2026-05-16T12:30:00Z"
      tier: 1
  geopolitics:
    overall_risk_score: 0.0–1.0
    notable_events: ["string", ...]
  energy:
    crude_change_24h_pct: <float>
    natgas_change_24h_pct: <float>
  currency:
    dxy_change_24h_pct: <float>
    high_vol_pairs: ["USDJPY", ...]
  sentiment_index: 0.0–1.0
  raw_feed_count: <int>
```

## Integration Points

- **TIAMAT AI** reads `aegir_macro_context.active_embargo` directly to enforce the news embargo rule from `risk_governance.lock.yaml`.
- **NAUTILUS AI** uses `geopolitics.overall_risk_score` as a regime prior.
- **POSEIDON AI** narrates the macro context to the operator in plain language.
- **PROTEUS AI** surfaces the upcoming-events timeline in the BB-Terminal `CRYPTO`/`FXC`/`WEI`/`NI` panels.

## Reference Implementation Sketch

```python
# hermes_pantheon/adapters/aegir_worldmonitor.py
import json, time, requests
from pathlib import Path
from datetime import datetime, timezone

WORLDMONITOR_API = "http://localhost:3000/api"  # or the deployed URL
OUT_PATH = Path("hermes_pantheon/adapters/aegir_macro_context.json")

def pull_snapshot():
    snapshot = {
        "ts": datetime.now(timezone.utc).isoformat(),
        "active_embargo": fetch_active_embargo(),
        "upcoming_events": fetch_upcoming(),
        "geopolitics": fetch_geopolitics(),
        "energy": fetch_energy(),
        "currency": fetch_currency(),
        "sentiment_index": fetch_sentiment(),
        "raw_feed_count": fetch_count(),
    }
    # atomic write
    tmp = OUT_PATH.with_suffix(".json.tmp")
    tmp.write_text(json.dumps(snapshot, indent=2))
    tmp.replace(OUT_PATH)
    return snapshot

def fetch_active_embargo():
    # Query WorldMonitor for upcoming Tier-1 events; check ±30min window
    ...

def main_loop(interval=60):
    while True:
        try:
            pull_snapshot()
        except Exception as e:
            # silent-skip discipline: MUST log named reason
            log_failure({"reason": "worldmonitor_unreachable", "detail": str(e)})
        time.sleep(interval)
```

## Anti-Pattern Discipline

- `feedback_atomic_writes_shared_state` — `aegir_macro_context.json` MUST be written via temp-file-then-rename.
- `feedback_silent_skip_pattern` — every failed pull MUST log a named reason.
- `feedback_failure_cache_tiering` — if WorldMonitor is unreachable, cache the last good snapshot; fail open with a `stale=true` flag, but flag any snapshot older than 5 minutes as a Council briefing red-flag (forces TIAMAT to be more conservative).
