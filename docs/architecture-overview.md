# Architecture Overview: Comparative Analysis of AI Agent Systems

> **Author:** Black Wealth Capital Research Division
> **Status:** Rough Draft — Omega System Foundation Document
> **Last Updated:** March 2026

---

## Executive Summary

This document provides a unified comparative analysis of four major AI agent architectures — OpenClaw, Manus, Runner H/Surfer H (H Company), and Vy (Vercept/Anthropic) — along with supplementary analysis of coding agents (Cursor, Windsurf, v0, Lovable, Devin) and foundation models (Claude, GPT-5, Gemini, Grok). The goal is to identify which architectural patterns, security models, and design decisions should be adopted, adapted, or avoided in the **Omega System**.

The Omega System is a **hierarchical multi-agent platform** consisting of 25 specialized agents organized into three tiers: one supreme commander (PRIME), five meta agents (LOOM, WARDEN, MAESTRO, PHANTOM, SIGMA), ten core operations agents (ARCANE through MODULUS), and nine specialist agents (RELIC through SYNTH). This document synthesizes the best practices from studied systems to inform Omega's architecture, security model, and agent coordination patterns.

---

## 1. System Classification Matrix

| System | Category | Execution Environment | Primary Interface | Model Type | Open Source |
|--------|----------|----------------------|-------------------|-----------|-------------|
| **OpenClaw** | General-purpose agent | Local machine | Multi-channel (WhatsApp, Telegram, etc.) | User-configured (Anthropic, OpenAI, Google, Ollama) | Yes |
| **Manus** | General-purpose agent | Cloud sandbox (Ubuntu VM) | Web interface | Platform-managed | No |
| **Runner H** | Browser-use agent | Cloud headless browser | API | Hollow One (proprietary VLM) | No |
| **Surfer H** | Browser-use agent | Local browser (Chrome extension) | Chrome extension | Hollow One (proprietary VLM) | No |
| **Vy (Vercept)** | Computer-use agent | Local desktop (macOS/Windows) | Desktop app | Proprietary VLM | No |
| **Cursor** | Coding agent | Local IDE | VS Code extension | Claude/GPT (user-selected) | No |
| **Windsurf** | Coding agent | Local IDE | Custom IDE | Cascade (proprietary) | No |
| **Devin** | Coding agent | Cloud sandbox | Web interface | Proprietary | No |
| **v0** | UI generation agent | Cloud | Web interface | Proprietary | No |
| **Lovable** | Full-stack generation agent | Cloud | Web interface | Proprietary | No |

---

## 2. Architectural Dimension Comparison

### 2.1 Execution Model

| Dimension | OpenClaw | Manus | Runner H | Vy |
|-----------|----------|-------|----------|-----|
| **Where it runs** | User's machine | Cloud VM | Cloud browser | User's desktop |
| **Isolation** | None (full host access) | Full sandbox | Browser sandbox | None (full desktop access) |
| **Persistence** | Files in ~/.openclaw/ | Files in sandbox (survives hibernation) | Per-task (destroyed after) | Local app data |
| **Internet access** | Full | Full | Full | Full |
| **Local file access** | Full | None (sandbox only) | None (browser only) | Full (visual only) |
| **Tool execution** | Shell, browser, MCP, skills | Shell, browser, MCP, file tools | Browser actions only | Mouse, keyboard, visual |

**Analysis:** The execution model is the single most important architectural decision. It determines the security boundary, the capability scope, and the user experience. For the Omega System, a **cloud sandbox model** (like Manus) provides the best security for financial operations, while a **local extension model** (like Surfer H) provides the lowest latency for time-sensitive trading.

### 2.2 Tool Architecture

| Dimension | OpenClaw | Manus | Runner H | Vy |
|-----------|----------|-------|----------|-----|
| **Core tools** | ~15 built-in | 29 curated | ~8 browser actions | ~5 desktop actions |
| **Extensibility** | Unlimited (skills + MCP) | MCP servers | None (fixed set) | Workflow learning |
| **Tool discovery** | Lazy loading from skill index | Full descriptions in prompt | Built into model | Visual element detection |
| **Tool validation** | Minimal | Schema validation | Model-internal | Visual verification |
| **Concurrency** | Serialized per session | One per iteration (strict) | Pipeline (policy→localizer→validator) | Sequential |

