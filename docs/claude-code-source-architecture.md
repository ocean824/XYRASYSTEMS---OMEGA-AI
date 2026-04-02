# Claude Code from Source — Complete Architecture Blueprint for DOMINION (Ø)

> **Source**: [claude-code-from-source.com](https://claude-code-from-source.com/) — 18 chapters, 7 parts, ~400 pages reverse-engineered from Claude Code's TypeScript source maps.
> **Role in ØMEGA AI**: This document defines the **main brain architecture** for DOMINION (Ø), the supreme orchestrator. Every pattern documented here is a direct implementation requirement for CodeSpring.
> **Reading Order**: Read this document AFTER `CODESPRING-MASTER-INSTRUCTIONS.md` and BEFORE writing any orchestrator code.

---

## Table of Contents

1. [The 10 Foundational Patterns](#the-10-foundational-patterns)
2. [The 6 Key Abstractions](#the-6-key-abstractions)
3. [The Agent Loop — query.ts Deep Dive](#the-agent-loop)
4. [4-Layer Context Compression](#4-layer-context-compression)
5. [The Tool System — Definition to Execution](#the-tool-system)
6. [Concurrent Tool Execution](#concurrent-tool-execution)
7. [Sub-Agent Spawning — 15-Step Lifecycle](#sub-agent-spawning)
8. [Fork Agents and Prompt Cache Sharing](#fork-agents-and-prompt-cache-sharing)
9. [Tasks, Coordination, and Swarms](#tasks-coordination-and-swarms)
10. [Memory — 4-Type Taxonomy with LLM Recall](#memory-system)
11. [Extensibility — Skills and Hooks](#extensibility-skills-and-hooks)
12. [MCP — The Universal Tool Protocol](#mcp-the-universal-tool-protocol)
13. [The 5 Architectural Bets](#the-5-architectural-bets)
14. [Transferable Patterns for ØMEGA AI](#transferable-patterns)
15. [Implementation Mapping to ØMEGA Agents](#implementation-mapping)

---

## The 10 Foundational Patterns

These are the 10 production-proven patterns extracted from Claude Code's source. Every one of them applies directly to DOMINION's implementation.

| # | Pattern | Description | ØMEGA Application |
|---|---------|-------------|-------------------|
| 1 | **AsyncGenerator as Agent Loop** | The `query()` function is an async generator that yields `Message` objects. Return type is a discriminated union `Terminal` encoding exactly why the loop stopped (10 terminal states). | DOMINION's main orchestration loop must be an async generator, not a callback pipeline. |
| 2 | **Speculative Tool Execution** | Start read-only tools while the model is still streaming its response, before the response completes. | ARCANE, MODULUS, and PHANTOM can begin file reads and searches before DOMINION finishes routing. |
| 3 | **Concurrent-Safe Batching** | Partition tool calls by per-input safety classification. Run reads in parallel, serialize writes. | All agents partition their tool calls through the same algorithm. |
| 4 | **Fork Agents for Cache Sharing** | Parallel children share byte-identical prompt prefixes, saving ~95% input tokens. | DOMINION spawns fork agents for parallel research, analysis, and verification tasks. |
| 5 | **4-Layer Context Compression** | Snip → Microcompact → Collapse → Auto-compact. Each progressively heavier. Circuit breaker after 3 auto-compact failures. | DOMINION implements all 4 layers to manage 200K+ token sessions. |
| 6 | **File-Based Memory with LLM Recall** | Sonnet side-query selects relevant memories from YAML frontmatter, not keyword matching. | ARCHIVE implements this exact pattern for persistent knowledge. |
| 7 | **Two-Phase Skill Loading** | Frontmatter loads at startup (cheap). Full content loads only on invocation (expensive). | All 25 agents load their skill manifests in Phase 1, execute in Phase 2. |
| 8 | **Sticky Latches for Cache Stability** | Once a beta header is sent, never unset mid-session. Prompt cache stability is an architectural concern. | DOMINION structures prompts with stable content first, volatile content last. |
| 9 | **Slot Reservation** | 8K default output cap, escalate to 64K only when the cap is hit. Saves context in 99% of requests. | All agents use slot reservation to prevent context waste. |
| 10 | **Hook Config Snapshot** | Freeze hook configuration at startup to prevent runtime injection attacks. | WARDEN enforces snapshot security for all hook configurations. |

---

## The 6 Key Abstractions

Claude Code's entire architecture rests on 6 abstractions. DOMINION inherits all of them.

### Abstraction 1: The Query Loop (`query.ts`, ~1,730 lines)

The single most important file in the system. It is an async generator that streams model responses, collects tool calls, executes them, appends results, and loops. Every interaction — REPL, SDK, sub-agent, headless runner — passes through this one function.

The return type is a discriminated union `Terminal` that encodes exactly why the loop stopped. There are **10 terminal states** (normal completion, user abort, token budget exhaustion, stop hook intervention, max turns, unrecoverable error, etc.) and **7 continuation states** (tool calls pending, model wants more turns, etc.). The type system enforces exhaustive handling — you cannot add a new stop reason without updating every caller.

**DOMINION Implementation**: The master orchestrator loop must be an async generator with typed terminal states. Every agent's loop follows this same pattern.

### Abstraction 2: The Tool System (`Tool.ts`, `tools.ts`, `services/tools/`)

Tools are not simple function wrappers. Each tool carries ~45 interface members across 5 critical categories:

| Member | Purpose |
|--------|---------|
| `call()` | Execute the tool |
| `inputSchema` | Zod schema for validation (converts to JSON Schema for API + runtime validation) |
| `isConcurrencySafe(input)` | Per-input concurrency classification (e.g., `ls` safe, `rm` not) |
| `checkPermissions()` | Permission evaluation |
| `validateInput()` | Semantic validation beyond schema |

The `buildTool()` factory enforces **fail-closed defaults**: new tools are serial, non-read-only, and non-destructive by default. The tool registry (`assembleToolPool()`) merges built-in and MCP tools, sorts alphabetically for prompt cache stability, and applies deny-rule filtering.

**14-Step Execution Pipeline**: Validation (4 steps) → Preparation (2 steps) → Permission (3 steps) → Execution and Cleanup (5 steps).

### Abstraction 3: Tasks

Background work units with a state machine: `pending → running → completed | failed | killed`. Seven task types exist:

| Type | Prefix | Purpose |
|------|--------|---------|
| `local_bash` | b | Background shell commands |
| `local_agent` | a | Background sub-agents |
| `remote_agent` | r | Remote sessions |
| `in_process_teammate` | t | Swarm teammates |
| `local_workflow` | w | Workflow script executions |
| `monitor_mcp` | m | MCP server monitors |
| `dream` | d | Speculative background thinking |

Storage is a flat map (`Record<string, TaskState>`), not a tree. Parent-child relationships are implicit via `toolUseId`. Garbage collection uses an `evictAfter` deadline.

### Abstraction 4: State (Two-Tier Architecture)

The system maintains two distinct state layers. The first is a mutable singleton (`STATE`) with approximately 80 fields covering working directory, model configuration, cost tracking, and session ID. The second is a minimal reactive store (34 lines, Zustand-shaped) that drives the UI. This separation ensures that infrastructure state changes do not trigger unnecessary UI re-renders.

**Sticky latches** are a key pattern: once a configuration value is set (e.g., a beta header), it is never unset mid-session. This preserves prompt cache stability.

### Abstraction 5: Memory (3 Tiers)

Memory operates at three levels: project-level (`CLAUDE.md` files in the repository), user-level (`~/.claude/MEMORY.md`), and team-level (shared via symlinks). An LLM side-query selects relevant memories at session start based on conversation context, not keyword matching.

### Abstraction 6: Hooks (27 Events, 4 Types)

The hook system fires at 27 distinct lifecycle events across 4 execution types: shell commands, single-shot LLM calls, multi-turn agent loops, and HTTP webhooks. Hooks can block actions, modify inputs, inject content, or silently observe. The configuration is frozen at startup (snapshot security model) to prevent runtime injection.

---

## The Agent Loop

The `query()` function in `query.ts` is the beating heart of the system. It is approximately 1,730 lines of TypeScript implementing a single async generator.

### Why an Async Generator

Most agent frameworks use a callback pipeline: define tools, register handlers, let the framework orchestrate. The developer writes callbacks. Claude Code inverts this. The `query()` function is an async generator — the developer owns the loop. The model streams a response, the generator yields tool calls, the caller executes them, appends results, and the generator loops. There is one function, one data flow, one place where every interaction passes through.

The bet was that a single generator function, even one that grew to 1,700 lines, would be more comprehensible than a distributed callback graph. The type system enforces exhaustive handling of all terminal and continuation states.

### The State Object

Each invocation of `query()` receives a `ToolUseContext` with approximately 40 fields:

- **Configuration**: tool set, model name, MCP connections, debug flags
- **Execution state**: abortController, readFileState (LRU file cache), messages
- **UI callbacks**: setToolJSX, addNotification, requestPrompt
- **Agent context**: agentId, renderedSystemPrompt (frozen for fork sub-agents)

### Immutable Transitions in a Mutable Loop

The loop body follows a strict pattern: read state, call model, collect tool uses, execute tools, append results, check termination conditions, loop. State transitions are immutable — each iteration produces a new state rather than mutating the previous one. This makes debugging and replay possible.

---

## 4-Layer Context Compression

As conversations grow, context must be compressed to stay within the model's window. Claude Code implements 4 progressively heavier compression layers, plus a per-tool result budget (Layer 0).

| Layer | Name | Description | Cost |
|-------|------|-------------|------|
| 0 | **Tool Result Budget** | Per-tool output size limits. Prevents a single tool from consuming the entire context. | Negligible |
| 1 | **Snip Compact** | Removes old tool results while preserving conversation structure. | Low |
| 2 | **Microcompact** | Aggressive inline compression of message content. | Medium |
| 3 | **Context Collapse** | Collapses entire conversation sections into summaries. | High |
| 4 | **Auto-Compact** | Forks an entire Claude conversation to summarize history. Circuit breaker after 3 consecutive failures. | Very High |

### Auto-Compact Thresholds

The thresholds are derived from the model's context window:

```
effectiveContextWindow = contextWindow - min(modelMaxOutput, 20000)

Thresholds (relative to effectiveContextWindow):
  Auto-compact fires:      effectiveWindow - 13,000
  Blocking limit (hard):   effectiveWindow - 3,000
```

The 13,000-token buffer means auto-compact fires well before the hard limit. The gap between the auto-compact threshold and the blocking limit is where reactive compact operates — if proactive auto-compact fails, reactive compact catches the 413 error and compacts on demand.

### Error Recovery — Escalation Ladder

When errors occur during the loop, the system escalates through three levels: retry with backoff, model fallback (switch to a different provider), and the death spiral guard (prevents infinite retry loops that were observed in production burning 250K API calls per day).

---

## The Tool System

### The 14-Step Execution Pipeline

Every tool call passes through exactly 14 steps:

**Steps 1-4 (Validation)**: Input parsing via Zod schema, JSON Schema check, semantic validation (does the input make sense?), abort signal check.

**Steps 5-6 (Preparation)**: Context setup (working directory, environment), pre-hooks fire (PreToolUse lifecycle event).

**Steps 7-9 (Permission)**: Permission mode check (7 modes from `bypassPermissions` to `bubble`), rule matching against allow/deny lists, user prompt if required.

**Steps 10-14 (Execution and Cleanup)**: Tool `call()` execution, result budgeting (truncate oversized output), post-hooks fire (PostToolUse), message append to conversation, error handling and recovery.

### Permission System — 7 Modes

| Mode | Behavior |
|------|----------|
| `bypassPermissions` | Everything allowed. Internal/testing only. |
| `dontAsk` | All allowed, logged. No prompts. |
| `auto` | Transcript classifier (LLM) decides allow/deny. |
| `acceptEdits` | File edits auto-approved; other mutations prompt. |
| `default` | Standard interactive. User approves each action. |
| `plan` | Read-only. All mutations blocked. |
| `bubble` | Escalate to parent agent (sub-agent mode). |

### Individual Tool Highlights

**BashTool** is the most complex tool. It performs command classification (safe vs. unsafe subcommands), timeout handling, and output truncation. The `isConcurrencySafe()` method parses the command string, checks if every subcommand is read-only (`ls`, `grep`, `cat`, `git status`), and returns `true` only if the entire compound command is a pure read.

**FileEditTool** implements staleness detection — it checks whether the file has changed since the last read before applying edits, preventing silent overwrites of concurrent modifications.

**AgentTool** spawns sub-agents with context modifiers, enabling recursive agent hierarchies.

---

## Concurrent Tool Execution

### Core Insight

Concurrency safety is **per-call, not per-tool-type**. `Bash("ls -la")` is safe to parallelize. `Bash("rm -rf build/")` is not. The same tool, different inputs, different concurrency classification. The system must inspect the input before deciding.

### The Partition Algorithm (`partitionToolCalls` in `toolOrchestration.ts`)

The algorithm walks tool calls left to right:

1. Look up the tool definition by name
2. Parse the input with the tool's Zod schema via `safeParse()`
3. Call `isConcurrencySafe(parsedInput)` — per-input classification
4. If safe AND previous batch is safe, append to batch; otherwise start new batch
5. **Fail-closed**: parse failure or exception → serial

The result is a sequence of batches alternating between concurrent groups and individual serial entries.

### Streaming Tool Executor (`StreamingToolExecutor`)

This is the speculative execution engine. It starts running tools **while the model is still streaming its response**, harvesting results before the response is even complete.

The executor has 4 key methods: `addTool()` queues tools during the stream, `processQueue()` performs the admission check (is the tool safe? is the input complete?), `executeTool()` runs the core execution loop, and `discard()` provides an escape hatch for invalidated speculative results.

Order preservation is guaranteed — results are returned in the order tools were requested, not the order they completed.

### Batch Execution

Concurrent batches use `boundedAll()` with `MAX_CONCURRENCY=10` (configurable via `CLAUDE_CODE_MAX_TOOL_USE_CONCURRENCY`). Context modifiers are collected during concurrent execution and applied in tool-order after the batch completes, preserving deterministic context evolution.

---

## Sub-Agent Spawning

### The 15-Step `runAgent` Lifecycle

When DOMINION spawns a sub-agent, it follows exactly 15 steps:

| Step | Action | Purpose |
|------|--------|---------|
| 1 | Model Resolution | Override to sonnet/opus/haiku based on agent type |
| 2 | Agent ID Creation | Unique identifier for tracking |
| 3 | Context Preparation | Build initial context from parent |
| 4 | CLAUDE.md Stripping | Remove parent's memory to save tokens |
| 5 | Permission Isolation | Set to `bubble` mode (escalate to parent) |
| 6 | Tool Resolution | Subset of parent's tools based on agent role |
| 7 | System Prompt | Agent-type-specific prompt injection |
| 8 | Abort Controller Isolation | Child gets own abort controller |
| 9 | Hook Registration | Register agent-specific hooks |
| 10 | Skill Preloading | Load relevant skills for agent type |
| 11 | MCP Initialization | Connect to required MCP servers |
| 12 | Context Creation | `createSubagentContext()` builds the full context |
| 13 | Cache-Safe Params | Callback for prompt cache optimization |
| 14 | The Query Loop | New `query()` generator with own message history |
| 15 | Cleanup | Release resources, close connections |

### 8 Built-In Agent Types

| Type | Model | Access | Purpose |
|------|-------|--------|---------|
| General-Purpose | Same as parent | Full tool access | Default sub-agent |
| Explore | Haiku | Read-only | Cheap codebase search |
| Plan | Same as parent | Read-only | Generate plans without executing |
| Verification | Same as parent | Full | Adversarial testing, runs tests |
| Claude Code Guide | Haiku | Read-only | Answers questions about Claude Code |
| Statusline Setup | Haiku | Limited | Configures terminal status line |
| Worker (Coordinator) | Same as parent | Full | Receives tasks from coordinator |
| Fork Agent | Same as parent | Full | Parallel children sharing prompt cache |

### Agent Design Dimensions

Every sub-agent is defined along 5 dimensions: what it can see (tool access, file access), what it can do (read-only vs read-write), how it interacts with the user (silent vs interactive), how it relates to the parent (sync vs async, result passing), and how expensive it is (model choice, token budget).

Sub-agents can spawn grandchildren recursively. Filesystem isolation uses git worktrees for parallel agents. Named agents communicate via `SendMessage({to: name})`.

---

## Fork Agents and Prompt Cache Sharing

Fork agents are the key cost optimization for parallel work. When DOMINION needs to explore multiple approaches simultaneously, it forks rather than spawning fresh agents.

The trick is **byte-identical prompt prefixes**. The parent's conversation history up to the fork point is shared across all children. Because the prompt cache keys on exact byte sequences, all forked children hit the same cache entry, saving approximately 95% of input tokens.

This is why prompt cache stability (Pattern #8, sticky latches) is an architectural concern, not just an optimization. If any part of the shared prefix changes between children, the cache miss costs real money.

---

## Tasks, Coordination, and Swarms

### Coordinator Mode

A 370-line system prompt defines the coordinator role. The coordinator is a manager that delegates all writes to worker agents while retaining read-only access for oversight. Coordinator mode and fork mode are mutually exclusive.

### Swarm System

The swarm system enables multi-agent collaboration through several mechanisms:

- **Team context** for grouping agents into logical teams
- **Agent name registry** for addressing specific agents by name
- **In-process teammates** that share memory space
- **Mailbox system** for message queuing between agents
- **Permission forwarding** between agents
- **SendMessage tool** for inter-agent communication
- **Auto-resume pattern**: dead agents can be revived by incoming messages
- **TaskStop kill switch** for emergency termination

### Communication Patterns

Foreground communication uses a generator chain (synchronous, yields messages). Background communication uses three channels: an output file on disk, a pending messages queue, and progress tracking.

---

## Memory System

### 4-Type Memory Taxonomy

Claude Code's memory system uses exactly 4 types, each with a clear purpose. The system explicitly **excludes derivable information** (code patterns, git history, architecture) — forcing the model to read the codebase rather than relying on stale memories.

| Type | Purpose | Example |
|------|---------|---------|
| `user` | Person info (role, goals, expertise) | "Senior Go engineer, new to React" |
| `feedback` | Corrections and confirmations | "Integration tests must hit real DB, not mocks" |
| `project` | Ongoing work context (who, what, when) | "Sprint 4 deadline: 2026-03-15" |
| `reference` | Bookmarks to external systems | "Linear project URL, Grafana dashboard" |

### Memory File Format

Each memory is a Markdown file with YAML frontmatter containing `name`, `description`, and `type`. The `description` field is the "search index" — it is consumed by the LLM during recall, not by a search engine. Only the first 30 lines (frontmatter) are scanned during the recall phase.

### Recall Pipeline

The recall pipeline operates in 3 steps:

1. `scanMemoryFiles()` reads frontmatter from all memory files
2. A Sonnet side-query selects relevant memories based on conversation context
3. Full content is loaded only for selected memories

This design means a project with 200 memories pays the token cost of 200 short descriptions during recall, not 200 full documents.

### 3 Memory Tiers

| Tier | Location | Scope |
|------|----------|-------|
| Project | `CLAUDE.md` files in repo | Shared with all agents working on the project |
| User | `~/.claude/MEMORY.md` | Personal, available everywhere |
| Team | Shared via symlinks | Collaborative, with minimal sync/conflict resolution |

### Advanced Memory Features

**KAIROS Mode**: Append-only daily logs for long-running sessions. **Dream Consolidation**: Background memory consolidation with file locking during idle time. **Background Extraction**: Async memory extraction that runs during idle periods. **Staleness**: Memories have expiration tracking; the system handles stale memory detection and cleanup.

---

## Extensibility — Skills and Hooks

### Skills: Two-Phase Loading

**Phase 1 (Startup)**: Read YAML frontmatter only from `SKILL.md` files across 7 sources. Extract `name`, `description`, and `whenToUse`. Build system prompt menu so the model knows skills exist. A project with 50 skills pays the token cost of 50 short descriptions, not 50 full documents.

**Phase 2 (Invocation)**: Full content loaded on demand. Variable substitution (`$ARGUMENTS`, `${CLAUDE_SKILL_DIR}`, `${CLAUDE_SESSION_ID}`), inline shell execution (backtick-prefixed with `!`). Content blocks injected into conversation.

### 7 Skill Sources (by priority)

| Priority | Source | Location | Trust |
|----------|--------|----------|-------|
| 1 | Managed (Policy) | Enterprise-controlled path | Pre-approved |
| 2 | User | `~/.claude/skills/` | Personal |
| 3 | Project | `.claude/skills/` (walked up to home) | Version-controlled |
| 4 | Additional Dirs | Via `--add-dir` flag | Explicit |
| 5 | Legacy Commands | `.claude/commands/` | Backwards-compatible |
| 6 | Bundled | Compiled into binary | Feature-gated |
| 7 | MCP | MCP server prompts | Untrusted (NO shell execution) |

### MCP Security Boundary

MCP skills **never execute inline shell commands**. An MCP prompt containing `` !`rm -rf /` `` would execute with full permissions if allowed. MCP skills are content-only.

### Dynamic Discovery

Skills are not only loaded at startup. When the model touches files, `discoverSkillDirsForPaths` walks up from each path looking for `.claude/skills/` directories. Skills with `paths` frontmatter activate only when touched paths match their glob patterns — context-sensitive capability expansion.

### Hooks: 4 User-Configurable Types

| Type | Mechanism | Use Case |
|------|-----------|----------|
| **Command** | Shell process, stdin JSON, exit code protocol | Linting, validation, branch protection |
| **Prompt** | Single LLM call, returns ok/not-ok | AI-powered validation |
| **Agent** | Multi-turn agentic loop (max 50 turns) | Complex verification workflows |
| **Webhook** | HTTP POST | External service integration |

### Exit Code Semantics

Exit code `0` means allow (pass). Exit code `2` means block (hard stop). Any other exit code produces a non-blocking warning.

### 5 Most Important Lifecycle Events

`PreToolUse`, `PostToolUse`, `PreModelResponse`, `PostModelResponse`, `SessionStart`

### 6 Hook Sources

User, project, managed, skill-declared, internal, dynamic.

### Snapshot Security Model

Hook configuration is **frozen at startup** to prevent runtime injection attacks. A malicious tool cannot modify hook behavior mid-session.

### Stop Hooks

Stop hooks can force the model to continue working past its natural stopping point — useful for ensuring thorough verification before reporting completion.

---

## MCP — The Universal Tool Protocol

### 8 Transport Types

| Transport | Use Case |
|-----------|----------|
| `stdio` | Default. Local subprocess, no network, no auth, just pipes. |
| `http` | Streamable HTTP. Current spec recommendation for remote services. |
| `sse` | Legacy Server-Sent Events. Widely deployed. |
| `sdk` | Internal SDK transport. |
| `ws-ide` | WebSocket for IDE integration (Bun/Node runtime split). |
| `claudeai-proxy` | Routes through Claude.ai infrastructure for vendor-side OAuth. |
| `in-process` | 63 lines. Same-process MCP server/client via `queueMicrotask()`. |
| `dynamic` | Runtime injection via SDK. |

### 7 Configuration Scopes

| Scope | Source | Trust Level |
|-------|--------|-------------|
| `local` | `.mcp.json` in working directory | Requires user approval |
| `user` | `~/.claude.json` mcpServers field | User-managed |
| `project` | Project-level config | Shared project settings |
| `enterprise` | Managed enterprise config | Pre-approved by org |
| `managed` | Plugin-provided servers | Auto-discovered |
| `claudeai` | Claude.ai web interface | Pre-authorized via web |
| `dynamic` | Runtime injection (SDK) | Programmatically added |

Deduplication is **content-based, not name-based**. Two servers with different names but the same command or URL are recognized as the same server.

### Tool Wrapping (4 Stages)

When an MCP server connects, its tools are wrapped into Claude Code's internal `Tool` interface through 4 stages:

1. **Name normalization**: `mcp__{serverName}__{toolName}` (must match `^[a-zA-Z0-9_-]{1,64}$`)
2. **Description truncation**: Capped at 2,048 characters (OpenAPI-generated servers dump 15-60KB otherwise)
3. **Schema passthrough**: No transformation at wrapping time; errors surface at call time
4. **Annotation mapping**: `readOnlyHint` → concurrent execution, `destructiveHint` → extra permission scrutiny

After wrapping, the model **cannot distinguish between a built-in tool and an MCP tool**.

### OAuth for MCP Servers

Full OAuth 2.0 + PKCE flow with RFC-based discovery. Cross-App Access (XAA) enables federated token exchange — one IdP login unlocks multiple MCP servers. Error body normalization handles OAuth servers that violate the spec (e.g., Slack returning HTTP 200 for errors).

### Connection Management

5 connection states: `connected`, `failed`, `needs-auth` (15-minute TTL cache), `pending`, `disabled`.

### Timeout Architecture

| Layer | Duration | Protects Against |
|-------|----------|-----------------|
| Connection | 30s | Unreachable or slow-starting servers |
| Per-request | 60s (fresh per request) | Stale timeout signal bug |
| Tool call | ~27.8 hours | Legitimately long operations |
| Auth | 30s per OAuth request | Unreachable OAuth servers |

### Batched Connections

Local servers connect in batches of 3 (file descriptor limits). Remote servers connect in batches of 20.

---

## The 5 Architectural Bets

These are the 5 fundamental design decisions that define Claude Code's architecture. DOMINION inherits all 5.

### Bet 1: The Generator Loop Over Callbacks

A single `query()` function (1,700 lines) as an async generator. 10 terminal states, 7 continuation states. The type system enforces exhaustive handling. The bet was that one comprehensible function is better than a distributed callback graph.

### Bet 2: File-Based Memory Over Databases

Plain Markdown files instead of SQLite or vector databases. The 4-type taxonomy (user, feedback, project, reference) and the derivability test ("can this be re-derived from the current project state?") are reusable design heuristics. Works at 200 memories. Open question at 2,000+.

### Bet 3: Self-Describing Tools Over Central Orchestrators

Tools carry their own permissions, concurrency declarations, and rendering logic. There is no central router that decides what each tool can do. The tool itself knows.

### Bet 4: Fork Agents for Cache Sharing

Byte-identical prompt prefixes save ~95% input tokens on parallel children. This makes parallel exploration economically viable.

### Bet 5: Hooks Over Plugins

27 lifecycle events, 4 execution types. Configuration snapshot frozen at startup to prevent injection. Hooks are control flow; skills are content. The separation is deliberate.

---

## Transferable Patterns

These patterns transfer to **any** agent system, not just Claude Code:

1. **Generator loop with discriminated union return type** — eliminates "why did the agent stop?" debugging
2. **File-based memory with LLM recall + 4-type taxonomy + derivability test** — simple storage, intelligent retrieval
3. **Asymmetric read/write channels for remote execution** — reads are high-frequency streams, writes are low-frequency RPCs
4. **Bitmap pre-filters for search** — 26-bit letter bitmap, 4 bytes per entry, one integer comparison per candidate
5. **Prompt cache stability as architectural concern** — stable content first, volatile content last

---

## Implementation Mapping to ØMEGA Agents

This table maps every Claude Code architecture component to the specific ØMEGA AI agent responsible for implementing it.

| Architecture Component | ØMEGA Agent | Implementation Notes |
|----------------------|-------------|---------------------|
| `query()` async generator loop | **DOMINION (Ø)** | Main orchestration loop. All 25 agents inherit this pattern. |
| 4-layer context compression | **DOMINION (Ø)** | Manages context for all agent conversations. |
| Tool partition algorithm | **DOMINION (Ø)**, **ARCANE** | DOMINION partitions at orchestrator level; ARCANE partitions coding tools. |
| Streaming tool executor | **ARCANE**, **MODULUS** | Speculative execution for read-heavy coding and analysis tasks. |
| Sub-agent spawning (15 steps) | **DOMINION (Ø)** | Spawns all 24 sub-agents using this lifecycle. |
| Fork agents / cache sharing | **DOMINION (Ø)**, **PHANTOM** | Parallel research and intelligence gathering. |
| Coordinator mode | **DOMINION (Ø)** | Manager-worker hierarchy for complex multi-agent campaigns. |
| Swarm system | **PHANTOM**, **MODULUS** | Parallel intelligence and analysis swarms. |
| 4-type memory taxonomy | **ARCHIVE** | Persistent knowledge management across all sessions. |
| Memory recall pipeline | **ARCHIVE** | LLM-powered memory selection at session start. |
| KAIROS mode | **ARCHIVE** | Long-running session logging. |
| Dream consolidation | **ARCHIVE** | Background memory optimization. |
| Two-phase skill loading | **ALL AGENTS** | Every agent loads skills via frontmatter-first pattern. |
| Hook system (27 events) | **WARDEN**, **SENTINEL** | Security hooks, permission enforcement, audit logging. |
| Snapshot security model | **WARDEN** | Freeze hook config at startup. |
| Stop hooks | **SIGMA** | Force agents to continue verification before reporting completion. |
| MCP 8 transports | **DOMINION (Ø)** | Universal tool protocol for all external integrations. |
| MCP tool wrapping | **DOMINION (Ø)** | Normalize all MCP tools into internal Tool interface. |
| MCP OAuth + XAA | **WARDEN** | Secure authentication for remote MCP servers. |
| Permission system (7 modes) | **WARDEN**, **DOMINION (Ø)** | Tiered permission enforcement across all agents. |
| BashTool command classification | **ARCANE** | Safe/unsafe command detection for concurrent execution. |
| FileEditTool staleness detection | **ARCANE** | Prevent silent overwrites of concurrent modifications. |
| Result budgeting | **DOMINION (Ø)** | Per-tool and per-conversation token budget management. |
| Slot reservation (8K/64K) | **ALL AGENTS** | Context-efficient output management. |
| Sticky latches | **DOMINION (Ø)** | Prompt cache stability across sessions. |

---

> **This document is the architectural blueprint for DOMINION's brain. Every pattern, every abstraction, every implementation detail documented here must be reflected in the CodeSpring build. The source is [claude-code-from-source.com](https://claude-code-from-source.com/) — the most comprehensive reverse-engineering of a production AI agent ever published.**
