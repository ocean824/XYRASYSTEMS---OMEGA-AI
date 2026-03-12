# Manus Architecture: Deep Dive

> **Author:** Black Wealth Capital Research Division
> **Status:** Rough Draft — Omega System Foundation Document
> **Last Updated:** March 2026

---

## Executive Summary

Manus is a general-purpose AI agent platform developed by the Manus team that operates within a sandboxed virtual machine environment with internet access. Unlike OpenClaw, which runs locally on user hardware, Manus executes entirely in a cloud-hosted sandbox — an Ubuntu 22.04 Linux environment with pre-installed development tools, a Chromium browser, and full internet connectivity [1]. This architectural choice fundamentally shapes its security model: the sandbox provides isolation that local agents lack, but introduces dependency on cloud infrastructure.

This document provides a comprehensive analysis of Manus's architecture based on leaked system prompts, agent loop definitions, module configurations, and tool schemas sourced from multiple public repositories [2] [3] [4]. It synthesizes these sources into a unified architectural understanding and provides refinement suggestions for the Omega System.

---

## 1. High-Level Architecture

Manus follows a **Cloud Sandbox + Agent Loop** architecture. The user communicates through a web interface, and the agent operates within an isolated virtual machine that persists across sessions.

### 1.1 The Sandbox Environment

The sandbox is the defining architectural feature of Manus. Every task executes within a clean, isolated Ubuntu 22.04 environment with the following characteristics [1]:

| Component | Specification | Significance |
|-----------|--------------|--------------|
| **OS** | Ubuntu 22.04 (linux/amd64) | Standard Linux environment with full package management |
| **User** | `ubuntu` with sudo privileges | Can install any software, modify system configuration |
| **Python** | 3.11.x with pip | Pre-installed with numpy, pandas, matplotlib, requests, etc. |
| **Node.js** | 22.x with pnpm | Full JavaScript/TypeScript development environment |
| **Browser** | Chromium stable | Persistent login state across tasks |
| **Internet** | Full access | Can reach any public API, website, or service |
| **Persistence** | State survives hibernation | Installed packages and files persist across sessions |

The sandbox provides a critical security advantage over local agents like OpenClaw: **the agent cannot access the user's personal files, credentials, or local network**. It operates in a completely isolated environment. However, this also means Manus cannot directly interact with the user's desktop, applications, or local services [1].

**Refinement Suggestion for Omega System:** The sandbox model is ideal for trading applications. Each trading agent should operate in its own isolated environment with only the specific API keys and permissions it needs. A compromised signal analysis agent should never have access to order execution credentials.

### 1.2 The Agent Loop

Manus operates in a continuous agent loop with six defined steps [1]:

```
1. Analyze Events → Understand user needs and current state
2. Select Tools   → Choose next tool call based on state and plan
3. Wait           → Tool action executes in sandbox
4. Iterate        → Repeat until task completion
5. Submit Results → Send deliverables to user
6. Enter Standby  → Wait for new tasks
```

This loop is structurally similar to OpenClaw's agentic loop but with a critical constraint: **Manus executes exactly one tool call per iteration** [1]. This is explicitly enforced in the system prompt:

> "MUST respond with exactly one tool call per response; parallel function calling is strictly forbidden." [1]

This serialization ensures deterministic execution order and simplifies debugging, but limits throughput for tasks that could benefit from parallel tool execution.

---

## 2. Tool System (29 Core Tools)

Manus exposes 29 tools organized into six categories [5]. This is a carefully curated set — far fewer than the unlimited tool expansion possible through OpenClaw's skill system, but each tool is deeply integrated and well-documented.

### 2.1 Communication Tools

| Tool | Function | Security Notes |
|------|----------|---------------|
| `message_notify_user` | Send information without requiring response | One-way communication, no user data exposure |
| `message_ask_user` | Ask question and wait for response | Blocks execution until user responds |

### 2.2 File System Tools

