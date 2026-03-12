# Omega System — Rough Architecture Research

> **Organization:** Black Wealth Capital
> **Project:** Omega System — AI-Powered Trading Agent Platform
> **Status:** Rough Draft / Research Phase
> **Last Updated:** March 2026

---

## What Is This Repository?

This repository contains the foundational research for the **Omega System**, an AI-powered trading agent platform designed for Black Wealth Capital. It synthesizes architectural analysis from four major AI agent systems — **OpenClaw**, **Manus**, **Runner H/Surfer H** (H Company), and **Vy** (Vercept/Anthropic) — along with security analyses from CrowdStrike, Trend Micro, Cisco, Snyk, NSFOCUS, Oasis Security, Microsoft, and Giskard, and leaked system prompts from 14+ AI platforms.

The goal is not to exploit these prompts, but to **understand the engineering patterns** behind the most capable AI systems in the world and apply those patterns — with appropriate security hardening — to our own trading and analysis platform.

---

## Table of Contents

| Section | Description |
|---------|-------------|
| [Architecture Overview](docs/architecture-overview.md) | Comparative analysis of agent architectures across all studied systems |
| [Security Guardrails Framework](security/guardrails-framework.md) | 5-layer defense-in-depth security framework with trading-specific controls |
| [OpenClaw Deep Dive](architectures/openclaw-architecture.md) | Gateway, agentic loop, skills, memory, heartbeat — with security hardening |
| [Manus Deep Dive](architectures/manus-architecture.md) | Sandbox model, 29 tools, module system, sub-agents |
| [Runner H & Surfer H](architectures/runner-h-surfer-h.md) | H Company's browser-use agent with Hollow One VLMs |
| [Vy by Vercept](architectures/vy-vercept.md) | Visual-first computer use agent (now acquired by Anthropic) |
| [Agent Design Patterns](docs/agent-design-patterns.md) | 12 recurring patterns across all studied systems |
| [Prompt Engineering Taxonomy](docs/prompt-taxonomy.md) | Classification of system prompt techniques from 123 prompts |
| [OpenClaw Vulnerabilities](security/openclaw-vulnerabilities.md) | Complete vulnerability catalog from 8 security firms |
| [Prompt Injection Taxonomy](security/prompt-injection-taxonomy.md) | Classification of injection techniques and defenses |

---

## Repository Structure

```
omega-system-rough/
├── README.md                                    ← This file
├── architectures/                               ← Deep-dive architecture documents
│   ├── openclaw-architecture.md                 ← OpenClaw: 7 components, security analysis
│   ├── manus-architecture.md                    ← Manus: sandbox model, 29 tools, modules
│   ├── runner-h-surfer-h.md                     ← H Company: Hollow One VLMs, browser-use
│   └── vy-vercept.md                            ← Vercept/Anthropic: visual-first, workflow learning
├── security/                                    ← Security framework and vulnerability analysis
│   ├── guardrails-framework.md                  ← 5-layer defense-in-depth framework
│   ├── openclaw-vulnerabilities.md              ← All known CVEs and exploits
│   └── prompt-injection-taxonomy.md             ← Classification of injection techniques
├── docs/                                        ← Cross-cutting analysis documents
│   ├── architecture-overview.md                 ← Comparative analysis of all systems
│   ├── agent-design-patterns.md                 ← 12 recurring patterns across all agents
│   └── prompt-taxonomy.md                       ← System prompt engineering techniques
└── prompts/                                     ← Raw leaked system prompts (source material)
    ├── anthropic/                               ← Claude system prompts
    ├── openai/                                  ← GPT-5, ChatGPT, Deep Research prompts
    ├── google/                                  ← Gemini system prompts
    ├── xai/                                     ← Grok system prompts
    ├── meta/                                    ← Meta AI system prompts
    ├── manus/                                   ← Manus agent loop, modules, tools
    ├── openclaw/                                ← OpenClaw AGENTS.md, SOUL.md, IDENTITY.md
    ├── cursor/                                  ← Cursor IDE agent prompts
    ├── windsurf/                                ← Windsurf/Cascade agent prompts
    ├── v0/                                      ← v0 UI generation prompts
    ├── lovable/                                 ← Lovable full-stack generation prompts
    ├── devin/                                   ← Devin coding agent prompts
    ├── emergent/                                ← Emergent agent prompts
    └── misc/                                    ← Perplexity, Notion AI, GitHub Copilot, etc.
```

---

## Prompt Collection

System prompts are organized by provider/platform. Each directory contains the raw prompts sourced from four major repositories plus independent research.

### Foundation Models