### 2.3 Memory Architecture

| Dimension | OpenClaw | Manus | Runner H | Vy |
|-----------|----------|-------|----------|-----|
| **Long-term memory** | MEMORY.md (persistent) | Files in sandbox | None (per-task) | Workflow recordings |
| **Short-term memory** | Conversation history | Conversation history | Action history | Screen history |
| **Identity memory** | SOUL.md, IDENTITY.md | System prompt | N/A | User preferences |
| **Operational memory** | AGENTS.md, daily logs | Module configurations | N/A | Scheduled tasks |
| **Memory format** | Markdown files | Various file formats | Structured data | Visual recordings |

### 2.4 Security Model

| Dimension | OpenClaw | Manus | Runner H | Vy |
|-----------|----------|-------|----------|-----|
| **Isolation** | None | Full sandbox | Browser sandbox | None |
| **Credential storage** | Plaintext files | Environment variables | Per-task API | Existing sessions |
| **Input validation** | Minimal | Schema validation | Model-internal | N/A |
| **Output filtering** | None by default | Sandbox boundary | API response filtering | N/A |
| **Approval flow** | Tiered (autonomous/confirmed/prohibited) | Confirmation for sensitive ops | Task-level approval | User watches in real-time |
| **Known CVEs** | Multiple (RCE, takeover, exfiltration) | None published | None published | None published |

---

## 3. What Each System Does Best

### 3.1 OpenClaw: Best Multi-Channel Operations Agent

OpenClaw's strength is its ability to operate across multiple communication channels (WhatsApp, Telegram, Slack, Discord, email) with a unified agent identity. No other system matches this breadth of channel integration. The proactive heartbeat system enables monitoring and operations use cases that purely reactive agents cannot handle.

**What to adopt:** Multi-channel architecture, proactive heartbeat, tiered approval flows, file-based memory with daily logs, skill system for extensibility.

**What to avoid:** Plaintext credential storage, unrestricted shell access, lack of input validation, lack of output filtering.

### 3.2 Manus: Best Sandboxed Development Agent

Manus's strength is its sandboxed execution environment combined with a comprehensive tool set. The sandbox provides security guarantees that no local agent can match, and the web development workflow demonstrates how specialized capabilities can be layered on top of a general-purpose agent.

**What to adopt:** Sandbox isolation, one-tool-per-iteration for write operations, module system for behavioral rules, save-before-execute pattern, specialized sub-agents (debugging agent), structured JSON output.

**What to avoid:** Cloud-only execution (adds latency), platform lock-in, limited extensibility compared to skill systems.

### 3.3 Runner H: Best Browser Automation Agent

Runner H's strength is its purpose-built VLM (Hollow One) that understands web pages visually. The Policy → Localizer → Validator pipeline separates decision-making from execution from verification, producing more reliable browser automation than any general-purpose agent.

**What to adopt:** Policy → Localizer → Validator pipeline, purpose-built models for domain-specific tasks, visual understanding of UI elements, structured action history.

**What to avoid:** Browser-only scope (too narrow for trading), cloud-only execution for Surfer H use cases.

### 3.4 Vy (Vercept): Best Desktop Automation Agent

Vy's strength is its visual-first architecture that works with any GUI application. Workflow learning enables non-technical users to teach the agent new capabilities by demonstration. The privacy-centric design (local processing, no data retention) is a model for handling sensitive financial data.

**What to adopt:** Visual-first chart analysis, workflow learning for trading processes, privacy-centric design, cross-application workflows, scheduled automation.

**What to avoid:** Full desktop access without isolation, reliance on visual-only interaction (slower than API-based).

---

## 4. Omega System Architecture Recommendation

Based on the comparative analysis, the Omega System should implement a **hybrid architecture** that combines the best patterns from each system:

### 4.1 Core Architecture