| Tool | Function | Security Notes |
|------|----------|---------------|
| `file_read` | Read file content | Sandboxed to `/home/ubuntu` |
| `file_write` | Create or overwrite files | Cannot write outside sandbox |
| `file_str_replace` | Find and replace in files | Targeted edits without full rewrites |
| `file_find_in_content` | Search file contents (grep) | Pattern matching across files |
| `file_find_by_name` | Find files by name pattern | Directory traversal within sandbox |

### 2.3 Shell Tools

| Tool | Function | Security Notes |
|------|----------|---------------|
| `shell_exec` | Execute shell commands | Full sudo access within sandbox |
| `shell_view` | View shell session output | Check command results |
| `shell_wait` | Wait for long-running processes | Timeout management |
| `shell_write_to_process` | Send stdin to running process | Interactive process control |
| `shell_kill_process` | Terminate running processes | Resource cleanup |

### 2.4 Browser Tools

| Tool | Function | Security Notes |
|------|----------|---------------|
| `browser_view` | View current page content | Returns elements + screenshot |
| `browser_navigate` | Navigate to URL | Full internet access |
| `browser_restart` | Restart browser session | State reset capability |
| `browser_click` | Click page elements | By index or coordinates |
| `browser_input` | Fill form fields | Text input to editable elements |
| `browser_move_mouse` | Move cursor | Hover effects and positioning |
| `browser_press_key` | Simulate key presses | Keyboard shortcuts |
| `browser_select_option` | Select dropdown options | Form interaction |
| `browser_scroll_up/down` | Scroll page content | Content navigation |
| `browser_console_exec` | Execute JavaScript | Full DOM access |
| `browser_console_view` | View console output | Debug information |

### 2.5 Search and Discovery Tools

| Tool | Function | Security Notes |
|------|----------|---------------|
| `info_search_web` | Web search | Returns results for further browsing |

### 2.6 Deployment Tools

| Tool | Function | Security Notes |
|------|----------|---------------|
| `deploy_expose_port` | Expose local port publicly | Temporary public access |
| `deploy_apply_deployment` | Deploy to production | Permanent hosting |
| `make_manus_page` | Create Manus Page from MDX | Content publishing |
| `idle` | Signal task completion | Enters standby mode |

**Refinement Suggestion:** The Omega System should define a similar tool taxonomy but specialized for trading:

| Category | Tools |
|----------|-------|
| **Market Data** | `fetch_price`, `fetch_orderbook`, `fetch_candles`, `fetch_funding_rate` |
| **Signal Processing** | `parse_tradingview_webhook`, `calculate_indicator`, `score_signal` |
| **Risk Management** | `calculate_position_size`, `run_monte_carlo`, `check_portfolio_risk` |
| **Order Execution** | `place_order`, `modify_order`, `cancel_order`, `check_order_status` |
| **Chart Annotation** | `add_entry_marker`, `add_exit_marker`, `draw_support_resistance` |
| **Communication** | `notify_user`, `send_trade_alert`, `update_dashboard` |

---

## 3. Module System (Behavioral Rules)

Manus organizes its behavioral rules into XML-tagged modules within the system prompt [1]. Each module defines rules for a specific domain of operation:

### 3.1 Module Taxonomy

| Module | Purpose | Key Rules |
|--------|---------|-----------|
| `<info_rules>` | Information gathering | Prefer search tools over browser; verify snippets by visiting source URLs |
| `<browser_rules>` | Web browsing | Must access user-provided URLs; actively explore links; save key info to files |
| `<shell_rules>` | Command execution | Use `-y`/`-f` flags; chain with `&&`; save code to files before execution |
| `<coding_rules>` | Code writing | Must save to files first; use Python for complex math; search for solutions |
| `<deploy_rules>` | Service deployment | Listen on `0.0.0.0`; test locally first; expose ports for user access |
| `<writing_rules>` | Document creation | Write in paragraphs (not lists); minimum several thousand words; cite sources |
| `<error_handling>` | Failure recovery | Verify tool names/args; try alternatives; report to user after multiple failures |

### 3.2 The "Save Before Execute" Pattern

One of Manus's most distinctive rules is the requirement to **save code to files before execution** [1]:

> "Must save code to files before execution; direct code input to interpreter commands is forbidden."

