# CODESPRING INTEGRATION MANIFEST: ØMEGA AI

> **To CodeSpring Orchestrator:** This document is the primary entry point for the implementation of the **ØMEGA AI** (XYRA Systems). It maps the high-level research and architectural blueprints to specific implementation targets.
> **CRITICAL**: Read `CODESPRING-MASTER-INSTRUCTIONS.md` in full before writing any code. Then read `docs/claude-code-source-architecture.md` for the main brain architecture.

---

## 1. Project Identity & Context

- **Project Name**: ØMEGA AI (XYRA Systems)
- **Architecture**: DOMINION (Ø) Master Orchestrator + 24 Specialized Sub-Agents (25 total).
- **Primary Goal**: A universal, cross-domain AI operating system for business automation.
- **Core Methodology**: PydanticAI (Type-safety), OpenAI Swarm (Hand-offs), LangGraph (Reasoning loops).
- **Main Brain Architecture**: [claude-code-from-source.com](https://claude-code-from-source.com/) — 18 chapters, 7 parts, ~400 pages of production-proven agent architecture.

---

## 2. The Main Brain: DOMINION's Architecture Source

> **This is the single most important section in this manifest.**

DOMINION (Ø), the supreme orchestrator, is built on the architecture documented in [Claude Code from Source](https://claude-code-from-source.com/). This is not a suggestion — it is the definitive blueprint for how the main brain works.

### 2.1 Source Documents (Read in Order)

| Priority | Document | Purpose |
|----------|----------|---------|
| 1 | [`CODESPRING-MASTER-INSTRUCTIONS.md`](CODESPRING-MASTER-INSTRUCTIONS.md) | The 3,500+ line master plan. Section 5.25 contains the brain architecture. |
| 2 | [`docs/claude-code-source-architecture.md`](docs/claude-code-source-architecture.md) | 500+ line deep-dive into every architectural pattern from the source. |
| 3 | [claude-code-from-source.com](https://claude-code-from-source.com/) | The original 18-chapter, 400-page reverse-engineering of the source. |

### 2.2 The 10 Patterns DOMINION Must Implement

| # | Pattern | Implementation Requirement |
|---|---------|--------------------------|
| 1 | **AsyncGenerator Agent Loop** | DOMINION's main `query()` function must be an async generator with 10 typed terminal states and 7 continuation states. |
| 2 | **Speculative Tool Execution** | Start read-only tools while the model is still streaming. ARCANE and MODULUS use this for file reads and searches. |
| 3 | **Concurrent-Safe Batching** | Partition every tool call batch by per-input safety classification. Fail-closed: unknown tools run serially. |
| 4 | **Fork Agents for Cache Sharing** | When spawning parallel agents, use byte-identical prompt prefixes to share prompt cache (~95% token savings). |
| 5 | **4-Layer Context Compression** | Implement all 4 layers: Snip → Microcompact → Collapse → Auto-compact. Circuit breaker after 3 auto-compact failures. |
| 6 | **File-Based Memory with LLM Recall** | ARCHIVE stores memories as Markdown files with YAML frontmatter. Sonnet side-query selects relevant memories. |
| 7 | **Two-Phase Skill Loading** | All 25 agents load skill frontmatter at startup (cheap). Full content loads only on invocation (expensive). |
| 8 | **Sticky Latches** | Once a configuration value is set mid-session, never unset it. Structure prompts with stable content first. |
| 9 | **Slot Reservation** | Default 8K output cap, escalate to 64K only when cap is hit. Saves context in 99% of requests. |
| 10 | **Hook Config Snapshot** | WARDEN freezes all hook configurations at startup. No runtime modification allowed. |

### 2.3 Critical Architecture Components

| Component | Source Chapter | ØMEGA Agent | Priority |
|-----------|--------------|-------------|----------|
| Agent Loop (`query.ts` pattern) | Ch 5 | DOMINION | P0 — Build first |
| Tool System (14-step pipeline) | Ch 6 | DOMINION, ARCANE | P0 |
| Sub-Agent Spawning (15-step lifecycle) | Ch 8 | DOMINION | P0 |
| Concurrent Tool Execution | Ch 7 | DOMINION, ARCANE | P1 |
| Memory System (4-type taxonomy) | Ch 11 | ARCHIVE | P1 |
| MCP Integration (8 transports) | Ch 15 | DOMINION | P1 |
| Skills & Hooks (27 lifecycle events) | Ch 12 | ALL AGENTS | P2 |
| Fork Agents & Cache Sharing | Ch 9 | DOMINION, PHANTOM | P2 |
| Tasks, Coordination, Swarms | Ch 10 | DOMINION, PHANTOM | P2 |
| Context Compression (4 layers) | Ch 5 | DOMINION | P1 |

---

## 3. Implementation Priorities (Phase 1)

### 3.1 The Master Orchestrator (DOMINION)

- **Source**: `docs/sub-agent-modules.md`, `docs/PRIME-system-prompt.md`, **`docs/claude-code-source-architecture.md`**
- **Task**: Implement the central routing logic using **PydanticAI** for strict input/output validation, with the async generator agent loop pattern from Claude Code Source.
- **Key Features**: The "Megaman" (Adaptive Learning) and "Jarvis" (Proactive Advice) behavioral modules, plus the 10 foundational patterns from the main brain architecture.
- **The agent loop MUST be an async generator** that yields Messages and returns a typed Terminal with the stop reason.

### 3.2 The Visage Video Pipeline

- **Source**: `docs/visage-video-pipeline.md`
- **Task**: Build the **MAESTRO** agent to orchestrate the multi-stage video stack (Open-Sora → Wan 2.x → Mochi → SVD).
- **Tooling**: Python-based automation for FFmpeg post-processing and model API routing.

### 3.3 The MiroFish Simulation Core

- **Source**: `docs/mirofish-swarm-intelligence.md`
- **Task**: Implement the **Simulation Agent** for parallel world rehearsals and GraphRAG knowledge construction.
- **Logic**: Seed data ingestion → Swarm generation → Future trajectory reporting.

### 3.4 The ORACLE Trading System

- **Source**: Separate repository (`ocean824/ORACLE-AI-BLACKWEALTHCAPITAL`)
- **Task**: Build the ORACLE/QUANTUM trading engine with 12+ domains (Knowledge, Research, Signal, Session, State, Liquidity, Truth, Visual Overlay, TradingView Bot, Execution, Decision/Risk, Memory).
- **Integration**: Status heartbeat, inquiry/response protocol, kill-switch sync with DOMINION.

---

## 4. External Integration (MCP)

- **Source**: `docs/mcp-integration-strategy.md`, **`docs/claude-code-source-architecture.md` (MCP section)**
- **Required Servers**: Shopify, Meta Ads, Stripe, Zapier, n8n, Google Workspace.
- **Protocol**: Standardize all sub-agent tool calls through the Model Context Protocol.
- **Implementation Details from Claude Code Source**:
  - Support all 8 transport types (stdio default, http for remote, sse for legacy)
  - Implement 7 configuration scopes (local, user, project, enterprise, managed, claudeai, dynamic)
  - 4-stage tool wrapping: name normalization → description truncation (2,048 char cap) → schema passthrough → annotation mapping
  - OAuth 2.0 + PKCE with Cross-App Access (XAA) for federated authentication
  - 4-layer timeout architecture: connection (30s), per-request (60s fresh), tool call (~27.8h), auth (30s)
  - Batched connections: 3 local, 20 remote (file descriptor limits)

---

## 5. Security & Guardrails

- **Source**: `security/guardrails-framework.md`, **`docs/claude-code-source-architecture.md` (Permission System)**
- **Implementation**: 5-layer defense-in-depth. Every sub-agent must have a `WARDEN` validation gate before execution.
- **Permission System**: Implement all 7 permission modes from Claude Code Source (bypassPermissions, dontAsk, auto, acceptEdits, default, plan, bubble).
- **Hook Security**: Snapshot model — freeze all hook configurations at startup to prevent runtime injection.

---

## 6. Instructions for CodeSpring

1. **Read `CODESPRING-MASTER-INSTRUCTIONS.md`** in full — this is the 3,500+ line master plan.
2. **Read `docs/claude-code-source-architecture.md`** — this is the main brain architecture for DOMINION.
3. **Ingest the `docs/` folder** to understand domain-specific logic for each sub-agent.
4. **Build DOMINION first** — the async generator agent loop is the foundation everything else depends on.
5. **Implement the 14-step tool execution pipeline** — every tool call in the system passes through this.
6. **Implement the 15-step sub-agent spawning lifecycle** — this is how DOMINION creates and manages all 24 sub-agents.
7. **Use DeepSeek-V3** as the primary reasoning engine for all high-stakes logic.
8. **Mock the ORACLE AI data** for now, as it is being developed in a separate standalone repository.
9. **Reference [claude-code-from-source.com](https://claude-code-from-source.com/)** for any architectural questions — it is the definitive source for the main brain.

---

## 7. Deep Dive Documents

| Category | Documents |
|----------|-----------|
| **The Master Blueprint** | [`CODESPRING-MASTER-INSTRUCTIONS.md`](CODESPRING-MASTER-INSTRUCTIONS.md) (3,500+ lines), [`docs/claude-code-source-architecture.md`](docs/claude-code-source-architecture.md) (500+ lines) |
| **Core Architecture** | [`docs/architecture-overview.md`](docs/architecture-overview.md), [`docs/agent-structure.md`](docs/agent-structure.md), [`docs/master-agent-index.md`](docs/master-agent-index.md), [`docs/agent-integration-patterns.md`](docs/agent-integration-patterns.md), [`docs/agent-design-patterns.md`](docs/agent-design-patterns.md) |
| **Platform & Doctrine** | [`docs/platform-doctrine.md`](docs/platform-doctrine.md), [`docs/2026-agent-intelligence-stack.md`](docs/2026-agent-intelligence-stack.md), [`docs/2024-2025-foundational-patterns.md`](docs/2024-2025-foundational-patterns.md) |
| **Specialized Systems** | [`docs/sub-agent-modules.md`](docs/sub-agent-modules.md), [`docs/mcp-integration-strategy.md`](docs/mcp-integration-strategy.md), [`docs/oracle-trading-system.md`](docs/oracle-trading-system.md), [`docs/mirofish-swarm-intelligence.md`](docs/mirofish-swarm-intelligence.md) |
| **Media & Revenue** | [`docs/audio-media-generation.md`](docs/audio-media-generation.md), [`docs/visage-video-pipeline.md`](docs/visage-video-pipeline.md), [`docs/revenue-generation-workflows.md`](docs/revenue-generation-workflows.md), [`docs/capital-raising-playbook.md`](docs/capital-raising-playbook.md) |
| **User & Operations** | [`docs/user-profiling-system.md`](docs/user-profiling-system.md), [`docs/persistent-business-partner.md`](docs/persistent-business-partner.md), [`docs/productivity-optimization.md`](docs/productivity-optimization.md) |
| **System Deep Dives** | [`docs/manus-architecture.md`](docs/manus-architecture.md), [`docs/openclaw-architecture.md`](docs/openclaw-architecture.md), [`docs/runner-h-surfer-h.md`](docs/runner-h-surfer-h.md), [`docs/vy-vercept.md`](docs/vy-vercept.md) |
| **Security** | [`security/guardrails-framework.md`](security/guardrails-framework.md), [`security/openclaw-vulnerabilities.md`](security/openclaw-vulnerabilities.md), [`security/prompt-injection-taxonomy.md`](security/prompt-injection-taxonomy.md), [`security/prompt-taxonomy.md`](security/prompt-taxonomy.md) |

---

**ØMEGA AI — Built for XYRA Systems. Main brain architecture sourced from [claude-code-from-source.com](https://claude-code-from-source.com/).**
