# Agent Design Patterns: Cross-System Analysis

> **Author:** Black Wealth Capital Research Division
> **Status:** Rough Draft — ØMEGA AI Foundation Document
> **Last Updated:** March 2026

---

## Executive Summary

After analyzing the architectures of OpenClaw, Manus, Runner H/Surfer H, Vy/Vercept, Cursor, Windsurf, v0, Lovable, Devin, and Emergent — along with their leaked system prompts — a set of recurring design patterns emerges. These patterns are not coincidental; they represent convergent evolution toward solutions that work at scale. This document catalogs these patterns, explains why they exist, and provides implementation guidance for the ØMEGA AI.

---

## 1. The ReAct Loop (Reason + Act)

**Observed in:** All studied systems (OpenClaw, Manus, Cursor, Windsurf, Devin, Runner H, v0, Lovable, Emergent)

**Pattern Description:**
The ReAct loop is the foundational pattern of all agentic AI systems. The agent alternates between reasoning (deciding what to do) and acting (executing a tool call), with the results of each action feeding back into the next reasoning step.

```
while task_not_complete:
    reasoning = model.think(context + history)
    if reasoning.is_final_answer:
        return reasoning.answer
    action = reasoning.next_tool_call
    result = execute(action)
    history.append(action, result)
```

**Why It Works:** The ReAct loop converts the inherently one-shot nature of LLM inference into an iterative process. Each tool result provides new information that the model could not have predicted, enabling it to adapt its approach based on real-world feedback.

**Variations Across Systems:**

| System | Loop Constraint | Rationale |
|--------|----------------|-----------|
| Manus | One tool per iteration (strict) | Maximum auditability and determinism |
| OpenClaw | One tool per turn, serialized per session | Prevents state corruption |
| Cursor | Multiple tool calls per turn allowed | Speed for code editing workflows |
| Runner H | Policy → Localizer → Validator pipeline | Separates decision from execution from verification |
| Devin | Long-running autonomous loops | Complex multi-step development tasks |

**Refinement Suggestion for ØMEGA AI:** Implement a **hybrid loop** — strict single-tool-per-iteration for write operations (order placement, position modification) but allow parallel read operations (multiple market data fetches) for performance.

---

## 2. Gateway / Runtime Separation

**Observed in:** OpenClaw, Manus, Runner H

**Pattern Description:**
A Gateway (control plane) handles routing, session management, and authentication, while the Runtime (data plane) handles model inference and tool execution. The Gateway never directly invokes LLMs; the Runtime never directly handles user connections.

```
User → Gateway → [Routing, Auth, Session] → Runtime → [Context, LLM, Tools]
                                                ↓
User ← Gateway ← [Response Formatting]    ← Runtime ← [Results]
```

**Why It Works:** Separation of concerns enables independent scaling, security policy enforcement at the gateway level, and graceful degradation (the gateway can serve cached responses or error messages if the runtime fails).

**ØMEGA AI Implementation:**

| Component | Gateway Responsibilities | Runtime Responsibilities |
|-----------|------------------------|------------------------|
| **Signal Gateway** | Webhook ingestion, HMAC verification, rate limiting, routing | Signal analysis, scoring, recommendation generation |
| **Execution Gateway** | Order validation, risk limit enforcement, audit logging | Order construction, exchange API interaction |
| **Data Gateway** | Market data aggregation, caching, normalization | Indicator calculation, pattern recognition |

---

## 3. Context Assembly (Prompt Construction)

**Observed in:** All studied systems

**Pattern Description:**
Before every model inference, the system assembles a context package from multiple sources. The context is not a static prompt — it is dynamically constructed from base instructions, relevant skills/modules, conversation history, tool results, and environmental state.

**Context Assembly Components (Cross-System Comparison):**

| Component | OpenClaw | Manus | Cursor | Windsurf |
|-----------|----------|-------|--------|----------|
| **Base prompt** | AGENTS.md + SOUL.md + IDENTITY.md | System prompt + modules | Agent prompt | Cascade prompt |
| **Skill/capability list** | Compact skill names + descriptions | Tool descriptions | Available tools | Available tools |
| **Conversation history** | Full history with compaction | Full history | Recent context | Recent context |
| **Environmental state** | Bootstrap files, workspace state | Sandbox state, file system | Open files, cursor position | Open files, project structure |
| **Memory** | MEMORY.md + daily logs (on-demand) | File-based (in sandbox) | Project-level context | Project-level context |
| **Per-run overrides** | Task-specific instructions | Task-specific instructions | User settings | User settings |