| Component | Inspiration | Implementation |
|-----------|-------------|---------------|
| **Execution environment** | Manus (sandbox) | Each agent type runs in its own isolated container |
| **Agent loop** | All systems (ReAct) | Hybrid: parallel reads, serialized writes |
| **Tool system** | OpenClaw (skills) + Manus (curated tools) | Curated core tools + extensible skill system |
| **Memory** | OpenClaw (file-based) + Manus (structured) | Encrypted file-based memory with integrity verification |
| **Communication** | OpenClaw (multi-channel) | TradingView webhooks, Telegram alerts, web dashboard |
| **Proactive behavior** | OpenClaw (heartbeat) | Multi-frequency heartbeats for different monitoring needs |
| **Security** | Manus (sandbox) + custom guardrails | Defense-in-depth with trading-specific controls |
| **Visual analysis** | Runner H (VLM) + Vy (visual-first) | Chart screenshot analysis with specialized model |

### 4.2 Agent Specialization

| Agent | Model Type | Tools | Approval Tier |
|-------|-----------|-------|---------------|
| **Signal Analyst** | Fast reasoning model | Market data (read), indicators, scoring | Autonomous |
| **Risk Manager** | Precise reasoning model | Risk engine, portfolio state | Autonomous (advisory) |
| **Order Executor** | Fast model | Exchange API (write) | Requires confirmation |
| **Chart Annotator** | Multimodal VLM | Chart rendering, annotation | Autonomous |
| **Research Analyst** | Deep reasoning model | Web search, data APIs | Autonomous |
| **Performance Auditor** | Analytical model | Trade journal, statistics | Autonomous |

### 4.3 Security Architecture

| Layer | Controls |
|-------|----------|
| **Input** | Signal authentication (HMAC), schema validation, injection detection, rate limiting |
| **Runtime** | Tool-level ACLs per agent, parameter validation, behavioral monitoring, confirmation gates |
| **Output** | Credential scanning, data classification, exfiltration detection |
| **Infrastructure** | Container isolation per agent, network segmentation, credential isolation, encryption |
| **Operational** | Comprehensive logging, real-time alerting, audit trail, kill switch, incident response |

---

## 5. Implementation Roadmap

### Phase 1: Foundation (Weeks 1-4)
- Implement core agent loop with ReAct pattern
- Set up containerized execution environments
- Implement Signal Gateway with webhook ingestion and HMAC verification
- Build basic Signal Analyst agent with market data tools
- Implement kill switch and basic monitoring

### Phase 2: Risk and Execution (Weeks 5-8)
- Implement Risk Manager agent with position sizing
- Implement Order Executor agent with exchange API integration
- Build tiered approval flow (autonomous signals → confirmed orders)
- Implement order execution guardrails (max size, price sanity, rate limits)
- Set up comprehensive logging and audit trail

### Phase 3: Intelligence (Weeks 9-12)
- Implement Chart Annotator agent with visual analysis
- Build Research Analyst agent with web search
- Implement multi-frequency heartbeat system
- Add Telegram notification channel
- Build web dashboard for monitoring

### Phase 4: Hardening (Weeks 13-16)
- Implement prompt injection detection
- Build behavioral monitoring and anomaly detection
- Add memory integrity verification
- Conduct red team exercises
- Performance optimization and load testing

---

## References

[1]: https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools "System Prompts and Models of AI Tools — x1xhlol"
[2]: https://github.com/dontriskit/awesome-ai-system-prompts "Awesome AI System Prompts — dontriskit"
[3]: https://github.com/asgeirtj/system_prompts_leaks "System Prompts Leaks — asgeirtj"
[4]: https://github.com/jujumilk3/leaked-system-prompts "Leaked System Prompts — jujumilk3"
[5]: https://www.crowdstrike.com/en-us/blog/what-security-teams-need-to-know-about-openclaw-ai-super-agent/ "CrowdStrike OpenClaw Analysis"
[6]: https://bibek-poudel.medium.com/how-openclaw-works-understanding-ai-agents-through-a-real-architecture-5d59cc7a4764 "How OpenClaw Works — Bibek Poudel"
[7]: https://medium.com/@kram254/runner-h-surfer-h-a-masterclass-in-modern-browser-use-agents-fb68cb666b29 "Runner H & Surfer H — Masterclass"
[8]: https://aiadoptionagency.com/vercept-and-vy-redefining-human-computer-interaction-through-ai/ "Vercept and Vy — AI Adoption Agency"
