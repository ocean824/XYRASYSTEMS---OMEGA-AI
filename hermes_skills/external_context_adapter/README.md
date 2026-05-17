# Skill: `external_context_adapter`

> **Primary caller:** Λ — NEXUS (Integration Hub). Common downstream consumers: Ε — PHANTOM, Χ — SENTINEL, Θ — QUANTUM, Ν — VANGUARD, Δ — MAESTRO.
> **Purpose:** A generic adapter contract for pulling external context into PRIME-orchestrated task envelopes — with explicit staleness handling and conservative-mode fallback.

## Why It Exists

NEXUS already owns API/OAuth/webhook integration. This skill standardizes the *shape* of the context payload so any agent that needs upstream signal (PHANTOM scraping competitors, QUANTUM ingesting WorldMonitor news, SENTINEL watching error feeds, MAESTRO pulling marketing analytics) gets a consistent envelope with consistent staleness semantics.

## Adapter Contract

Every external-context adapter is a function with this signature:

```yaml
adapter_id: <string>          # e.g., "worldmonitor", "unusualwhales", "google_news", "cve_feed"
poll_interval_sec: <int>
output_schema:
  fetched_at: <iso8601>
  source: <adapter_id>
  payload: <free-form structured>
  stale: <bool>
  stale_reason: <string|null>  # e.g., "upstream_5xx", "quota_exhausted", "config_missing"
```

## Staleness & Conservative Mode

When an adapter cannot reach its upstream within the polling interval:

1. The adapter returns the last successful payload with `stale: true` and a named `stale_reason`.
2. NEXUS marks the adapter as degraded in its registry.
3. Any downstream agent reading the payload checks the `stale` flag:
   - **Β — WARDEN:** if a `stale: true` adapter is upstream of an irreversible action and the staleness exceeds the policy floor, WARDEN forces conservative mode (read-only, no irreversible commits).
   - **Χ — SENTINEL:** alerts the operator if any adapter is stale beyond its grace period.
   - **Other agents:** continue to operate but log the `stale` annotation in any decision they make off the payload.

This is the **failure-cache tiering** anti-pattern guard from ØMEGA's existing engineering doctrine.

## Reference Adapters Provided

| Adapter | Purpose | Vertical |
|---|---|---|
| `worldmonitor` | Global macro/news/geopolitical feed (the koala73/worldmonitor open-source project). Drives news-embargo enforcement. | Trading (QUANTUM) |
| `unusualwhales` (placeholder) | Options flow / dark-pool sweeps. | Trading (QUANTUM) |
| `competitor_scraper` (placeholder) | Public-web competitor positioning. | Marketing (PHANTOM/MAESTRO) |
| `cve_feed` (placeholder) | CVE / vulnerability ingestion. | Security (WARDEN/SENTINEL) |
| `regulatory_feed` (placeholder) | Regulatory updates / compliance bulletins. | Legal (JURIS) |

Operators add new adapters by writing a Python module that satisfies the contract above and registering it with NEXUS. The skill is the contract; the adapters are the implementations.

## Hard Rules

- Adapters never write to LOOM/ARCHIVE directly; they emit payloads NEXUS routes.
- An adapter that cannot find its API key/credential at startup must register itself as `stale=true, stale_reason=config_missing` and refuse to emit fresh payloads. Silent failure is forbidden.
- Every payload is timestamped with `fetched_at` for replay/audit.

## Reference Instantiation
The trading-specialized WorldMonitor adapter (mapping news, geopolitical risk scores, and macro events to QUANTUM/LEVIATHAN's news-embargo enforcement) lives in [`ocean824/LEVIATHAN-AI-BLACKWEALTHCAPITAL/hermes_pantheon/adapters/aegir_worldmonitor.md`](https://github.com/ocean824/LEVIATHAN-AI-BLACKWEALTHCAPITAL/blob/main/hermes_pantheon/adapters/aegir_worldmonitor.md).