**Why It Works:** Dynamic context assembly ensures the model always has the most relevant information for the current task, without wasting context window space on irrelevant instructions. It also enables the same base model to behave differently depending on the assembled context.

**The Critical Insight:** Context assembly is the most important engineering decision in any agentic system. Everything the model knows, believes, and can do flows through this stage. A vulnerability in context assembly (e.g., injected content) compromises the entire system.

**Refinement Suggestion:** The ØMEGA AI should implement a **context firewall** between assembly and inference:

```
[Base Prompt] + [Trading Rules] + [Active Strategy] + [Market State] + [History]
                                    ↓
                          [Context Firewall]
                    (Injection detection, integrity check)
                                    ↓
                          [Model Inference]
```

---

## 4. Lazy Loading / On-Demand Instruction Retrieval

**Observed in:** OpenClaw (skills), Manus (skills), Cursor (file reading), Devin (knowledge retrieval)

**Pattern Description:**
Rather than injecting all possible instructions into the system prompt, the agent receives a compact index of available capabilities and retrieves detailed instructions on demand when they become relevant.

**How It Works in OpenClaw:**
1. Context assembly injects a list: `[{name: "github-pr-reviewer", description: "Review PRs", path: "skills/github-pr-reviewer/SKILL.md"}, ...]`
2. When the model decides a skill is relevant, it reads the SKILL.md file
3. The detailed instructions are added to the current context
4. The model follows the skill's instructions for the current task

**Why It Works:** Context windows are finite. A system with 100 skills cannot inject all 100 SKILL.md files into every prompt. Lazy loading keeps the base context lean while allowing unlimited capability expansion.

**Refinement Suggestion:** The ØMEGA AI should implement lazy loading for trading strategies:

```
Base context: "Available strategies: [market-cipher-scalp, algopro-swing, 
vwap-mean-reversion, rbi-breakout, ...]. Read the strategy file when 
processing a signal that matches."

On signal arrival: Agent reads the relevant strategy SKILL.md and follows 
its specific entry/exit/sizing rules.
```

---

## 5. Tiered Approval Flow

**Observed in:** OpenClaw (AGENTS.md), Manus (confirmation gates)

**Pattern Description:**
Agent actions are categorized into tiers based on their impact. Low-impact actions execute autonomously, medium-impact actions require user confirmation, and high-impact actions are prohibited entirely.

**Cross-System Comparison:**

| Tier | OpenClaw | Manus | ØMEGA AI (Proposed) |
|------|----------|-------|------------------------|
| **Autonomous** | Read data, summarize, draft, research | Read files, search web, analyze | Fetch data, calculate indicators, score signals |
| **Requires Approval** | Send messages, schedule meetings, publish | Sensitive browser operations, deployments | Place orders, modify positions, change stops |
| **Prohibited** | DM strangers, make purchases, delete data | N/A (sandbox-isolated) | Transfer funds, change API keys, override risk limits |

**Why It Works:** Tiered approval balances autonomy with safety. Agents can be productive on routine tasks without human intervention, while high-impact actions always have a human in the loop.

**Refinement Suggestion:** Implement **dynamic tier adjustment** — during high-volatility periods, automatically elevate more actions to the "requires approval" tier.

---

## 6. File-Based Memory

**Observed in:** OpenClaw (MEMORY.md, daily logs), Manus (sandbox files), Cursor (project files)

**Pattern Description:**
Agent memory is stored as plain-text files (typically Markdown) rather than in databases. The agent reads and writes these files using standard file tools, and the files serve as both persistent memory and audit trail.

**Memory File Taxonomy:**

| File Type | OpenClaw Example | Purpose | Retention |
|-----------|-----------------|---------|-----------|
| **Identity** | SOUL.md, IDENTITY.md | Personality, boundaries | Permanent |
| **Operational** | AGENTS.md | Rules, approval flows | Permanent |
| **Long-term** | MEMORY.md | Facts, preferences, contacts | Permanent, updated |
| **Ephemeral** | memory/YYYY-MM-DD.md | Daily logs, decisions | Rolling window |
| **Task-specific** | Heartbeat checklist | Proactive task list | Updated per heartbeat |