| Provider | Directory | Key Prompts |
|----------|-----------|-------------|
| Anthropic (Claude) | [`prompts/anthropic/`](prompts/anthropic/) | Claude Opus 4.x, Sonnet 4.x, Haiku 4.5, Claude Code, Claude Artifacts |
| OpenAI (GPT/o-series) | [`prompts/openai/`](prompts/openai/) | GPT-5.x, o3, o4-mini, ChatGPT Agent Mode, Codex, Deep Research |
| Google (Gemini) | [`prompts/google/`](prompts/google/) | Gemini 3.x Pro/Flash, Gemini Workspace, NotebookLM |
| Meta | [`prompts/meta/`](prompts/meta/) | Meta AI WhatsApp |
| xAI (Grok) | [`prompts/xai/`](prompts/xai/) | Grok 2, Grok 3 |

### AI Coding Agents

| Platform | Directory | Key Prompts |
|----------|-----------|-------------|
| Cursor | [`prompts/cursor/`](prompts/cursor/) | Agent v2.0, CLI Agent, Chat Mode |
| Windsurf (Codeium) | [`prompts/windsurf/`](prompts/windsurf/) | Cascade Wave 11, R1 variant |
| v0 (Vercel) | [`prompts/v0/`](prompts/v0/) | Full prompt + tools |
| Lovable | [`prompts/lovable/`](prompts/lovable/) | Agent prompt + tools |
| Devin | [`prompts/devin/`](prompts/devin/) | Full prompt, DeepWiki |
| Emergent | [`prompts/emergent/`](prompts/emergent/) | Agent prompt + tools |

### AI Agent Platforms

| Platform | Directory | Key Prompts |
|----------|-----------|-------------|
| Manus | [`prompts/manus/`](prompts/manus/) | Full system prompt, agent loop, modules, tools (29 tool schemas) |
| OpenClaw | [`prompts/openclaw/`](prompts/openclaw/) | AGENTS.md, SOUL.md, IDENTITY.md, skill system |
| Misc | [`prompts/misc/`](prompts/misc/) | Perplexity, Notion AI, GitHub Copilot, Discord Clyde, and others |

---

## Architecture Studies

Each architecture document follows a consistent structure: executive summary, component analysis, security assessment, comparison with other systems, and **refinement suggestions for the Omega System**.

| Document | Focus | Key Insights |
|----------|-------|-------------|
| [OpenClaw Architecture](architectures/openclaw-architecture.md) | Gateway → Channel Adapters → Routing → Agentic Loop → Skills → MCP → Memory → Heartbeat | 7-component architecture, proactive heartbeat system, multi-channel operations, tiered approval flows. Security analysis from 8 firms identifies RCE, credential theft, skill poisoning. |
| [Manus Architecture](architectures/manus-architecture.md) | Agent loop, sandbox execution, tool orchestration, MCP integration | Cloud sandbox model, 29 curated tools, XML-tagged module system, save-before-execute pattern, specialized sub-agents (debugging agent), structured JSON output. |
| [Runner H & Surfer H](architectures/runner-h-surfer-h.md) | Policy → Localizer → Validator → Memory pipeline for browser-use agents | Hollow One VLMs purpose-built for browser interaction, visual understanding vs. text extraction, cloud vs. local deployment tradeoffs. |
| [Vy by Vercept](architectures/vy-vercept.md) | Visual-first, privacy-centric computer use agent | Screenshot-based interaction with any GUI application, workflow learning by demonstration, privacy-centric design (local processing, no data retention), Anthropic acquisition implications. |

---

## Security Analysis

| Document | Focus | Key Insights |
|----------|-------|-------------|
| [Security Guardrails Framework](security/guardrails-framework.md) | 5-layer defense-in-depth for AI agents | Input validation → Runtime controls → Output filtering → Infrastructure isolation → Operational monitoring. Trading-specific guardrails: order limits, risk controls, signal authentication, kill switch. Implementation priority matrix and incident response playbooks. |
| [OpenClaw Vulnerabilities](security/openclaw-vulnerabilities.md) | Complete vulnerability catalog | RCE (NSFOCUS), website-to-agent takeover (Oasis), credential exfiltration (Cisco), 13.4% of ClawHub skills critical (Snyk), memory poisoning (Microsoft), session leakage (Giskard), in-the-wild crypto drain attempt (CrowdStrike). |
| [Prompt Injection Taxonomy](security/prompt-injection-taxonomy.md) | Classification of injection techniques | Direct (override, role manipulation, delimiter escape), indirect (web content, data feed, email), persistent (memory poisoning, skill poisoning), evasion (encoding, fragmentation). Trading-specific: signal poisoning via webhook injection. |

---

## Cross-Cutting Analysis