This pattern provides several benefits:
- **Auditability** — Every piece of executed code exists as a file that can be reviewed
- **Reproducibility** — Code can be re-run without reconstructing it from conversation history
- **Debugging** — Errors reference specific files and line numbers
- **Version control** — File modifications are tracked

**Refinement Suggestion:** The Omega System should enforce this pattern for all trading logic. Every signal processing rule, risk calculation, and order execution script must exist as a versioned file, never as inline code. This creates an audit trail for regulatory compliance.

### 3.3 The "Active Information Saving" Pattern

Manus requires agents to **actively save key information to text files** during browsing and research [1]:

> "MUST actively save key information obtained through browser to text files, especially information from images and tables, as subsequent operations may not have access to multimodal understanding."

This addresses a fundamental limitation of LLM-based agents: **context windows are finite and information can be lost between tool calls**. By explicitly persisting important findings to files, the agent creates a durable knowledge base that survives context compaction.

**Refinement Suggestion:** The Omega System should implement a similar pattern for market data. When the agent analyzes a chart or processes market data, key findings (support/resistance levels, trend direction, volume analysis) should be persisted to structured files, not just held in conversation context.

---

## 4. Authentication and User Management

Manus implements OAuth-based authentication with a session cookie model [6]:

1. User initiates login via Manus OAuth portal
2. OAuth callback at `/api/oauth/callback` drops a session cookie
3. Each API request builds context via `context.ts`, injecting `ctx.user`
4. Protected procedures require authentication; public procedures do not
5. Frontend reads auth state via `trpc.auth.me.useQuery()`

The system includes role-based access control with `admin` and `user` roles, enforced at the procedure level [6].

**Security Analysis:** The OAuth flow is well-structured but relies on cookie-based sessions, which are vulnerable to CSRF attacks if not properly configured with SameSite attributes. The system mitigates this by routing all RPC traffic through `/api/trpc`, making it easier to apply CSRF protections at the gateway level [6].

---

## 5. MCP Integration

Manus supports Model Context Protocol (MCP) integration for extending capabilities through external services [1]. MCP servers are configured at the task level and accessed via the `manus-mcp-cli` utility:

```bash
# List available tools
manus-mcp-cli tool list --server <server_name>

# Call a tool
manus-mcp-cli tool call <tool_name> --server <server_name> --input '<json_args>'
```

Pre-configured MCP servers include integrations for Meta Ads, Stripe, Zapier, n8n, Gmail, Google Calendar, PayPal, Canva, and others [1]. Each MCP server exposes a set of tools with defined schemas, and the agent discovers and calls these tools using a standard interface.

**Refinement Suggestion:** The Omega System should implement MCP servers for:
- **TradingView** — Receive webhook signals, manage alert configurations
- **Binance/Exchange** — Order management, account balance, position tracking
- **CoinGecko** — Market data, token information, trending analysis
- **Risk Engine** — Kelly Criterion calculations, Monte Carlo simulations

---

## 6. Web Development Workflow (Specialized Capability)

Manus includes a sophisticated web development workflow that demonstrates how specialized agent capabilities can be layered on top of the core agent loop [6]. This workflow includes:

| Component | Function |
|-----------|----------|
| `webdev_init_project` | Scaffold new projects with templates |
| `webdev_add_feature` | Extend projects with databases, auth, payments |
| `webdev_check_status` | Monitor dev server health |
| `webdev_save_checkpoint` | Version control with rollback capability |
| `webdev_request_secrets` | Manage environment variables and API keys |
| `webdev_execute_sql` | Direct database queries |
| `webdev_debug` | Invoke specialized debugging agent |

The web development workflow demonstrates a pattern of **specialized sub-agents** — the debugging agent, for example, is a completely separate agent with its own context and system prompt, invoked when the primary agent gets stuck [6].

**Refinement Suggestion:** The Omega System should implement similar specialized sub-agents:
- **Backtesting Agent** — Invoked when a strategy needs historical validation
- **Risk Audit Agent** — Independent agent that reviews trading decisions for risk compliance
- **Performance Analysis Agent** — Analyzes trade history and suggests strategy refinements

---

## 7. LLM Integration Patterns

