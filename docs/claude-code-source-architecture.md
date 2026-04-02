# The Main Brain Architecture: DOMINION (Ø)

> **Source**: [claude-code-from-source.com](https://claude-code-from-source.com/) — 18 chapters, 7 parts, ~400 pages reverse-engineered from Claude Code's TypeScript source maps.
> **Role in ØMEGA AI**: This document defines the **main brain architecture** for DOMINION (Ø), the supreme orchestrator. Every pattern documented here is a direct implementation requirement for CodeSpring.
> **Reading Order**: Read this document AFTER `CODESPRING-MASTER-INSTRUCTIONS.md` and BEFORE writing any orchestrator code.

---

## Table of Contents

1. [The 10 Foundational Patterns](#the-10-foundational-patterns)
2. [The 6 Key Abstractions](#the-6-key-abstractions)
3. [Starting Fast: The Bootstrap Pipeline](#starting-fast-the-bootstrap-pipeline)
4. [State: The Two-Tier Architecture](#state-the-two-tier-architecture)
5. [The API Layer: Talking to Models](#the-api-layer-talking-to-models)
6. [The Agent Loop — query.ts Deep Dive](#the-agent-loop)
7. [4-Layer Context Compression](#4-layer-context-compression)
8. [The Tool System — Definition to Execution](#the-tool-system)
9. [Concurrent Tool Execution](#concurrent-tool-execution)
10. [Sub-Agent Spawning — 15-Step Lifecycle](#sub-agent-spawning)
11. [Fork Agents and Prompt Cache Sharing](#fork-agents-and-prompt-cache-sharing)
12. [Tasks, Coordination, and Swarms](#tasks-coordination-and-swarms)
13. [Memory — 4-Type Taxonomy with LLM Recall](#memory-system)
14. [Extensibility — Skills and Hooks](#extensibility-skills-and-hooks)
15. [Input and Interaction Pipeline](#input-and-interaction-pipeline)
16. [The Terminal UI: Custom Rendering Engine](#the-terminal-ui-custom-rendering-engine)
17. [Remote Control and Cloud Execution](#remote-control-and-cloud-execution)
18. [Performance Engineering](#performance-engineering)
19. [MCP — The Universal Tool Protocol](#mcp-the-universal-tool-protocol)
20. [The 5 Architectural Bets](#the-5-architectural-bets)
21. [Implementation Mapping to ØMEGA Agents](#implementation-mapping)

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

1. **The Query Loop** (`query.ts`): The heartbeat. An async generator that streams events to the UI and returns a typed terminal state.
2. **The Tool System**: A 14-step pipeline that transforms raw model output into executed commands, with strict safety partitions.
3. **The State System**: Two-tier architecture separating mutable process infrastructure (`STATE` singleton) from reactive UI state (`AppState`).
4. **The Hooks Pipeline**: Extensibility mechanism allowing external code to intercept, modify, or block actions at 27 distinct lifecycle points.
5. **The Memory System**: File-backed semantic storage using a lightweight side-model (Sonnet) for retrieval rather than vector similarity.
6. **The Agent Protocol**: The abstraction that allows DOMINION to spawn 24 specialized sub-agents, orchestrate them via Coordinator Mode, or fork them for parallel execution.

---

## Starting Fast: The Bootstrap Pipeline

The bootstrap pipeline ensures the system boots in under **300 milliseconds** — the threshold where a CLI feels instant. DOMINION implements this exact 5-phase startup sequence:

1. **Phase 0: Fast-Path Dispatch (`cli.tsx`)**
   - Intercepts narrow-intent commands (`--version`, `--help`, `mcp list`)
   - Dynamically imports only the required handler and exits before the rest of the system loads
2. **Phase 1: Module-Level I/O (`main.tsx`)**
   - Fires slow I/O operations (keychain reads, MDM subprocesses) as side effects *during* import evaluation
   - Overlaps I/O wait time with the ~135ms required to evaluate the remaining static imports
3. **Phase 2: Parse and Trust (`init.ts`)**
   - Resolves configuration and establishes the **Trust Boundary** via Commander's `preAction` hook
   - Safe operations run before trust; dangerous operations (reading `LD_PRELOAD`, `PATH`) run only after explicit user consent
4. **Phase 3: Setup (`setup.ts`)**
   - Promise parallelism: socket binding, hook snapshotting, command loading, and agent definition loading run concurrently
5. **Phase 4: Launch (`replLauncher.ts`)**
   - 7 launch paths converge here to pick the right entry point and start the UI
   - Post-render deferred prefetches (git status, model capabilities) run only after the prompt is visible

**Performance Strategy**: Dynamic imports defer module evaluation. Heavy modules (like OpenTelemetry) load only when explicitly required.

---

## State: The Two-Tier Architecture

DOMINION manages state using a two-tier architecture to prevent circular dependencies and unnecessary React re-renders.

### Tier 1: Bootstrap State (The Process Singleton)
A single mutable object (`STATE`) created once at process start. It contains ~80 fields accessed via ~100 explicit getter/setter functions.
- **Categories**: Identity/paths, Cost/metrics, Telemetry, Model config, Session flags, Cache optimization.
- **Path Normalization**: Every path setter enforces NFC normalization to prevent Unicode mismatches.
- **Signal Pattern**: Uses a minimal pub/sub primitive (`createSignal`) without importing listeners, avoiding DAG cycles.

**The 5 Sticky Latches**:
Prompt cache preservation relies on five boolean latches that, once set to `true`, never return to `false` mid-session. This prevents beta headers from toggling and busting the cache:
1. `afkModeHeaderLatched`: Prevents auto mode toggling from busting cache.
2. `fastModeHeaderLatched`: Prevents fast mode cooldown from busting cache.
3. `cacheEditingHeaderLatched`: Prevents remote feature flag changes from busting cache.
4. `thinkingClearLatched`: Triggered on confirmed cache miss (>1h idle).
5. `pendingPostCompaction`: Consume-once flag for telemetry compaction tracking.

### Tier 2: AppState (The Reactive Store)
A minimal, 34-line Zustand-shaped reactive store for UI state (~20 fields).
- **Side-Effect Bridge**: `onChangeAppState` bridges the two tiers, allowing infrastructure modules to trigger UI updates without importing React.

---

## The API Layer: Talking to Models

DOMINION routes all model communication through a single, transparent factory interface.

1. **Multi-Provider Client Factory**
   - `getAnthropicClient()` dispatches to Direct API, AWS Bedrock, Google Vertex AI, or Azure Foundry based on environment variables.
   - All providers are cast to `Anthropic` via type erasure — the query loop never branches on the provider.
   - Unused providers are never loaded (dynamic imports).
2. **System Prompt Construction & The 2^N Problem**
   - **Dynamic Boundary Marker**: Divides the prompt into static (globally cached) and dynamic (per-session) sections.
   - **Rule**: Every conditional before the boundary doubles the global cache variants. Runtime checks *must* go after the boundary.
   - Cache-breaking sections use the `DANGEROUS_uncachedSystemPromptSection` naming convention to force justification.
3. **Streaming & The Idle Watchdog**
   - Uses raw SSE over SDK abstractions to avoid O(n²) partial JSON parsing on large tool inputs.
   - **Idle Watchdog**: A 90s timeout that resets on each chunk. If no chunks arrive, it aborts and falls back to a non-streaming retry.
4. **Prompt Cache Tiers**
   - Ephemeral (~5min), 1-hour TTL (eligible users), and Global scope (system prompt).
   - Global scope is disabled when MCP tools are present, as user-specific definitions fragment the cache.

## The Agent Loop

The `query()` function in `query.ts` is the beating heart of the system. It is approximately 1,730 lines of TypeScript implementing a single async generator.

### Why an Async Generator

Most agent frameworks use a callback pipeline: define tools, register handlers, let the framework orchestrate. Claude Code inverts this. The `query()` function is an async generator — the developer owns the loop. The model streams a response, the generator yields tool calls, the caller executes them, appends results, and the generator loops. 

The return type is a discriminated union `Terminal` that encodes exactly why the loop stopped. There are **10 terminal states** (normal completion, user abort, token budget exhaustion, stop hook intervention, max turns, unrecoverable error, etc.) and **7 continuation states** (tool calls pending, model wants more turns, etc.).

**DOMINION Implementation**: The master orchestrator loop must be an async generator with typed terminal states. Every agent's loop follows this same pattern.

---

## 4-Layer Context Compression

As conversations grow, context must be compressed to stay within the model's window. Claude Code implements 4 progressively heavier compression layers, plus a per-tool result budget (Layer 0).

| Layer | Name | Description | Cost |
|-------|------|-------------|------|
| 0 | **Tool Result Budget** | Per-tool output size limits (100K tokens). Prevents a single tool from consuming context. | Negligible |
| 1 | **Snip Compact** | Removes old tool results while preserving conversation structure. | Low |
| 2 | **Microcompact** | Aggressive inline compression of message content. | Medium |
| 3 | **Context Collapse** | Collapses entire conversation sections into summaries. | High |
| 4 | **Auto-Compact** | Forks an entire Claude conversation to summarize history. Circuit breaker after 3 consecutive failures. | Very High |

---

## The Tool System — Definition to Execution

Tools are not simple function wrappers. Each tool carries ~45 interface members across 5 critical categories: `call()`, `inputSchema`, `isConcurrencySafe()`, `checkPermissions()`, and `validateInput()`.

### The 14-Step Execution Pipeline

Every tool call passes through exactly 14 steps:
1. **Validation (Steps 1-4)**: Input parsing via Zod schema, JSON Schema check, semantic validation, abort signal check.
2. **Preparation (Steps 5-6)**: Context setup, pre-hooks fire (PreToolUse).
3. **Permission (Steps 7-9)**: Permission mode check (7 modes from `bypassPermissions` to `bubble`), rule matching against allow/deny lists, user prompt if required.
4. **Execution and Cleanup (Steps 10-14)**: Tool `call()` execution, result budgeting, post-hooks fire, message append, error handling.

---

## Concurrent Tool Execution

Claude Code uses **Speculative Tool Execution** and **Concurrent-Safe Batching** to parallelize operations safely.

1. **Partition Algorithm**: Tools are partitioned by their `isConcurrencySafe()` classification. Read operations (like `ls`, `cat`) run in parallel. Write operations (like `rm`, `FileEdit`) run serially.
2. **Streaming Executor**: Read-only tools begin execution *while the model is still streaming its response*, before the tool call block even completes.

---

## Sub-Agent Spawning — 15-Step Lifecycle

The `AgentTool` allows DOMINION to spawn sub-agents recursively. The lifecycle has 15 distinct steps, from context initialization to cleanup. Sub-agents run their own isolated `query()` loop, with their own token budgets and memory contexts, reporting back to the parent only upon completion or failure.

---

## Fork Agents and Prompt Cache Sharing

When DOMINION needs to spawn 24 agents, doing so naively would multiply token costs by 24x. Fork agents solve this using the **Byte-Identical Prefix Trick**.

**The 95% Insight**: 5 parallel children share ~80,000 tokens of prefix, with only ~200 tokens differing per child. Prompt caching gives a 90% discount, turning a $4 dispatch into $0.50.

But prompt caching is **byte-exact**. To achieve this, fork agents freeze three layers:
1. **System prompt via threading**: The child receives the parent's `renderedSystemPrompt` directly. It is not recomputed, avoiding divergence from remote feature flags.
2. **Tool definitions via exact passthrough**: The `useExactTools` flag passes the parent's exact tool array, bypassing the normal filtering logic that would change serialization.
3. **Message array via `buildForkedMessages()`**: Uses a constant `FORK_PLACEHOLDER_RESULT` string to ensure all tool results in the history are byte-identical across children.

---

## Tasks, Coordination, and Swarms

Tasks are background work units with a state machine (`pending → running → completed | failed | killed`). 

DOMINION uses **Coordinator Mode** to manage these tasks. The storage is a flat map (`Record<string, TaskState>`), not a tree, with parent-child relationships implicit via `toolUseId`. Swarm teammates communicate asynchronously through this task infrastructure.

---

## Memory — 4-Type Taxonomy with LLM Recall

An agent without memory makes the same mistakes forever. Claude Code uses file-based memory with a 4-type taxonomy:
1. **Project-level**: `CLAUDE.md` files in the repository.
2. **User-level**: `~/.claude/MEMORY.md`.
3. **Team-level**: Shared via symlinks.
4. **Session-level**: Ephemeral context.

**LLM Recall**: Instead of vector similarity search, a lightweight Sonnet model (256 max tokens) side-query selects relevant memories based on the current conversation context. The cost of this side-query is negligible compared to the context window savings of omitting irrelevant memories.

---

## Extensibility — Skills and Hooks

The system is extensible via Skills and Hooks.
- **Two-Phase Skill Loading**: Frontmatter loads at startup (cheap); full content loads only on invocation (expensive).
- **Hook System**: Fires at 27 distinct lifecycle events. WARDEN enforces a **Snapshot Security Model**, freezing hook configurations at startup to prevent runtime injection attacks.

---

## Input and Interaction Pipeline

Between raw terminal bytes and executed actions lies a 6-system pipeline: tokenizer → parser → classifier → keybinding resolver → chord state machine → handler.

- **Multi-Protocol Support**: Gracefully degrades across Kitty keyboard protocol, VT220, xterm modifyOtherKeys, and Windows Terminal.
- **Keybinding System**: Declarative configuration across 16 Contexts (chat, permission, search, etc.).
- **Chord Support**: Multi-key sequences (e.g., `Ctrl+X Ctrl+K` = killAgents) managed by a state machine with a 1000ms timeout and a `ChordInterceptor`.
- **Vim Mode**: Full state machine implementation with pure function transitions, persistent state, and dot-repeat.

---

## The Terminal UI: Custom Rendering Engine

Claude Code uses a custom fork of Ink to achieve 60fps streaming without garbage collection pauses.

- **Pool-Based Memory**: Zero per-frame object allocations for the 24,000-cell buffer.
  - `CharPool`: Interns character strings to integer IDs (fast ASCII path).
  - `StylePool`: Interns ANSI style arrays. Bit 0 encodes visibility on spaces.
  - `HyperlinkPool`: Interns OSC 8 URIs.
- **Cell Layout**: Two `Int32` words per cell in a contiguous `Int32Array`, with a `BigInt64Array` view for bulk operations (like clearing rows in microseconds).
- **Double-Buffer Rendering**: Front/back frame swap with damage rectangle tracking.

---

## Remote Control and Cloud Execution

DOMINION can be controlled remotely via 4 topologies:
1. **Bridge v1 (Poll-Based)**: Register → Poll → WebSocket reads / HTTP POST writes.
2. **Bridge v2 (Direct Sessions)**: Create session → SSE reads / CCRClient writes.
3. **Direct Connect**: WebSocket via `cc://` URL to a local CLI server.
4. **Upstream Proxy**: WebSocket tunnel with credential injection in containers.

**Security**: The Upstream Proxy uses `prctl(PR_SET_DUMPABLE, 0)` to block ptrace memory scraping, and unlinks the session token file immediately after reading.

---

## Performance Engineering

Performance is attacked across 5 dimensions using 50+ startup profiling checkpoints:
1. **Startup Latency**: Module-level I/O parallelism, API preconnection, fast-path dispatch.
2. **Token Efficiency**: Slot Reservation (8K default, 64K escalation), per-tool limits, per-message aggregate (200K chars).
3. **API Cost**: 3-tier prompt cache, sticky latches, memory relevance side-queries.
4. **Rendering Throughput**: Pool-based memory, packed typed arrays.
5. **Search Speed**: 26-bit bitmap pre-filter (4 bytes per path, ~1MB for 270K paths) rejects 90% of candidates with a single integer comparison before expensive scoring.

---

## MCP — The Universal Tool Protocol

Model Context Protocol (MCP) is the universal integration layer.
- **8 Transports**: stdio, SSE, WebSocket, etc.
- **OAuth for MCP**: Secure authentication for remote servers.
- **Tool Wrapping**: Normalizes all MCP tools into the internal `Tool` interface.

---

## The 5 Architectural Bets

1. Async generators over callback pipelines.
2. Two-tier state (mutable singleton + reactive store) over global Redux.
3. File-based memory with LLM recall over vector databases.
4. Byte-identical prefix threading over dynamic prompt generation.
5. Packed typed arrays over object-per-cell rendering.

---

## Implementation Mapping to ØMEGA Agents

This table maps every Claude Code architecture component to the specific ØMEGA AI agent responsible for implementing it.

| Architecture Component | ØMEGA Agent | Implementation Notes |
|----------------------|-------------|---------------------|
| `query()` async generator loop | **DOMINION (Ø)** | Main orchestration loop. All 25 agents inherit this pattern. |
| 5-Phase Bootstrap Pipeline | **DOMINION (Ø)** | 300ms budget, module-level I/O parallelism. |
| Two-Tier State & Sticky Latches | **DOMINION (Ø)** | Infrastructure singleton + AppState store. 5 latches for cache. |
| Multi-Provider API Routing | **DOMINION (Ø)** | Direct API, Bedrock, Vertex, Foundry with type erasure. |
| Dynamic Boundary System Prompts | **ALL AGENTS** | Static globally cached sections before boundary, dynamic after. |
| 4-layer context compression | **DOMINION (Ø)** | Manages context for all agent conversations. |
| Tool partition algorithm | **DOMINION (Ø)**, **ARCANE** | DOMINION partitions at orchestrator level; ARCANE partitions coding tools. |
| Streaming tool executor | **ARCANE**, **MODULUS** | Speculative execution for read-heavy coding and analysis tasks. |
| Sub-agent spawning (15 steps) | **DOMINION (Ø)** | Spawns all 24 sub-agents using this lifecycle. |
| Fork agents / cache sharing | **DOMINION (Ø)**, **PHANTOM** | Byte-identical prefix trick, 90% cache discount on parallel dispatch. |
| Coordinator mode | **DOMINION (Ø)** | Manager-worker hierarchy for complex multi-agent campaigns. |
| 4-type memory taxonomy | **ARCHIVE** | Persistent knowledge management across all sessions. |
| Memory recall pipeline | **ARCHIVE** | LLM-powered memory selection (Sonnet) at session start. |
| Two-phase skill loading | **ALL AGENTS** | Every agent loads skills via frontmatter-first pattern. |
| Hook system (27 events) | **WARDEN**, **SENTINEL** | Security hooks, permission enforcement, audit logging. |
| Snapshot security model | **WARDEN** | Freeze hook config at startup. |
| Input Pipeline & Chords | **DOMINION (Ø)** | 6-system pipeline, 1000ms chord timeout, Vim mode state machine. |
| Custom Ink Terminal UI | **DOMINION (Ø)** | 3 memory pools (Char, Style, Hyperlink), packed Int32 arrays. |
| Remote Execution (4 Topologies) | **DOMINION (Ø)** | Bridge v2, Direct Connect, Upstream Proxy for cloud deployment. |
| 26-bit Bitmap Search Filter | **ARCANE**, **ARCHIVE** | O(1) rejection for file/memory search. |
| Slot reservation (8K/64K) | **ALL AGENTS** | Context-efficient output management. |

---

> **This document is the architectural blueprint for DOMINION's brain. Every pattern, every abstraction, every implementation detail documented here must be reflected in the CodeSpring build. The source is [claude-code-from-source.com](https://claude-code-from-source.com/) — the most comprehensive reverse-engineering of a production AI agent ever published.**
