# PROTEUS AI — BB-Terminal Adapter

> **Purpose:** Surface the entire Hermes Pantheon's reasoning into a Bloomberg-style operator console derived from the `vaughanf1/BB-Terminal` repository. Make the Council's debate, the RBI gauntlet, the TIAMAT verdict, and the live execution flow visible to the human operator in real time.
> **Owner:** PROTEUS AI (Visual Overlay — Servant tier).

## Source

- **Repository:** [`vaughanf1/BB-Terminal`](https://github.com/vaughanf1/BB-Terminal)
- **Stack:** React 19 / Vite / TypeScript / Tailwind, Bloomberg-inspired terminal UI with a function-code command bar.

## Function Code Mapping

The BB-Terminal already supports a Bloomberg-style function-code command bar. PROTEUS adds Pantheon-specific function codes (additively — existing codes remain authoritative).

| Function Code | Description (PROTEUS overlay) |
|---|---|
| `INTEL` | Live Council Session feed — every active and recent CouncilBriefing, Position, rebuttal, weighted aggregation, ConsensusSignal/Rejection. |
| `OMON` | Order Monitor — every MEGALODON-routed order with TIAMAT verdict trail. |
| `CURV` | Curve / regime view — NAUTILUS's HMM regime classification over time. |
| `QR` | Quote board with Pantheon overlay (SIREN charting, CHARYBDIS value-area, SCYLLA L2 imbalance). |
| `GP` | Genome Pipeline — NAUTILUS RBI gauntlet stages for every Strategy Genome. |
| `HP` | Heatmap — portfolio heat, correlation matrix, TIAMAT-monitored. |
| `KEY` | Keystroke log of every Pantheon command issued. |
| `FA` | Fundamental Aegir — macro context block from AEGIR. |
| `DVD` | Drawdown Dashboard — live vs. backtest expectation per genome. |
| `EE` | Embargo Events — upcoming Tier-1 events from AEGIR. |
| `NI` | News Index — WorldMonitor breaking-news scroll. |
| `MOV` | Movers — top-of-book intraday movers (AEGIR + SCYLLA filtered). |
| `CRYPTO` / `FXC` / `WEI` / `CC` | Asset-class panels with Pantheon overlay. |
| `HELP` / `DES` | Bloomberg-conventional help and description. |

> All existing BB-Terminal function codes remain unchanged. PROTEUS only adds; nothing in the original BB-Terminal app is removed.

## Live Data Flow

```
LEVIATHAN Pantheon ─── WebSocket ──► PROTEUS adapter ──► BB-Terminal (React)
                                          │
                                          ├── Topic: council.briefing
                                          ├── Topic: council.position
                                          ├── Topic: council.consensus
                                          ├── Topic: council.rejection
                                          ├── Topic: rbi.stage_complete
                                          ├── Topic: tiamat.verdict
                                          ├── Topic: megalodon.order
                                          ├── Topic: orca.position_update
                                          ├── Topic: aegir.macro_event
                                          └── Topic: poseidon.user_message
```

## Reference Implementation Sketch

```typescript
// hermes_pantheon/adapters/proteus_bbterminal.ts
// Wires Pantheon WebSocket events into the BB-Terminal React app's existing event bus.

import { TerminalEventBus } from "@/lib/terminal/eventBus";

const ws = new WebSocket("ws://localhost:8765/pantheon");

ws.onmessage = (msg) => {
  const evt = JSON.parse(msg.data);
  switch (evt.topic) {
    case "council.briefing":
      TerminalEventBus.emit("INTEL", evt);
      break;
    case "council.consensus":
      TerminalEventBus.emit("INTEL", evt);
      TerminalEventBus.emit("GP", { stage: "consensus", ...evt });
      break;
    case "rbi.stage_complete":
      TerminalEventBus.emit("GP", evt);
      break;
    case "tiamat.verdict":
      TerminalEventBus.emit("OMON", evt);
      TerminalEventBus.emit("DVD", evt);
      break;
    case "megalodon.order":
      TerminalEventBus.emit("OMON", evt);
      break;
    case "orca.position_update":
      TerminalEventBus.emit("OMON", evt);
      TerminalEventBus.emit("HP", evt);
      break;
    case "aegir.macro_event":
      TerminalEventBus.emit("FA", evt);
      TerminalEventBus.emit("EE", evt);
      TerminalEventBus.emit("NI", evt);
      break;
    case "poseidon.user_message":
      TerminalEventBus.emit("HELP", evt);
      break;
  }
};
```

## Anti-Pattern Discipline

- `feedback_atomic_writes_shared_state` — any cached terminal-state file uses atomic write.
- `feedback_silent_skip_pattern` — dropped WebSocket events MUST log a named reason.
- `feedback_loop_continuous_awareness` — the terminal UI must reflect the Pantheon's continuous-awareness cadence (no 20-30 min idle gaps in the operator's view).

## Operator Hotkeys

| Hotkey | Action |
|---|---|
| `Ctrl + I` | Jump to `INTEL` (live Council session) |
| `Ctrl + O` | Jump to `OMON` (order monitor) |
| `Ctrl + G` | Jump to `GP` (genome pipeline) |
| `Ctrl + H` | Jump to `HP` (heat / correlation) |
| `Ctrl + K` | Kill switch (writes `HALT.flag`) |

The Kill Switch is the single most important operator UI element — it must be one keystroke away at all times.