| Document | Focus | Key Insights |
|----------|-------|-------------|
| [Architecture Overview](docs/architecture-overview.md) | Comparative analysis across all systems | Side-by-side comparison across 10 dimensions, what each system does best, hybrid architecture recommendation for Omega System, 16-week implementation roadmap. |
| [Agent Design Patterns](docs/agent-design-patterns.md) | 12 recurring patterns | ReAct Loop, Gateway/Runtime Separation, Context Assembly, Lazy Loading, Tiered Approval, File-Based Memory, Proactive Heartbeat, Structured Output, Specialized Sub-Agents, Save-Before-Execute, Active Persistence. Pattern interaction map. |
| [Prompt Taxonomy](docs/prompt-taxonomy.md) | System prompt engineering techniques | Analysis of 123 prompts from 14 platforms: identity definition, behavioral constraints, tool use instructions, context management, safety techniques, domain-specific patterns. Omega System prompt architecture template. |

---

## Key Findings

### 1. The Security Gap Is Real

OpenClaw has been analyzed by CrowdStrike, Trend Micro, Cisco, Snyk, NSFOCUS, Oasis Security, Microsoft, and Giskard. Every analysis found critical vulnerabilities. The most alarming finding: **13.4% of community skills on ClawHub contain critical security issues** (Snyk), and **CrowdStrike documented a real-world crypto wallet drain attempt** using prompt injection on the Moltbook social network. For a financial trading system, these vulnerabilities are existential.

### 2. Sandbox Isolation Is Non-Negotiable

Manus's sandbox model is the only architecture that provides meaningful isolation between the agent and the user's environment. For the Omega System, each agent type must run in its own isolated container with only the specific permissions it needs. A compromised signal analysis agent must never have access to order execution credentials.

### 3. Purpose-Built Models Outperform General-Purpose Models

H Company's Hollow One VLMs, trained specifically for browser interaction, outperform general-purpose models on browser tasks. The same principle applies to trading: a model fine-tuned on chart analysis will outperform a general-purpose model using prompt engineering alone.

### 4. The ReAct Loop Is Universal but Insufficient

Every agent system implements some form of the ReAct loop (Reason + Act). But the loop alone does not guarantee safety. The Omega System needs additional layers: input validation before reasoning, tool-level access control during action, output filtering after response, and behavioral monitoring across the entire loop.

### 5. File-Based Memory Is Simple but Vulnerable

OpenClaw's file-based memory (MEMORY.md, SOUL.md) is elegant and auditable, but Microsoft's analysis shows it is vulnerable to persistent prompt injection. The Omega System must implement encrypted, integrity-verified memory with write validation.

---

## Omega System Design Principles

Based on all research, the Omega System should be built on these principles:

1. **Capital preservation is the highest priority.** Every architectural decision should be evaluated against this principle.
2. **Defense in depth.** Five layers of defense: input validation, runtime controls, output filtering, infrastructure isolation, and operational monitoring.
3. **Least privilege.** Each agent type has only the tools and credentials it needs. The Signal Analyst cannot place orders. The Order Executor cannot modify risk parameters.
4. **Auditability.** Every decision, every tool call, every order must be logged in an immutable audit trail.
5. **Graceful degradation.** When something fails, the system fails safely. A failed signal analysis does not trigger a trade.
6. **Human in the loop for critical actions.** Signal analysis can be autonomous. Order placement requires confirmation.

---

## How This Feeds the Omega System

These patterns will be refined and applied to build the Omega System's specialized agents:

| Agent | Role | Inspiration |
|-------|------|-------------|
| **Signal Analyst** | Processes Market Cipher + AlgoPro webhook signals | OpenClaw's skill system + Manus's structured output |
| **Risk Manager** | Applies Kelly Criterion, Monte Carlo validation | Manus's module system + custom guardrails |
| **Chart Annotator** | Annotates Lightweight Charts with entry/exit levels | Runner H's visual understanding + Vy's screenshot analysis |
| **Order Executor** | Manages positions with TP/SL/trailing stops | Manus's confirmation gates + OpenClaw's tiered approval |
| **Research Analyst** | Implements RBI (Research, Backtest, Implement) methodology | OpenClaw's proactive heartbeat + Manus's web research tools |

---

## Sources and Attribution

### Security Analysis Sources