**Why It Works:** File-based memory is simple, auditable, and portable. It requires no database infrastructure, can be version-controlled with git, and can be read by both humans and agents. The agent uses the same file tools it uses for everything else — no special memory API is needed.

**Security Concern:** File-based memory is vulnerable to tampering. Anyone with filesystem access can read or modify memory files, and prompt injection can cause the agent to write malicious content to persistent memory.

**Refinement Suggestion:** The ØMEGA AI should implement **encrypted, integrity-verified memory**:

```
Trade Journal (encrypted at rest):
├── strategies/          ← Active strategy configurations (signed)
├── positions/           ← Current position state (encrypted)
├── history/             ← Trade history (append-only, hashed)
├── signals/             ← Signal log with source attribution
└── performance/         ← Performance metrics (calculated, not stored)
```

---

## 7. Proactive Heartbeat

**Observed in:** OpenClaw (30-minute heartbeat), Manus (task-triggered only)

**Pattern Description:**
The agent is periodically woken up by a scheduled trigger (cron job, timer) and asked to evaluate whether any proactive action is needed. If nothing requires attention, the agent returns a silent acknowledgment.

**OpenClaw Heartbeat Flow:**
```
Timer fires (every 30 minutes)
    → Agent reads HEARTBEAT.md (proactive task checklist)
    → Agent evaluates each task
    → If action needed: execute and notify user
    → If nothing needed: return HEARTBEAT_OK (suppressed by Gateway)
```

**Why It Works:** Heartbeats transform agents from reactive (only responds to user messages) to proactive (monitors conditions and takes initiative). This is essential for any monitoring or operations use case.

**Refinement Suggestion:** The ØMEGA AI should implement **multi-frequency heartbeats**:

| Frequency | Purpose | Actions |
|-----------|---------|---------|
| 1 minute | Position health check | Verify stops, check liquidation distance |
| 5 minutes | Market monitoring | Check for significant price movements |
| 15 minutes | Signal aggregation | Compile and score pending signals |
| 30 minutes | Risk assessment | Portfolio-level risk calculation |
| 1 hour | Performance snapshot | Update P&L, win rate, drawdown metrics |
| Daily | End-of-day report | Comprehensive daily trading summary |

---

## 8. Structured Output (JSON Schema Enforcement)

**Observed in:** Manus (response_format), OpenAI (structured outputs), Cursor (tool schemas)

**Pattern Description:**
The model is constrained to produce output conforming to a strict JSON schema. This ensures type-safe communication between the agent and downstream systems.

**Why It Works:** Free-form text output is ambiguous and error-prone. When an agent needs to produce a trading signal, a structured schema ensures that every required field is present, every value is within expected ranges, and the output can be programmatically processed without parsing heuristics.

**ØMEGA AI Signal Schema:**
```json
{
  "type": "object",
  "properties": {
    "signal_id": { "type": "string", "format": "uuid" },
    "timestamp": { "type": "number" },
    "symbol": { "type": "string", "pattern": "^[A-Z]+/[A-Z]+$" },
    "direction": { "type": "string", "enum": ["long", "short"] },
    "confidence": { "type": "number", "minimum": 0, "maximum": 100 },
    "entry_price": { "type": "number", "minimum": 0 },
    "stop_loss": { "type": "number", "minimum": 0 },
    "take_profit": { "type": "number", "minimum": 0 },
    "position_size_pct": { "type": "number", "minimum": 0, "maximum": 5 },
    "strategy": { "type": "string" },
    "reasoning": { "type": "string" },
    "source_signals": { "type": "array", "items": { "type": "string" } }
  },
  "required": ["signal_id", "timestamp", "symbol", "direction", "confidence", "strategy"],
  "additionalProperties": false
}
```

---

## 9. Specialized Sub-Agents

**Observed in:** Manus (debugging agent), OpenClaw (multi-agent routing), Devin (planning + execution agents)

**Pattern Description:**
Complex tasks are decomposed and delegated to specialized sub-agents, each with their own context, tools, and behavioral rules. A coordinator agent manages the workflow and aggregates results.