Manus provides pre-configured LLM helpers that demonstrate best practices for model integration [6]:

### 7.1 Simple Chat Completion
```typescript
const response = await invokeLLM({
  messages: [
    { role: "system", content: "You are a helpful assistant." },
    { role: "user", content: "Hello, world!" },
  ],
});
```

### 7.2 Structured JSON Output
```typescript
const structured = await invokeLLM({
  messages: [...],
  response_format: {
    type: "json_schema",
    json_schema: {
      name: "trade_signal",
      strict: true,
      schema: {
        type: "object",
        properties: {
          symbol: { type: "string" },
          direction: { type: "string", enum: ["long", "short"] },
          confidence: { type: "number", minimum: 0, maximum: 100 },
          entry_price: { type: "number" },
          stop_loss: { type: "number" },
          take_profit: { type: "number" },
        },
        required: ["symbol", "direction", "confidence"],
        additionalProperties: false,
      },
    },
  },
});
```

### 7.3 Multimodal Input
Manus supports text, image, file, and audio inputs in a single message, enabling chart analysis, document processing, and voice transcription within the same agent loop [6].

**Refinement Suggestion:** The Omega System should use structured JSON output for all signal processing and order generation. This ensures that every agent output conforms to a strict schema, preventing malformed orders or ambiguous signals.

---

## 8. Comparison with OpenClaw

| Dimension | Manus | OpenClaw |
|-----------|-------|----------|
| **Execution Environment** | Cloud sandbox (Ubuntu VM) | Local machine (user's hardware) |
| **Security Model** | Sandbox isolation | Filesystem access controls |
| **Tool Count** | 29 curated tools | Unlimited via skills + MCP |
| **Tool Expansion** | MCP servers | Skills + MCP servers |
| **Memory** | File-based within sandbox | File-based in `~/.openclaw/` |
| **Proactive Behavior** | Task-triggered only | Heartbeat system (30-min intervals) |
| **Multi-Channel** | Web interface only | WhatsApp, Telegram, Slack, Discord, etc. |
| **Model Support** | Platform-managed | User-configured (Anthropic, OpenAI, Google, Ollama) |
| **Concurrency** | One tool per iteration | Serialized per session lane |
| **Deployment** | Built-in hosting | Self-hosted |
| **Cost** | Platform subscription | API costs only |

---

## 9. Key Architectural Lessons for the Omega System

1. **Sandbox isolation** is the gold standard for agent security. The Omega System should run each agent type in its own isolated container with only the permissions it needs.

2. **One tool per iteration** is conservative but safe. For trading, this means each decision step is atomic and auditable. Consider allowing parallel read-only operations (market data fetches) while serializing write operations (order placement).

3. **The module system** (XML-tagged behavioral rules) is an effective way to organize complex agent instructions. The Omega System should define modules for `<trading_rules>`, `<risk_rules>`, `<execution_rules>`, and `<communication_rules>`.

4. **Save-before-execute** creates an audit trail. Every trading decision should be logged as a file before execution.

5. **Specialized sub-agents** (like the debugging agent) demonstrate how to decompose complex tasks. The Omega System should use independent agents for backtesting, risk auditing, and performance analysis.

6. **Structured JSON output** ensures type-safe communication between agents and systems. All signal processing and order generation should use strict JSON schemas.

7. **MCP integration** provides a clean abstraction for external service access. Exchange APIs, market data feeds, and risk engines should all be exposed as MCP servers.

---

## References

[1]: https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools "Manus Agent Tools & Prompt — x1xhlol"
[2]: https://github.com/dontriskit/awesome-ai-system-prompts "Manus System Prompt — dontriskit"
[3]: https://github.com/jujumilk3/leaked-system-prompts "Manus Leaked System Prompts — jujumilk3"
[4]: https://github.com/asgeirtj/system_prompts_leaks "System Prompts Leaks — asgeirtj"
[5]: /tmp/omega-system-rough/prompts/manus/tools.json "Manus Tools Schema (29 tools)"
[6]: /tmp/omega-system-rough/prompts/manus/modules.txt "Manus Modules Configuration"