| Source | Organization | Focus | URL |
|--------|-------------|-------|-----|
| CrowdStrike | CrowdStrike Intelligence | Attack chain, wild exploit | [Link](https://www.crowdstrike.com/en-us/blog/what-security-teams-need-to-know-about-openclaw-ai-super-agent/) |
| Trend Micro | Trend Micro Research | CISO perspective, enterprise risk | [Link](https://www.trendmicro.com/en_us/research/26/c/cisos-in-a-pinch-a-security-analysis-openclaw.html) |
| Cisco | Cisco AI Security | Credential exposure, network risk | [Link](https://blogs.cisco.com/ai/personal-ai-agents-like-openclaw-are-a-security-nightmare) |
| Snyk | Snyk Security Research | ClawHub skill poisoning (13.4% critical) | [Link](https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/) |
| NSFOCUS | NSFOCUS Global | RCE vulnerabilities | [Link](https://nsfocusglobal.com/openclaw-security-issues-add-a-security-guardrail-to-your-ai-application/) |
| Oasis Security | Oasis Security Research | Website-to-agent takeover | [Link](https://www.oasis.security/blog/openclaw-vulnerability) |
| Microsoft | Microsoft Security Blog | Identity, isolation, runtime risk | [Link](https://www.microsoft.com/en-us/security/blog/2026/02/19/running-openclaw-safely-identity-isolation-runtime-risk/) |
| Giskard | Giskard AI Safety | Data leakage, prompt injection | [Link](https://www.giskard.ai/knowledge/openclaw-security-vulnerabilities-include-data-leakage-and-prompt-injection-risks) |

### Architecture Sources

| Source | Focus | URL |
|--------|-------|-----|
| Bibek Poudel (Medium) | OpenClaw architecture walkthrough | [Link](https://bibek-poudel.medium.com/how-openclaw-works-understanding-ai-agents-through-a-real-architecture-5d59cc7a4764) |
| H Company | Runner H / Surfer H official | [Link](https://www.hcompany.ai/) |
| Runner H Masterclass (Medium) | Detailed architecture analysis | [Link](https://medium.com/@kram254/runner-h-surfer-h-a-masterclass-in-modern-browser-use-agents-fb68cb666b29) |
| AI Adoption Agency | Vercept / Vy analysis | [Link](https://aiadoptionagency.com/vercept-and-vy-redefining-human-computer-interaction-through-ai/) |
| Vercept | Anthropic acquisition announcement | [Link](https://vercept.com/) |

### System Prompt Leak Repositories

| Repository | Stars | Prompts | URL |
|-----------|-------|---------|-----|
| x1xhlol/system-prompts-and-models-of-ai-tools | 20k+ | Manus, Cursor, Windsurf, v0, Lovable, Devin | [Link](https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools) |
| dontriskit/awesome-ai-system-prompts | 5k+ | Claude, GPT-5, Gemini, Grok, OpenClaw | [Link](https://github.com/dontriskit/awesome-ai-system-prompts) |
| asgeirtj/system_prompts_leaks | 2k+ | Anthropic, OpenAI, Google, xAI, Meta | [Link](https://github.com/asgeirtj/system_prompts_leaks) |
| jujumilk3/leaked-system-prompts | 10k+ | 50+ AI systems | [Link](https://github.com/jujumilk3/leaked-system-prompts) |

---

## TLDR: Step-by-Step Guide

### What This Repo Contains
1. **4 architecture deep dives** — OpenClaw, Manus, Runner H/Surfer H, Vy/Vercept — each with component analysis, security assessment, and Omega System refinement suggestions
2. **3 security documents** — 5-layer guardrails framework, complete OpenClaw vulnerability catalog, prompt injection taxonomy
3. **3 cross-cutting analyses** — comparative architecture overview, 12 agent design patterns, prompt engineering taxonomy from 123 prompts across 14 platforms
4. **Raw system prompts** — from 14+ AI platforms, organized by provider

### How to Use This Research
1. **Start with** `docs/architecture-overview.md` for the comparative analysis and hybrid architecture recommendation
2. **Read** `docs/agent-design-patterns.md` to understand the 12 patterns that every agent system implements
3. **Deep dive** into individual architectures (`architectures/`) for the systems most relevant to your implementation
4. **Study** `security/guardrails-framework.md` for the complete security architecture, including trading-specific controls
5. **Reference** `security/openclaw-vulnerabilities.md` to understand what can go wrong and how to prevent it
6. **Browse** `prompts/` for raw system prompt examples when implementing specific agent behaviors

### Next Steps for the Omega System
1. **Design the Signal Gateway** — TradingView webhook ingestion with HMAC verification and schema validation
2. **Implement the core agent loop** — ReAct pattern with hybrid concurrency (parallel reads, serialized writes)
3. **Build the Risk Manager** — Position sizing, portfolio limits, drawdown circuit breakers
4. **Build the Order Executor** — Exchange API integration with confirmation gates and execution guardrails
5. **Implement the security framework** — Start with P0 controls (input validation, tool ACLs, kill switch), then P1, P2, P3
6. **Set up monitoring** — Comprehensive logging, real-time alerting, audit trail
7. **Conduct red team exercises** — Test prompt injection, signal poisoning, credential exfiltration
8. **Iterate** — This is a rough draft; refine based on implementation experience

---

## License

This repository is for **educational and research purposes only**. System prompts are sourced from publicly available repositories. All original analysis and documentation is copyright Black Wealth Capital.
