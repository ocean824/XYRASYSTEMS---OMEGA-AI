# Skill: `sovereign_governance_lock`

> **Primary callers:** Β — WARDEN (Security & Risk Control), with Φ — JURIS (Legal & Compliance) auditing the policy file.
> **Extends:** `security/guardrails-framework.md`, `security/prompt-injection-taxonomy.md`.
> **Purpose:** Closes the *"agent edits its own safety rules"* failure mode. Constraints are physically unmodifiable from inside the system; modification requires manual, out-of-band operator action.

## What It Provides

1. **An immutable policy file** at a known path (e.g., `<deployment>/policy/governance.lock.yaml`) — encoded in YAML for human auditability.
2. **A SHA-256 master hash** of that file persisted at first-deploy; verified before every irreversible action and on a 60-second heartbeat.
3. **A standalone guardian process** (separate PID from DOMINION and from any of the 25 agents) that owns the verification loop and possesses the only authority to issue a hard halt.
4. **A documented manual modification ritual** that requires the operator to: (a) prove WARDEN-side audit log of intent, (b) `chattr -i` / `chflags nouchg` / ICACLS-restore the file, (c) edit, (d) re-hash, (e) re-lock, (f) restart the guardian.

## Why Not Just Trust DOMINION To Honor the Policy?

LLM-driven agents — including the orchestrator — are susceptible to prompt injection (`security/prompt-injection-taxonomy.md`), goal drift, and persuasion-attack sequences. A policy stored in a file the agent can rewrite is not a policy; it is a suggestion. This skill makes the policy a contract enforced *outside* the LLM blast radius.

## Lock Mechanism by OS

| OS | Immutable Bit | Modification Requires |
|---|---|---|
| Linux | `chattr +i policy/governance.lock.yaml` | `sudo chattr -i` (root) |
| macOS | `chflags uchg policy/governance.lock.yaml` | `sudo chflags nouchg` (root) |
| Windows | `icacls policy\governance.lock.yaml /deny "*S-1-1-0:(W)"` | `icacls ... /grant ... /reset` (admin) |

## Policy File Schema (deployment-tunable; trading-flavored example)

```yaml
schema_version: 1
deployment_vertical: trading            # or: marketing | engineering | outreach | ...
hash_algorithm: sha256
master_hash_path: ./policy/governance.master.hash

constraints:
  - id: max_daily_drawdown_pct
    value: 3.0
    enforcer: warden
  - id: max_portfolio_heat_pct
    value: 5.0
    enforcer: warden
  - id: max_leverage_x
    value: 5.0
    enforcer: warden
  - id: news_embargo_minutes_before_event
    value: 15
    enforcer: nexus + warden
  - id: kill_switch_consecutive_losses
    value: 5
    enforcer: warden
  - id: irreversible_action_requires_council_consensus
    value: true
    enforcer: dominion
  - id: irreversible_action_requires_validation_gauntlet_pass
    value: true
    enforcer: dominion

operator_overrides:
  unlock_ritual: ./policy/UNLOCK_RITUAL.md
  out_of_band_audit_log: ./policy/audit/ritual.log
```

For non-trading verticals, replace the `constraints` list with whatever the deployment requires (max-daily-bans, allow/deny lists, max-spend caps, content-policy floors, etc.). The schema is generic.

## Guardian Process Contract

Pseudocode for the standalone daemon WARDEN spawns at deploy:

```python
# guardian.py — runs in its own PID, separate from DOMINION and any of the 25 agents
INTERVAL_SEC = 60
POLICY_PATH  = "./policy/governance.lock.yaml"
MASTER_HASH  = open("./policy/governance.master.hash").read().strip()

while True:
    current = sha256(open(POLICY_PATH, "rb").read()).hexdigest()
    if current != MASTER_HASH:
        emit_kill_switch(reason="policy_hash_mismatch")    # halts all 25 agents
        write_audit("hash_mismatch_at_heartbeat", level="critical")
        sys.exit(1)
    sleep(INTERVAL_SEC)
```

In addition to the heartbeat, every irreversible action (e.g., MEGALODON broker order, OMNIPOST publication, TITHE charge, VANGUARD bulk send) calls the guardian *synchronously* before commit; a mismatch aborts the action.

## Hard Rules

- The guardian is the **only** authority that can issue a hard halt across all 25 agents. Not even DOMINION can override it from within the system.
- If the guardian process itself dies, SENTINEL alerts immediately; the system enters conservative mode (read-only) until the guardian is back up.
- Operator modifications produce audit-log entries WARDEN ingests on next deploy.

## Reference Instantiation
The trading-specialized lock (`risk_governance.lock.yaml` with drawdown/leverage/news-embargo constraints and a Python guardian implementation) lives in [`ocean824/LEVIATHAN-AI-BLACKWEALTHCAPITAL/hermes_pantheon/risk`](https://github.com/ocean824/LEVIATHAN-AI-BLACKWEALTHCAPITAL/tree/main/hermes_pantheon/risk).