**Why It Works:** No single prompt can make a model expert at everything. Specialized sub-agents can have focused system prompts, limited tool sets, and domain-specific knowledge, producing better results than a generalist agent.

**ØMEGA AI Sub-Agent Architecture:**

| Agent | Specialization | Tools | Model |
|-------|---------------|-------|-------|
| **Signal Analyst** | Interpret trading signals, score confidence | Market data, indicator calculation | Fast model (low latency) |
| **Risk Manager** | Calculate position sizes, enforce limits | Risk engine, portfolio state | Precise model (high accuracy) |
| **Order Executor** | Construct and place orders | Exchange API, order management | Fast model (low latency) |
| **Chart Annotator** | Mark entry/exit levels on charts | Chart rendering, annotation tools | Visual model (multimodal) |
| **Research Analyst** | Fundamental analysis, news synthesis | Web search, data APIs | Deep reasoning model |
| **Performance Auditor** | Analyze trade history, suggest improvements | Trade journal, statistics | Analytical model |

---

## 10. Save-Before-Execute

**Observed in:** Manus (mandatory), Cursor (file-first workflow), Devin (workspace-based)

**Pattern Description:**
All code, configurations, and executable content must be saved to files before execution. Direct code input to interpreters is forbidden.

**Why It Works:** This pattern provides auditability (every executed piece of code exists as a reviewable file), reproducibility (code can be re-run), debugging (errors reference specific files and lines), and version control (file modifications are tracked).

**Refinement Suggestion:** Every trading strategy, risk calculation, and order construction in the ØMEGA AI should exist as a versioned file. This creates a complete audit trail for regulatory compliance and performance analysis.

---

## 11. Active Information Persistence

**Observed in:** Manus (save key info to files), OpenClaw (daily logs)

**Pattern Description:**
During research, browsing, or data processing, the agent actively saves key findings to persistent files rather than relying on conversation context alone.

**Why It Works:** Context windows are finite, and information can be lost during context compaction. By explicitly persisting important findings, the agent creates a durable knowledge base that survives context limits.

**Refinement Suggestion:** When the ØMEGA AI analyzes a chart or processes market data, key findings should be persisted to structured files:

```
analysis/2026-03-11/
├── BTC-USDT-4h.md      ← Chart analysis (support/resistance, trend, patterns)
├── signals-scored.json  ← Scored signals with reasoning
├── risk-assessment.json ← Portfolio risk state
└── recommendations.md   ← Trading recommendations with confidence levels
```

---

## 12. Pattern Interaction Map

These patterns do not exist in isolation — they interact and reinforce each other:

| Pattern | Depends On | Enables |
|---------|-----------|---------|
| ReAct Loop | Tool system | All agent behavior |
| Gateway/Runtime | — | Security, scaling, routing |
| Context Assembly | Memory, Skills | ReAct Loop quality |
| Lazy Loading | Context Assembly | Unlimited capability expansion |
| Tiered Approval | Gateway | Safe autonomous operation |
| File-Based Memory | File tools | Context Assembly, Persistence |
| Heartbeat | ReAct Loop, Memory | Proactive behavior |
| Structured Output | — | Reliable inter-agent communication |
| Sub-Agents | Gateway/Runtime | Task decomposition |
| Save-Before-Execute | File tools | Auditability, debugging |
| Active Persistence | File tools | Knowledge durability |

---

## References

[1]: https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools "System Prompts and Models of AI Tools — x1xhlol"
[2]: https://github.com/dontriskit/awesome-ai-system-prompts "Awesome AI System Prompts — dontriskit"
[3]: https://github.com/asgeirtj/system_prompts_leaks "System Prompts Leaks — asgeirtj"
[4]: https://github.com/jujumilk3/leaked-system-prompts "Leaked System Prompts — jujumilk3"
[5]: https://bibek-poudel.medium.com/how-openclaw-works-understanding-ai-agents-through-a-real-architecture-5d59cc7a4764 "How OpenClaw Works — Bibek Poudel"
[6]: https://www.hcompany.ai/ "H Company — Runner H / Surfer H"
[7]: https://vercept.com/ "Vercept — Vy"
