# Claude Code: Advanced Architectural & Prompt Patterns

> **Source:** `src.zip` (Anthropic Claude Code internal source analysis)
> **Status:** Research & Integration Blueprint
> **Context:** Advanced patterns for prompt cache efficiency, persistent memory, and subagent orchestration.

---

## 1. Prompt Cache Optimization: The "Attachment" Pattern

Claude Code uses a sophisticated **Attachment System** to move volatile, session-specific context out of the static system prompt. This maximizes the reuse of the expensive system prompt cache.

### 1.1 Static vs. Dynamic Boundary
The system prompt is split by a `SYSTEM_PROMPT_DYNAMIC_BOUNDARY`. 
- **Static (Global Cache)**: Core instructions, tool schemas, and coding standards.
- **Dynamic (Session Cache)**: User preferences, environment details, and current task state.

### 1.2 Context Injection via Attachments
Instead of bloating the system prompt with every available tool or agent, Claude Code injects them as **`agent_listing_delta`** or **`mcp_instructions_delta`** attachments within the conversation.
- **Benefit**: Changing an MCP server or adding a custom agent doesn't bust the 20K+ token system prompt cache.
- **Mechanism**: The `attachments.ts` utility dynamically constructs `<system-reminder>` messages to surface relevant context only when needed.

---

## 2. Persistent Memory Architecture

Claude Code implements two distinct layers of persistent memory, both managed via structured Markdown files.

### 2.1 Session Memory (The "Live" State)
Managed in `src/services/SessionMemory/prompts.ts`, this tracks the immediate state of the current session.
- **Structure**: Fixed headers (Session Title, Current State, Task, Workflow, Learnings).
- **Update Contract**: The agent is strictly instructed to use the `Edit` tool to update sections in place, preserving italicized instructions.
- **Compaction**: Automatically triggers when sections exceed token budgets (e.g., 200 lines).

### 2.2 Persistent Knowledge (The "Memory" Subagent)
Managed in `src/services/extractMemories/prompts.ts`, this uses a dedicated **background subagent** to distill long-term knowledge.
- **Strategy**: Runs as a **fork** of the main conversation to avoid context bloat.
- **Batching**: Uses a "Read-then-Write" strategy (parallel reads in turn 1, parallel edits in turn 2).
- **Taxonomy**: Distinguishes between **Auto-Memory** (per-user) and **Team-Memory** (shared across a repository).

---

## 3. Subagent Orchestration: The "Fork" Pattern

The `AgentTool` (documented in `src/tools/AgentTool/prompt.ts`) introduces a critical distinction between **Fresh Agents** and **Forks**.

| Pattern | Context | Prompt Style | Use Case |
|---------|---------|--------------|----------|
| **Fresh Agent** | Zero context | **Briefing**: Full background, goals, and constraints. | Independent tasks (e.g., "Review this migration"). |
| **Fork** | Inherited context | **Directive**: Specific "what to do" instructions. | Parallel research or heavy tool-use tasks. |

### 3.1 Orchestration Rules
- **Never Delegate Understanding**: Prompts must include specific file paths and line numbers; don't ask the agent to "find and fix."
- **Don't Peek**: The coordinator should not read the fork's transcript mid-flight to avoid context noise.
- **Don't Race**: The coordinator must wait for the formal notification message before summarizing results.

---

## 4. Integration into ØMEGA AI

These patterns will be integrated into the **Omega Master Orchestrator (PRIME)** and the **Coding Subagent**.

1.  **Omega Attachment Layer**: Implement a similar "Attachment" system to keep the PRIME system prompt static.
2.  **Memory Extraction Fork**: Deploy a background "Memory Agent" to maintain the `claudemd.js` style persistent memory.
3.  **Directive Prompting**: Standardize on "Directive" style prompts for forked subagents to reduce token overhead.
