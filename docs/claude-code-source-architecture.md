# Claude Code Source Architecture — Complete Deep Dive

> **Source**: [claude-code-from-source.com](https://claude-code-from-source.com/) — the most comprehensive reverse-engineering of a production AI agent ever published.
> **Role in ØMEGA AI**: This is the **architectural blueprint for DOMINION's brain**. Every pattern, abstraction, and implementation detail documented here must be reflected in the CodeSpring build.

---

## Table of Contents

1. [The Six Key Abstractions](#the-six-key-abstractions)
2. [The Golden Path](#the-golden-path-from-keystroke-to-output)
3. [The Permission System (7 Modes)](#the-permission-system-7-modes)
4. [The Build System](#the-build-system)
5. [The Bootstrap Pipeline (5 Phases)](#the-bootstrap-pipeline-5-phases)
6. [Two-Tier State Architecture](#two-tier-state-architecture)
7. [The API Layer](#the-api-layer)
8. [The Agent Loop (query.ts)](#the-agent-loop-queryts)
9. [Context Compression (4 Layers)](#context-compression-4-layers)
10. [State Transitions (10 Terminal + 7 Continuation)](#state-transitions)
11. [Error Recovery: The Escalation Ladder](#error-recovery-the-escalation-ladder)
12. [The Tool System](#the-tool-system)
13. [The 14-Step Tool Execution Pipeline](#the-14-step-tool-execution-pipeline)
14. [Concurrent Tool Execution](#concurrent-tool-execution)
15. [Sub-Agent Spawning (15-Step Lifecycle)](#sub-agent-spawning-15-step-lifecycle)
16. [Built-In Agent Types](#built-in-agent-types)
17. [Fork Agents and Prompt Cache Sharing](#fork-agents-and-prompt-cache-sharing)
18. [Tasks, Coordination, and Swarms](#tasks-coordination-and-swarms)
19. [Memory System](#memory-system)
20. [Extensibility: Skills and Hooks](#extensibility-skills-and-hooks)
21. [The Terminal UI](#the-terminal-ui)
22. [Input and Interaction Pipeline](#input-and-interaction-pipeline)
23. [MCP — The Universal Tool Protocol](#mcp--the-universal-tool-protocol)
24. [Remote Control and Cloud Execution](#remote-control-and-cloud-execution)
25. [Performance Engineering](#performance-engineering)
26. [The Buddy/Companion System](#the-buddycompanion-system)
27. [The 5 Architectural Bets](#the-5-architectural-bets)
28. [5 Transferable Patterns](#5-transferable-patterns)
29. [Implementation Mapping to ØMEGA Agents](#implementation-mapping-to-ømega-agents)

---

## The Six Key Abstractions

Claude Code is a TypeScript monolith of nearly 2,000 files built on six core abstractions. Everything else — the 400+ utility files, the forked terminal renderer, the vim emulation, the cost tracker — exists to support these six.

| Abstraction | File(s) | Purpose |
|-------------|---------|---------|
| **Query Loop** | `query.ts` (~1,700 lines) | Async generator that is the heartbeat. Streams model response, collects tool calls, executes them, appends results, loops. Every interaction (REPL, SDK, sub-agent, headless `--print`) flows through this single function. Yields `Message` objects. Returns a discriminated union `Terminal` encoding exactly why the loop stopped. |
| **Tool System** | `Tool.ts`, `tools.ts`, `services/tools/` | ~45 interface members. Tools carry their own permission logic, concurrency declarations, progress reporting, and UI rendering. 40+ implementations. |
| **Tasks** | `Task.ts`, `tasks/` | Background work units (primarily sub-agents). State machine: `pending → running → completed | failed | killed`. Recursive capability: agents delegate to sub-agents. |
| **State** | Two layers | Mutable singleton `STATE` (~80 fields) for infrastructure. Minimal reactive store (34 lines, Zustand-shaped) for UI. |
| **Memory** | `memdir/` | Persistent context across sessions. 4-type taxonomy. File-based with LLM-powered recall. |
| **Hooks** | `hooks/`, `utils/hooks/` | User-defined lifecycle interceptors at 27+ distinct events across 4 execution types. Can block, modify, inject, or short-circuit. |

---

## The Golden Path: From Keystroke to Output

The data flow for every request follows this path:

1. User types prompt, presses Enter
2. REPL pulls messages from `query()` via `for await` (natural backpressure)
3. Query loop streams model response via API
4. `StreamingToolExecutor` starts concurrency-safe tools **during streaming** (speculative execution)
5. Tool results appended to message history
6. Loop calls model again with updated context
7. Model decides when done by not making more tool calls
8. Generator returns `Terminal` discriminated union with stop reason

The circular dependency between query loop and tool system is the system's defining characteristic. The generator pattern (rather than callbacks or event emitters) gives natural backpressure, clean cancellation, and typed terminal states.

---

## The Permission System (7 Modes)

| Mode | Behavior | ØMEGA Application |
|------|----------|-------------------|
| `bypassPermissions` | Everything allowed. No checks. Internal/testing only. | G0DM0D3 override for DOMINION |
| `dontAsk` | All allowed, but still logged. No user prompts. | Automated campaign execution |
| `auto` | Transcript classifier (LLM) decides allow/deny. Separate lightweight LLM call classifies tool invocation against conversation transcript. | Semi-autonomous agent operation |
| `acceptEdits` | File edits auto-approved; all other mutations prompt. | ARCANE code generation mode |
| `default` | Standard interactive mode. User approves each action. | Normal user interaction |
| `plan` | Read-only. All mutations blocked. | PHANTOM reconnaissance mode |
| `bubble` | Escalate decision to parent agent (sub-agent mode). Cannot approve own dangerous actions. | All 24 sub-agents default to this |

The resolution chain: Tool call → Check permission mode → If `auto`, run LLM classifier → If `bubble`, propagate to parent → If `default`, prompt user.

Sub-agents default to `bubble` mode, preventing them from silently running destructive commands the user never saw.

---

## The Build System

Claude Code ships as both an internal Anthropic tool and a public npm package using **Bun + esbuild**.

The `feature()` function from `bun:bundle` provides compile-time feature flags. At build time, each flag resolves to a boolean literal. The bundler's dead code elimination strips the `require()` call entirely when the flag is false — the module is never loaded, never included in the bundle, never shipped.

`require()` is used instead of `import()` specifically because dynamic `require()` can be fully eliminated by the bundler when the guard is false, while dynamic `import()` cannot (it returns a Promise that the bundler must preserve).

**Historical note**: The source maps published with early npm releases contained `sourcesContent` — the full original TypeScript source, including internal-only code paths. The feature flags successfully stripped the runtime code but left the source in the maps. This is how the Claude Code source became publicly readable.

---

## The Bootstrap Pipeline (5 Phases)

The system boots in ~300ms through 5 phases with module-level I/O parallelism:

| Phase | Action | Budget |
|-------|--------|--------|
| 1 | Parse CLI args, detect environment | ~20ms |
| 2 | Load config, resolve model, check auth | ~80ms |
| 3 | Initialize state, build system prompt | ~100ms |
| 4 | Connect to API, preconnect sockets | ~50ms |
| 5 | Enter REPL or execute `--print` | ~50ms |

There are **7 launch paths** (REPL, SDK, `--print`, `--resume`, sub-agent, headless, remote). All converge on `query()`.

The **trust boundary** is established during bootstrap: the user accepts or declines workspace trust, and this decision is frozen for the session.

---

## Two-Tier State Architecture

**Tier 1: `STATE` Singleton** (~80 fields, ~100 getters/setters). Mutable, set once at startup, mutated directly. No reactivity. Holds: working directory, model configuration, cost tracking, telemetry counters, session ID.

**Tier 2: `AppState` Reactive Store** (34 lines, Zustand-shaped). Drives the UI: messages, input mode, tool approvals, progress indicators. Changes constantly, must trigger re-renders.

**5 Sticky Latches** (once true, never revert to false — critical for prompt cache stability):

| Latch | Purpose |
|-------|---------|
| `hasUsedBash` | Unlocks shell-related prompt sections |
| `hasUsedFileEdit` | Unlocks file editing guidance |
| `hasUsedNotebook` | Unlocks notebook-related instructions |
| `hasUsedMcp` | Unlocks MCP tool guidance |
| `hasUsedWebSearch` | Unlocks web search instructions |

**NFC Path Normalization**: All paths are normalized to NFC Unicode form to prevent cache-busting from macOS NFD decomposition.

**Cost Tracking**: Running token counts and dollar costs tracked in STATE, updated after each API response.

---

## The API Layer

**Multi-Provider Client Factory**: `getAnthropicClient()` reads environment variables and returns the appropriate client. From that point, `callModel()` treats it as a generic Anthropic client.

| Provider | Transport | Notes |
|----------|-----------|-------|
| Direct API | HTTPS to api.anthropic.com | Default path |
| AWS Bedrock | Bedrock SDK wrapper | Same interface |
| Google Vertex | Vertex SDK wrapper | Same interface |
| Azure Foundry | Foundry SDK wrapper | Same interface |

**Type Erasure Pattern**: Provider-specific types are erased at the factory boundary. The query loop never checks which provider is active.

**The 2^N System Prompt Problem**: With N feature flags, there are 2^N possible system prompt combinations. Solution: **Dynamic Boundary Marker** — static globally-cached sections before the boundary, dynamic sections after. Only the dynamic portion changes between requests.

**`DANGEROUS_uncachedSystemPromptSection`**: Named deliberately to make developers think twice before adding content to the uncached section.

**3 Prompt Cache Tiers**:

| Tier | Scope | Mechanism |
|------|-------|-----------|
| API-level | Cross-request | Anthropic's server-side prompt caching (90% discount) |
| Local | In-process | Sticky latches prevent prompt changes |
| Fork | Cross-agent | Byte-identical prefix threading |

**Streaming**: Raw SSE over SDK for lower latency. **90-second idle watchdog** kills stalled connections.

---

## The Agent Loop (query.ts)

The file is ~1,730 lines. There is exactly one code path that talks to the model, executes tools, manages context, recovers from errors, and decides when to stop. The REPL calls it. The SDK calls it. Sub-agents call it. The headless runner calls it.

The loop is an **async generator** (`async function*`). It yields `Message` objects that the UI consumes. Its return type is a discriminated union called `Terminal` encoding exactly why the loop stopped.

**LoopState Object**: Full state object reconstructed at each `continue` — immutable transitions in a mutable loop. Self-documenting transitions, prevents partial updates.

**Token Budget Enforcement**: `checkTokenBudget` with rules: subagents always stop; continue if `turnTokens < budget * 0.9`; stop early if diminishing returns detected after 3+ continuations. When continuing, injects a nudge message about remaining budget.

**The Withholding Pattern**: Recoverable errors (like prompt-too-long 413) are withheld from yielding to prevent consumer disconnection during recovery.

**Model Fallback**: If the primary model fails, the system can fall back to alternative models.

---

## Context Compression (4 Layers)

Ordered to preserve granular context when possible, falling back to summaries:

| Layer | Name | Mechanism | Trigger |
|-------|------|-----------|---------|
| 0 | Tool Result Budget | Per-tool: 50K chars / 100K tokens. Per-conversation: 200K chars. Middle-out truncation (keep first/last N lines). | Always active |
| 1 | Snip Compact | Replaces old tool results with `[snipped]` markers. Preserves conversation structure. | Token count approaching limit |
| 2 | Microcompact | Removes entire message pairs that contribute least to current context. | Snip insufficient |
| 3 | Context Collapse | Summarizes entire conversation history into a compact representation. Staged collapses drained on 413. | Microcompact insufficient |
| 4 | Auto-Compact | Full LLM-powered summarization. Circuit breakers on failures. | Context collapse insufficient |

**Auto-Compact Thresholds**: Configurable. When triggered, the system uses a separate LLM call to summarize the conversation, then replaces the history with the summary.

---

## State Transitions

### 10 Terminal States

| Reason | Trigger |
|--------|---------|
| `blocking_limit` | Token count at hard limit, auto-compact OFF |
| `image_error` | ImageSizeError, ImageResizeError, or unrecoverable media error |
| `model_error` | Unrecoverable API/model exception |
| `aborted_streaming` | User abort during model streaming |
| `prompt_too_long` | Withheld 413 after all recovery exhausted |
| `completed` | Normal completion (no tool use, budget exhausted, or API error) |
| `stop_hook_prevented` | Stop hook explicitly blocked continuation |
| `aborted_tools` | User abort during tool execution |
| `hook_stopped` | PreToolUse hook stopped continuation |
| `max_turns` | Hit the `maxTurns` limit |

### 7 Continuation States

| Reason | Trigger |
|--------|---------|
| `collapse_drain_retry` | Context collapse drained staged collapses on 413 |
| `reactive_compact_retry` | Reactive compact succeeded after 413 or media error |
| `max_output_tokens_escalate` | 8K cap hit, escalating to 64K |
| `max_output_tokens_recovery` | 64K still hit, multi-turn recovery (up to 3 attempts) |
| `stop_hook_blocking` | Stop hook returned blocking errors, must retry |
| `token_budget_continuation` | Token budget not exhausted, nudge message injected |
| `next_turn` | Normal tool-use continuation |

---

## Error Recovery: The Escalation Ladder

Recovery follows a strict escalation with guards to prevent infinite loops:

1. **Reactive Compact**: `hasAttemptedReactiveCompact` guard. Compress context and retry.
2. **Token Escalation**: 8K → 64K output token cap.
3. **Multi-Turn Recovery**: Up to `MAX_OUTPUT_TOKENS_RECOVERY_LIMIT = 3` attempts.
4. **Context Collapse**: Drain staged collapses.
5. **Auto-Compact**: Full summarization with circuit breakers on failures.

**Death Spiral Guard**: Each recovery path has explicit limits. If all paths exhausted, the loop terminates with a terminal state rather than looping forever.

---

## The Tool System

The `Tool` interface has approximately **45 members**. Only 5 matter for understanding:

| Member | Purpose |
|--------|---------|
| `call()` | Execute the tool |
| `inputSchema` | Zod schema → JSON Schema for API + runtime `safeParse` |
| `isConcurrencySafe(input)` | Input-dependent: same tool safe for some inputs, unsafe for others |
| `checkPermissions()` | Tool-specific logic, runs AFTER general permission system |
| `validateInput()` | Semantic validation |

**Three Type Parameters**: `Tool<Input extends AnyObject, Output, P extends ToolProgressData>` where `P` is the progress event type (BashTool=stdout chunks, GrepTool=match counts, AgentTool=sub-agent transcripts).

**`buildTool()` Factory — Fail-Closed Defaults**:

| Default | Value | Rationale |
|---------|-------|-----------|
| `isParallelSafe` | `false` | New tools run serially by default |
| `isReadOnly` | `false` | Treated as writes by default |
| `isDestructive` | `false` | Safe default |
| `checkPermissions` | `allow` | Not fail-closed because runs AFTER general system |

**`ToolResult<T>` Return Type**:
- `data`: Typed output → serialized into `tool_result` content block
- `newMessages`: Inject additional messages (AgentTool appends sub-agent transcripts)
- `contextModifier`: Mutate `ToolUseContext` for subsequent tools (e.g., `EnterPlanMode` switches permission mode). Only honored for non-concurrency-safe tools.

**`ToolUseContext` (~40 fields)**: Configuration (options sub-object), Execution state (abortController, readFileState LRU cache, messages), UI callbacks (setToolJSX, addNotification, requestPrompt — undefined in SDK/headless), Agent context (agentId, renderedSystemPrompt frozen for fork sub-agents).

**Registry: `getAllBaseTools()`**: Always-present tools first, then feature-flag gated tools. `assembleToolPool()`: built-in (deny-rule filtered) + MCP tools, sorted alphabetically, concatenated. Sort order matters for prompt cache stability.

**Result Budgeting**: Per-tool 50K chars / 100K tokens. Per-conversation aggregate 200K chars. Truncation: middle-out (keep first/last N lines, replace middle with "[X lines truncated]").

**Individual Tool Highlights**:

| Tool | Key Feature |
|------|-------------|
| **BashTool** | Most complex. Classifies subcommands against known-safe sets. Input-dependent concurrency. |
| **FileEditTool** | Staleness detection (checks file hasn't changed since last read) |
| **FileReadTool** | Versatile reader (text, binary, images, directories) |
| **GrepTool** | Pagination via `head_limit` parameter |
| **AgentTool** | Spawns sub-agents with context modifiers |

---

## The 14-Step Tool Execution Pipeline

| Steps | Phase | Actions |
|-------|-------|---------|
| 1-4 | **Validation** | Zod parse, JSON Schema validation, semantic validation, abort check |
| 5-6 | **Preparation** | Context setup, PreToolUse hooks fire |
| 7-9 | **Permission** | Mode check, rule matching, bubble escalation |
| 10-14 | **Execution** | `call()`, result budgeting, PostToolUse hooks, cleanup, yield |

---

## Concurrent Tool Execution

**Partition Algorithm**: Tools partitioned by `isConcurrencySafe()` classification. Read operations (`ls`, `cat`) run in parallel. Write operations (`rm`, `FileEdit`) run serially.

**Streaming Executor**: Read-only tools begin execution **while the model is still streaming its response**, before the tool call block even completes. This is speculative execution — if the model's final output invalidates the tool call (rare), the result is discarded.

**Concurrency is Input-Dependent**: BashTool is the canonical example: `ls -la` is read-only and concurrency-safe, but `rm -rf /tmp/build` is not. The tool parses the command, classifies each subcommand, and returns `true` only when every part is a search or read operation.

---

## Sub-Agent Spawning (15-Step Lifecycle)

The `runAgent()` function in `runAgent.ts` is an async generator (~400 lines) that drives every sub-agent's lifecycle. Every sub-agent — fork, built-in, custom, coordinator worker — flows through this single function.

| Step | Action | Detail |
|------|--------|--------|
| 1 | **Model Resolution** | Chain: agent preference → parent model → caller override → permission mode |
| 2 | **Agent ID Creation** | Unique ID or reuse override if resuming |
| 3 | **Context Preparation** | Clone file state for forks, fresh for new agents. Filter incomplete tool calls. |
| 4 | **CLAUDE.md Stripping** | Omit for read-only agents (feature-flag gated via `tengu_slim_subagent_claudemd`) |
| 5 | **Permission Isolation** | Overlay permission modes, scope `allowedTools` |
| 6 | **Tool Selection** | `useExactTools` for forks (parent's exact array), `resolveAgentTools()` for others |
| 7 | **System Prompt** | Generate or reuse override (frozen for forks to preserve cache) |
| 8 | **Abort Controller** | Shared for sync agents, new `AbortController()` for async agents |
| 9 | **Hook Registration** | `registerFrontmatterHooks()` if agent definition includes hooks |
| 10 | **Skill Preloading** | Load specified skills into conversation |
| 11 | **MCP Initialization** | `initializeAgentMcpServers()` for custom MCP servers |
| 12 | **Context Creation** | `createSubagentContext()` with isolation choices per field |
| 13 | **Cache-Safe Params** | Invoke callback for background summarization |
| 14 | **The Query Loop** | `for await (const message of query({...}))` — yield messages, record to transcript |
| 15 | **Cleanup** | MCP cleanup, hook deregistration, file state clear, shell task kill, perfetto unregister, agent tracking removal |

---

## Built-In Agent Types

| Type | Mode | Purpose |
|------|------|---------|
| **General-Purpose** | Default | Standard delegation for any task |
| **Explore** | Read-only | Codebase exploration without mutations |
| **Plan** | Read-only | Planning mode, generates strategy without executing |
| **Verification** | Adversarial | Verification pass that challenges assumptions |
| **Claude Code Guide** | Read-only | Help and documentation agent |
| **Statusline Setup** | Limited | Terminal statusline configuration |
| **Worker Agent** | Coordinator | Managed worker in coordinator-worker hierarchy |

**5 Design Dimensions for Agent Types**:

| Dimension | Question |
|-----------|----------|
| 1 | What Can It See? (context scope) |
| 2 | What Can It Do? (tool restrictions) |
| 3 | How Does It Interact with the User? (permission prompts) |
| 4 | How Does It Relate to the Parent? (sync/async, bubble mode) |
| 5 | How Expensive Is It? (model selection, token budgets) |

---

## Fork Agents and Prompt Cache Sharing

**The 95% Insight**: 5 parallel children share ~80,000 tokens of prefix, with only ~200 tokens differing per child. Prompt caching gives a 90% discount, turning a **$4 dispatch into $0.50**.

Prompt caching is **byte-exact**. To achieve this, fork agents freeze three layers:

| Layer | Mechanism | Detail |
|-------|-----------|--------|
| 1 | **System prompt via threading** | Child receives parent's `renderedSystemPrompt` directly. Not recomputed, avoiding divergence from remote feature flags. |
| 2 | **Tool definitions via exact passthrough** | `useExactTools` flag passes parent's exact tool array, bypassing normal filtering that would change serialization. |
| 3 | **Message array via `buildForkedMessages()`** | Uses constant `FORK_PLACEHOLDER_RESULT` string to ensure all tool results are byte-identical across children. |

**Recursive Fork Prevention**: Fork agents cannot spawn their own forks (prevents exponential explosion).

**GrowthBook Feature Flags**: Remote feature flags can warm up asynchronously, causing system prompt divergence between parent and child. Threading the parent's rendered prompt eliminates this.

---

## Tasks, Coordination, and Swarms

Tasks are background work units with state machine: `pending → running → completed | failed | killed`.

**Storage**: Flat map (`Record<string, TaskState>`), not a tree. Parent-child relationships implicit via `toolUseId`.

**Coordinator Mode**: DOMINION manages workers. The coordinator dispatches tasks, monitors progress, and aggregates results. Workers communicate asynchronously through the task infrastructure.

---

## Memory System

### The 4-Type Taxonomy

| Type | What It Captures | Example |
|------|-----------------|---------|
| **User** | Information about the person (role, goals, expertise) | "Senior Go engineer, new to React" |
| **Feedback** | Guidance about approach (corrections AND confirmations) | "Integration tests must hit real DB, not mocks" |
| **Project** | Ongoing work context (who, what, why, when) | "Auth migration due 2026-03-15" |
| **Reference** | Bookmarks to external systems | "Grafana dashboard: https://..." |

**Derivability Test**: Can this knowledge be re-derived from the current project state? If yes, don't save it. Code patterns, git history, architecture diagrams — all excluded.

### Frontmatter Contract

```yaml
---
name: {{memory name}}
description: {{one-line description -- used to decide relevance}}
type: {{user, feedback, project, reference}}
---
```

The `description` is the most load-bearing field — it's the memory's search index, consumed by an LLM that understands nuance.

### Write Path (2-Step Protocol)

1. Write `.md` file with YAML frontmatter using `FileWriteTool`
2. Update `MEMORY.md` index with one-line pointer (~150 chars max)

No special memory API exists. The model uses the same tools it uses for code editing.

### Recall Path

**Two-tier architecture**: MEMORY.md index always loaded (lightweight). Individual files surfaced on-demand via LLM side-query.

**The Sonnet Side-Query**: Lightweight Sonnet model (256 max tokens) receives the manifest of memory descriptions and selects up to 5 relevant memories per turn. Structured output: `{ selected_memories: string[] }`. Async prefetch hides latency.

### MEMORY.md Index Caps

| Cap | Value | Reason |
|-----|-------|--------|
| Line cap | 200 lines | Catches normal growth |
| Byte cap | 25,000 bytes | Catches long lines (197 lines = 197KB at 97th percentile) |

### Staleness System

Computes memory age in days. Today/yesterday: no warning. Older: caveat injected ("47 days ago — code claims may be outdated, verify against current code"). Human-readable format triggers staleness reasoning better than ISO timestamps (validated: 3/3 vs 0/3 in evals).

### KAIROS Mode (Long-Lived Sessions)

Append-only daily logs: `<autoMemPath>/logs/YYYY/MM/YYYY-MM-DD.md`. Short timestamped bullets, no reorganization during capture. Path described as pattern (not literal date) for cache optimization.

**`/dream` Consolidation** (4 phases):

| Phase | Action |
|-------|--------|
| Orient | List directory, read index, skim existing files |
| Gather | Search logs, check for drifted memories |
| Consolidate | Write/update files, merge rather than duplicate |
| Prune | Update index under 200 lines, remove stale pointers |

**Consolidation Lock**: `.consolidate-lock` file. Content = PID (mutual exclusion). Mtime = `lastConsolidatedAt`. Auto-dream fires when: hours since last > 24, sessions modified > 5, no lock held. Crash recovery: `process.kill(pid, 0)` with 1-hour staleness timeout.

### Background Extraction

Forked agent at end of each query loop. Shares parent's prompt cache. Analyzes recent messages, writes memories main agent missed. Constrained tool budget: read-only + write only to memory directory. Two-turn strategy: turn 1 reads in parallel, turn 2 writes in parallel. Cooperative: defers when main agent already saved.

### Team Memory

Subdirectory `<autoMemPath>/team/`. Feature-flag gated. 3-layer security:

| Layer | Defense |
|-------|---------|
| 1 | Input sanitization (null bytes, URL-encoded traversals, Unicode normalization attacks, backslashes, absolute paths) |
| 2 | String-level path validation (`path.resolve()` + prefix check with trailing separator) |
| 3 | Symlink resolution (`realpathDeepestExisting()` catches symlink attacks) |

All failures produce `PathTraversalError`. Fail closed.

### Path Resolution Priority

| Priority | Source | Notes |
|----------|--------|-------|
| 1 | `CLAUDE_COWORK_MEMORY_PATH_OVERRIDE` | Full-path override for Cowork |
| 2 | `autoMemoryDirectory` in settings.json | Trusted sources only (NOT project settings — prevents malicious repos) |
| 3 | Default computed | `~/.claude/projects/<sanitized-git-root>/memory/` |

---

## Extensibility: Skills and Hooks

### Skills: 7 Sources with Priority

| Priority | Source | Location |
|----------|--------|----------|
| 1 | Managed (Policy) | `<MANAGED_PATH>/.claude/skills/` |
| 2 | User | `~/.claude/skills/` |
| 3 | Project | `.claude/skills/` (walked up to home) |
| 4 | Additional Dirs | `<add-dir>/.claude/skills/` |
| 5 | Legacy Commands | `.claude/commands/` |
| 6 | Bundled | Compiled into binary |
| 7 | MCP | MCP server prompts (untrusted) |

**Two-Phase Loading**: Frontmatter loads at startup (cheap); full content loads only on invocation (expensive).

**Dynamic Discovery**: `discoverSkillDirsForPaths` walks up from touched paths. Skills with `paths` frontmatter activate only when touched paths match patterns — context-sensitive capability expansion.

### Skill Frontmatter Fields

| Field | Purpose |
|-------|---------|
| `name` | User-facing display name |
| `description` | Shown in autocomplete and system prompt |
| `when_to_use` | Detailed usage scenarios for model discovery |
| `allowed-tools` | Which tools the skill can use |
| `disable-model-invocation` | Block autonomous model use |
| `context` | `'fork'` to run as sub-agent |
| `hooks` | Lifecycle hooks registered on invocation |
| `paths` | Glob patterns for conditional activation |

**MCP Security Boundary**: MCP skills **never** execute inline shell commands. MCP servers are external systems treated as content-only.

### Hooks: 6 Types

| Type | User-Configurable | Mechanism |
|------|-------------------|-----------|
| **Command** | Yes | Spawn shell process, stdin=JSON, communicate via exit code + stdout/stderr |
| **Prompt** | Yes | Single LLM call, returns `{ok: true/false, reason}` |
| **Agent** | Yes | Multi-turn agentic loop (max 50 turns, `dontAsk`, thinking disabled) |
| **HTTP** | Yes | POST to URL for remote policy servers and audit logging |
| **Callback** | No (internal) | Registered programmatically, -70% overhead fast path |
| **Function** | No (internal) | Session-scoped TypeScript callbacks |

### Complete Hook Lifecycle Events

**5 Most Important Events**:

| Event | Fires | Can Block? | Key Use |
|-------|-------|------------|---------|
| `PreToolUse` | Before every tool execution | Yes | Block, modify input, auto-approve, inject context |
| `PostToolUse` | After successful execution | No | Inject context, replace MCP output |
| `Stop` | Before Claude concludes | Yes (forces continuation) | Automated verification loops |
| `SessionStart` | At session beginning | No | Set env vars, override first message, file watches |
| `UserPromptSubmit` | When user submits prompt | Yes | Input validation, content filtering |

**All Remaining Events**:

| Category | Events |
|----------|--------|
| Tool lifecycle | `PostToolUseFailure`, `PermissionDenied`, `PermissionRequest` |
| Session | `SessionEnd` (1.5s timeout), `Setup` |
| Subagent | `SubagentStart`, `SubagentStop` |
| Compaction | `PreCompact`, `PostCompact` |
| Notification | `Notification`, `Elicitation`, `ElicitationResult` |
| Configuration | `ConfigChange`, `InstructionsLoaded`, `CwdChanged`, `FileChanged`, `TaskCreated`, `TaskCompleted`, `TeammateIdle` |

### Exit Code Semantics

| Code | Meaning | Blocks? |
|------|---------|---------|
| 0 | Success, stdout parsed if JSON | No |
| 2 | Blocking error, stderr shown as system message | Yes |
| Other | Non-blocking warning, shown to user only | No |

Exit code 2 chosen deliberately — exit 1 is too common (any unhandled exception produces it).

### 6 Hook Sources

| Source | Trust Level | Notes |
|--------|-------------|-------|
| `userSettings` | User | `~/.claude/settings.json`, highest priority |
| `projectSettings` | Project | `.claude/settings.json`, version-controlled |
| `localSettings` | Local | `.claude/settings.local.json`, gitignored |
| `policySettings` | Enterprise | Cannot be overridden |
| `pluginHook` | Plugin | Priority 999 (lowest) |
| `sessionHook` | Session | In-memory only, registered by skills |

### Snapshot Security Model

`captureHooksConfigSnapshot()` called once at startup. `executeHooks()` reads from snapshot, never re-reads settings. Updated only through `/hooks` command or file watcher detection.

Policy cascade: `disableAllHooks` clears everything. `allowManagedHooksOnly` excludes user/project hooks. Enterprise-managed hooks cannot be disabled by users.

### Permission Behavior Precedence

`deny > ask > allow` — deny always wins. `executePreToolHooks()` can: block, auto-approve, force ask, deny, modify input, add context.

### Stop Hooks: Forcing Continuation

When a Stop hook returns exit code 2, stderr is shown to the model as feedback and the conversation continues. This turns a single-shot prompt-response into a goal-directed loop. **The most powerful integration point in the system.**

---

## The Terminal UI

Custom fork of Ink achieving 60fps streaming without GC pauses.

**3 Memory Pools (zero per-frame allocations for 24,000-cell buffer)**:

| Pool | Purpose |
|------|---------|
| `CharPool` | Interns character strings to integer IDs (fast ASCII path) |
| `StylePool` | Interns ANSI style arrays. Bit 0 encodes visibility on spaces. |
| `HyperlinkPool` | Interns OSC 8 URIs |

**Cell Layout**: Two `Int32` words per cell in a contiguous `Int32Array`, with a `BigInt64Array` view for bulk operations (clearing rows in microseconds).

**Double-Buffer Rendering**: Front/back frame swap with damage rectangle tracking. Only changed regions are redrawn.

**Markdown Rendering Pipeline**: Converts model output to terminal-renderable Markdown with syntax highlighting, code blocks, and inline formatting.

---

## Input and Interaction Pipeline

Between raw terminal bytes and executed actions lies a **6-system pipeline**:

| Stage | System | Purpose |
|-------|--------|---------|
| 1 | Tokenizer | Raw bytes → tokens |
| 2 | Parser | Tokens → key events |
| 3 | Classifier | Key events → intent classification |
| 4 | Keybinding Resolver | Intent → action lookup across 16 contexts |
| 5 | Chord State Machine | Multi-key sequences (1000ms timeout, `ChordInterceptor`) |
| 6 | Handler | Action → execution |

**5 Terminal Protocols** (graceful degradation):

| Protocol | Level |
|----------|-------|
| Kitty keyboard protocol | Best (full modifier support) |
| xterm modifyOtherKeys | Good |
| VT220 | Basic |
| Windows Terminal | Platform-specific |
| Raw mode fallback | Minimal |

**16 Keybinding Scopes**: chat, permission, search, vim-normal, vim-insert, vim-visual, etc.

**Vim Mode**: Full state machine implementation with pure function transitions, persistent state, and dot-repeat.

---

## MCP — The Universal Tool Protocol

| Aspect | Detail |
|--------|--------|
| Transports | 8 types: stdio, SSE, WebSocket, streamable HTTP, etc. |
| Scopes | 7: user, project, local, managed, bundled, agent frontmatter, remote |
| OAuth | Secure authentication for remote servers |
| Tool Wrapping | Normalizes all MCP tools into internal `Tool` interface |
| Security | MCP skills never execute inline shell commands |

**Transport Selection Logic**: Determined by server configuration. stdio for local processes, SSE/WebSocket for remote servers.

---

## Remote Control and Cloud Execution

4 topologies for remote operation:

| Topology | Mechanism | Use Case |
|----------|-----------|----------|
| **Bridge v1 (Poll-Based)** | Register → Poll → WebSocket reads / HTTP POST writes | Legacy |
| **Bridge v2 (Direct Sessions)** | Create session → SSE reads / CCRClient writes | Current default |
| **Direct Connect** | WebSocket via `cc://` URL to local CLI server | Local network |
| **Upstream Proxy** | WebSocket tunnel with credential injection in containers | Cloud deployment |

**Asymmetric Read/Write Design**: Reads and writes use different protocols optimized for their access patterns.

**Security**:
- `prctl(PR_SET_DUMPABLE, 0)` blocks ptrace memory scraping
- Token file unlinked immediately after reading
- Protobuf hand-encoding for wire protocol
- `FlushGate`: Ensures all pending writes complete before session teardown
- `BoundedUUIDSet`: Prevents replay attacks with bounded memory

---

## Performance Engineering

Performance attacked across **5 dimensions** with **50+ startup profiling checkpoints**:

| Dimension | Techniques |
|-----------|-----------|
| **Startup Latency** | Module-level I/O parallelism, API preconnection, fast-path dispatch |
| **Token Efficiency** | Slot Reservation (8K default, 64K escalation), per-tool limits, per-message aggregate (200K chars) |
| **API Cost** | 3-tier prompt cache, sticky latches, memory relevance side-queries |
| **Rendering Throughput** | Pool-based memory, packed typed arrays, damage rectangles |
| **Search Speed** | 26-bit bitmap pre-filter (4 bytes per path, ~1MB for 270K paths) rejects 90% of candidates with single integer comparison |

**Context Window Sizing**: 200K default, 1M via suffix models.

---

## The Buddy/Companion System

> Source: [grayashh/buddy-reroll](https://github.com/grayashh/buddy-reroll)

A procedural companion generation system that creates unique pixel-art buddies using deterministic algorithms.

**Core Algorithm — Mulberry32 PRNG**: Seeded random number generator ensures the same seed always produces the same buddy. Uses 32-bit integer arithmetic for cross-platform consistency.

**Generation Pipeline**:

| Component | Method | Values |
|-----------|--------|--------|
| Species | `seededRandom() * species.length` | Cat, Dog, Bunny, Fox, Owl, Frog, Penguin, Bear |
| Rarity | Weighted: Common 50%, Uncommon 30%, Rare 15%, Legendary 5% | Determines stat multipliers |
| Eyes | Random from species-specific set | 4-8 variants per species |
| Hat | Random from universal set | ~12 options including None |
| Shiny | 1/100 chance | Golden palette swap |
| Stats | 5 stats with peak/dump system | HP, ATK, DEF, SPD, LCK |

**Stat Generation**: Peak stat gets +3 bonus, dump stat gets -2 penalty. Ensures each buddy has a distinct personality.

**Binary Patching**: Sprite modifications applied via coordinate-based patching rather than full redraws. Each eye/hat variant is a set of `[x, y, colorIndex]` patches applied to the base species sprite.

**Sprite Rendering**: 16x16 pixel grid with indexed color palettes. Each species has a base sprite, color palette, and patch sets for customization.

**ØMEGA Application**: The buddy system demonstrates procedural generation patterns applicable to DOMINION's agent personality system — each of the 25 agents could have procedurally generated visual identities using similar deterministic algorithms.

---

## The 5 Architectural Bets

| Bet | Choice | Over |
|-----|--------|------|
| 1 | Async generators | Callback pipelines |
| 2 | Two-tier state (mutable singleton + reactive store) | Global Redux |
| 3 | File-based memory with LLM recall | Vector databases |
| 4 | Byte-identical prefix threading | Dynamic prompt generation |
| 5 | Packed typed arrays | Object-per-cell rendering |

---

## 5 Transferable Patterns

These patterns from Claude Code's architecture transfer to any agentic system:

| Pattern | Principle | Problem It Solves |
|---------|-----------|-------------------|
| **Generator Loop** | Use async generator as agent loop, not callbacks | Callbacks make it difficult to know when/why the loop stopped |
| **Self-Describing Tools** | Every tool declares own concurrency, permissions, rendering | Central orchestrator becomes god object updated for every new tool |
| **Two-Tier State** | Separate infrastructure state from reactive state | Making everything reactive adds overhead to state that changes once |
| **Permission Modes** | Named modes (plan, default, auto, bypass), not scattered checks | Inconsistent enforcement when checks are scattered through tools |
| **Recursive Agents** | Sub-agents are same loop with own history, not special-cased paths | Divergent sub-agent logic leads to subtle behavior differences |

---

## Implementation Mapping to ØMEGA Agents

| Architecture Component | ØMEGA Agent | Implementation Notes |
|----------------------|-------------|---------------------|
| `query()` async generator loop | **DOMINION (Ø)** | Main orchestration loop. All 25 agents inherit this pattern. |
| 5-Phase Bootstrap Pipeline | **DOMINION (Ø)** | 300ms budget, module-level I/O parallelism. |
| Two-Tier State & 5 Sticky Latches | **DOMINION (Ø)** | Infrastructure singleton + AppState store. |
| Multi-Provider API Routing | **DOMINION (Ø)** | Direct API, Bedrock, Vertex, Foundry with type erasure. |
| 7 Permission Modes | **WARDEN**, **SENTINEL** | Mode-based resolution chain with bubble escalation. |
| Dynamic Boundary System Prompts | **ALL AGENTS** | Static cached before boundary, dynamic after. |
| 4-Layer Context Compression | **DOMINION (Ø)** | Manages context for all agent conversations. |
| Tool Partition Algorithm | **DOMINION (Ø)**, **ARCANE** | Partition at orchestrator and coding tool level. |
| Streaming Tool Executor | **ARCANE**, **MODULUS** | Speculative execution for read-heavy tasks. |
| 14-Step Tool Pipeline | **ALL AGENTS** | Every tool call flows through same pipeline. |
| Sub-Agent Spawning (15 steps) | **DOMINION (Ø)** | Spawns all 24 sub-agents using this lifecycle. |
| 7 Built-In Agent Types | **DOMINION (Ø)** | Maps to ØMEGA agent specializations. |
| Fork Agents / Cache Sharing | **DOMINION (Ø)**, **PHANTOM** | Byte-identical prefix trick, 90% cache discount. |
| Coordinator Mode | **DOMINION (Ø)** | Manager-worker hierarchy for campaigns. |
| 4-Type Memory Taxonomy | **ARCHIVE** | Persistent knowledge management. |
| Memory Recall Pipeline | **ARCHIVE** | Sonnet side-query at session start. |
| Staleness System | **ARCHIVE** | Age warnings on old memories. |
| KAIROS Mode & /dream | **ARCHIVE** | Long-session memory with consolidation. |
| Background Extraction | **ARCHIVE** | Forked agent catches missed memories. |
| Team Memory | **ARCHIVE**, **WARDEN** | Shared memory with 3-layer security. |
| 7 Skill Sources | **ALL AGENTS** | Priority-based skill loading. |
| Two-Phase Skill Loading | **ALL AGENTS** | Frontmatter-first pattern. |
| 27+ Hook Events | **WARDEN**, **SENTINEL** | Security hooks, permission enforcement, audit. |
| 6 Hook Types | **WARDEN** | Command, Prompt, Agent, HTTP, Callback, Function. |
| Snapshot Security Model | **WARDEN** | Freeze hook config at startup. |
| Stop Hooks (Forcing Continuation) | **ALL AGENTS** | Goal-directed loops via exit code 2. |
| Custom Ink Terminal UI | **DOMINION (Ø)** | 3 pools, packed Int32 arrays, double-buffer. |
| Input Pipeline (6 systems) | **DOMINION (Ø)** | Tokenizer → Parser → Classifier → Resolver → Chord → Handler. |
| Vim Mode State Machine | **DOMINION (Ø)** | Pure function transitions, dot-repeat. |
| Remote Execution (4 Topologies) | **DOMINION (Ø)** | Bridge v2, Direct Connect, Upstream Proxy. |
| 26-bit Bitmap Search Filter | **ARCANE**, **ARCHIVE** | O(1) rejection for file/memory search. |
| Slot Reservation (8K/64K) | **ALL AGENTS** | Context-efficient output management. |
| Build System (Bun + esbuild) | **ARCANE** | Feature flags, dead code elimination. |
| Buddy/Companion System | **DOMINION (Ø)** | Procedural agent identity generation. |
| Error Recovery Escalation | **ALL AGENTS** | Death spiral guard, recovery limits. |

---

> **This document is the complete architectural blueprint for DOMINION's brain. Every pattern, every abstraction, every table, every implementation detail from [claude-code-from-source.com](https://claude-code-from-source.com/) is captured here with ØMEGA AI applications mapped. Nothing from the source has been omitted.**
