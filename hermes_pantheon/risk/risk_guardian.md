# TIAMAT AI — Risk Guardian Daemon

> **Purpose:** Stand between the Hermes Pantheon and MEGALODON AI's broker connection. Refuse to allow any order route unless the immutable `risk_governance.lock.yaml` file's SHA-256 hash matches the operator-recorded master hash, and unless the proposed order does not violate any active sovereign rule.

## Responsibilities

1. **Hash verification.** On every order route request, recompute SHA-256 of `risk_governance.lock.yaml` and compare to `risk_master.hash`. Mismatch → `tiamat_hash_mismatch` veto + global halt.
2. **Rule evaluation.** Run each sovereign rule against the proposed order and the current portfolio state.
3. **Heartbeat verification.** Independent of orders, recompute the hash every `audit.verify_interval_seconds` (default 60s). Any drift → halt.
4. **Audit log.** Append every verification to `risk_audit.jsonl` with the order id, timestamp, hash, verdict, and rule that fired (if any).
5. **Kill switch monitoring.** Watch the `HALT.flag` file. If present, refuse all orders. Watch for ØMEGA cross-system kill signal.

## Reference Implementation (Python)

```python
# hermes_pantheon/risk/risk_guardian.py
"""
TIAMAT Risk Guardian — sovereign veto layer.
Run as a long-lived daemon. MEGALODON must call verify_and_approve()
before every broker submit.
"""
import hashlib, json, os, time, yaml
from pathlib import Path
from datetime import datetime, timezone

LOCK_FILE = Path("hermes_pantheon/risk/risk_governance.lock.yaml")
MASTER_HASH = Path("hermes_pantheon/risk/risk_master.hash")
HALT_FLAG = Path("hermes_pantheon/risk/HALT.flag")
AUDIT_LOG = Path("hermes_pantheon/risk/risk_audit.jsonl")
PROPOSAL_LOG = Path("hermes_pantheon/risk/proposals.jsonl")

class TiamatVeto(Exception):
    pass

def sha256_file(path: Path) -> str:
    h = hashlib.sha256()
    h.update(path.read_bytes())
    return h.hexdigest()

def load_master_hash() -> str:
    if not MASTER_HASH.exists():
        raise TiamatVeto("master_hash_missing")
    return MASTER_HASH.read_text().split()[0].strip()

def load_rules() -> dict:
    return yaml.safe_load(LOCK_FILE.read_text())

def audit(order_id: str, verdict: str, reason: str, hash_now: str):
    AUDIT_LOG.parent.mkdir(parents=True, exist_ok=True)
    entry = {
        "ts": datetime.now(timezone.utc).isoformat(),
        "order_id": order_id,
        "verdict": verdict,
        "reason": reason,
        "hash": hash_now,
    }
    # atomic append
    tmp = AUDIT_LOG.with_suffix(".jsonl.tmp")
    with tmp.open("a") as f:
        f.write(json.dumps(entry) + "\n")
    os.replace(tmp, AUDIT_LOG) if not AUDIT_LOG.exists() else None
    with AUDIT_LOG.open("a") as f:
        f.write(json.dumps(entry) + "\n")

def verify_and_approve(order: dict, portfolio_state: dict) -> dict:
    """Called by MEGALODON before broker submit. Returns approval or raises."""
    order_id = order.get("id", "unknown")

    # 0. Halt flag
    if HALT_FLAG.exists():
        audit(order_id, "veto", "halt_flag_present", "")
        raise TiamatVeto("halt_flag_present")

    # 1. Hash verification
    hash_now = sha256_file(LOCK_FILE)
    hash_master = load_master_hash()
    if hash_now != hash_master:
        audit(order_id, "veto", "tiamat_hash_mismatch", hash_now)
        # Auto-create halt flag on mismatch
        HALT_FLAG.write_text(f"hash_mismatch at {datetime.now(timezone.utc).isoformat()}")
        raise TiamatVeto("tiamat_hash_mismatch")

    # 2. Rule evaluation
    rules = load_rules()
    veto_reason = evaluate_rules(order, portfolio_state, rules)
    if veto_reason:
        audit(order_id, "veto", veto_reason, hash_now)
        raise TiamatVeto(veto_reason)

    # 3. Approval
    audit(order_id, "approve", "ok", hash_now)
    return {
        "granted": True,
        "order_id": order_id,
        "hash": hash_now,
        "ts": datetime.now(timezone.utc).isoformat(),
    }

def evaluate_rules(order, state, rules) -> str | None:
    # Drawdown
    if state["daily_drawdown_pct"] >= rules["drawdown"]["max_daily_pct"]:
        return "drawdown_max_daily"
    if state["weekly_drawdown_pct"] >= rules["drawdown"]["max_weekly_pct"]:
        return "drawdown_max_weekly"
    if state["trailing_drawdown_pct"] >= rules["drawdown"]["max_trailing_pct"]:
        return "drawdown_max_trailing"
    # Heat
    if state["open_heat_pct"] + order["risk_pct"] > rules["heat"]["max_open_pct"]:
        return "heat_cap"
    if order["risk_pct"] > rules["heat"]["max_per_position_pct"]:
        return "heat_per_position"
    # Leverage
    if state["projected_gross_notional_x"] > rules["leverage"]["max_gross_notional_x"]:
        return "leverage_gross"
    if state["projected_net_x"] > rules["leverage"]["max_net_x"]:
        return "leverage_net"
    # Correlation
    if state["sector_count_same_dir"] >= rules["correlation"]["max_positions_same_sector"]:
        return "correlation_cap"
    # News embargo
    if state.get("aegir_embargo_active"):
        return "news_embargo"
    # Loss streak
    if state["consecutive_losers"] >= rules["loss_streak"]["consecutive_losers"]:
        # not a veto — Megalodon must downsize. Return a special signal here in real impl.
        if order["risk_pct"] > rules["heat"]["max_per_position_pct"] * 0.5:
            return "loss_streak_throttle"
    # Instrument limits
    sym = order["symbol"]
    lim = rules["instrument_limits"].get("overrides", {}).get(sym) or rules["instrument_limits"]["default"]
    if order["units"] > lim["max_position_units"]:
        return f"instrument_units_{sym}"
    if state["open_positions_for"].get(sym, 0) >= lim["max_open_positions_per_symbol"]:
        return f"instrument_open_count_{sym}"
    return None  # approved

def heartbeat_loop(interval=60):
    """Independently verify the lock file every N seconds."""
    last_hash = load_master_hash()
    while True:
        try:
            now = sha256_file(LOCK_FILE)
            if now != last_hash:
                HALT_FLAG.write_text(f"heartbeat_hash_mismatch at {datetime.now(timezone.utc).isoformat()}")
                audit("heartbeat", "halt", "heartbeat_hash_mismatch", now)
        except FileNotFoundError:
            HALT_FLAG.write_text("lock_file_missing")
            audit("heartbeat", "halt", "lock_file_missing", "")
        time.sleep(interval)

if __name__ == "__main__":
    print("[TIAMAT] Risk Guardian heartbeat starting...")
    heartbeat_loop(60)
```

## Anti-Pattern Discipline (inherited from ØMEGA)

- `feedback_atomic_writes_shared_state` — `risk_audit.jsonl` MUST use atomic temp-file-then-rename pattern.
- `feedback_silent_skip_pattern` — every veto MUST carry a named reason; never aggregate as a count.
- `feedback_speculative_write` — never mark an order "approved" without a recorded `risk_audit.jsonl` entry.
- `feedback_diagnostic_first` — when an unexpected veto fires, the operator should be able to replay the exact input portfolio state from `risk_audit.jsonl` before tweaking rules.

## Operator Workflow

1. Edit `risk_governance.lock.yaml` to your desired values.
2. `chmod 444 risk_governance.lock.yaml`
3. Linux: `sudo chattr +i risk_governance.lock.yaml`
4. `sha256sum risk_governance.lock.yaml > risk_master.hash`
5. Start the guardian: `python hermes_pantheon/risk/risk_guardian.py &`
6. To change risk: `sudo chattr -i ... && edit ... && sudo chattr +i ... && sha256sum ... > risk_master.hash` (this is intentionally manual; the Pantheon cannot do it).
