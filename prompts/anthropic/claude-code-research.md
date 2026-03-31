# Claude Code Research & Architectural Patterns

> **Source:** [instructkr/claw-code](https://github.com/instructkr/claw-code)
> **Status:** Research / Porting Workspace
> **Context:** Clean-room Python rewrite of the exposed Claude Code agent harness.

---

## 1. System Overview

The `claw-code` project is a Python-first reimplementation of the "Claude Code" agent harness. It focuses on the architectural patterns of how a high-capability coding agent wires tools, orchestrates tasks, and manages runtime context.

### Key Metrics (from mirrored snapshot)
- **Total TS-like files:** 1902
- **Command entries:** 207
- **Tool entries:** 184

---

## 2. Architectural Subsystems

The system is organized into specialized subsystems, mirroring the original TypeScript architecture:

| Subsystem | Responsibility | Notable Modules |
|-----------|----------------|-----------------|
| **Assistant** | Session and history management | `sessionHistory.ts` |
| **Constants** | Core behavioral rules and prompts | `prompts.ts`, `system.ts`, `systemPromptSections.ts`, `xml.ts` |
| **Services** | Domain-specific logic and API integration | `MagicDocs`, `PromptSuggestion`, `SessionMemory`, `claude.ts` |
| **Skills** | Extensible agent capabilities | `claudeApi`, `claudeInChrome`, `debug`, `remember`, `verify` |
| **Tools** | Atomic execution units | `AgentTool`, `BashTool`, `FileEditTool`, `AskUserQuestionTool` |
| **Commands** | User-facing CLI operations | `add-dir`, `advisor`, `agents`, `branch`, `doctor` |

---

## 3. Tool & Command Patterns

### Tool Surface
The agent uses a sophisticated tool pool with specific validation and permission logic.
- **AgentTool:** Orchestrates sub-agents (e.g., `planAgent`, `verificationAgent`, `exploreAgent`).
- **BashTool:** Handles shell execution with security guardrails and `sed` edit parsing.
- **AskUserQuestionTool:** Manages human-in-the-loop interactions.

### Command Surface
Commands provide the primary interface for the user to interact with the agent's environment.
- **`advisor`**: Strategic guidance for the coding task.
- **`agents`**: Management of active sub-agents.
- **`good-claude`**: Specialized behavior or evaluation mode.

---

## 4. Prompt Engineering Patterns

The source code reveals a highly modular approach to system prompts and memory:
- **Prompt Cache Efficiency**: Uses an **Attachment System** to move volatile context (MCP instructions, agent lists) out of the static system prompt to maximize cache reuse.
- **Persistent Memory Layers**: Implements **Session Memory** (live state) and **Memory Extraction** (long-term knowledge) using dedicated background subagents and structured Markdown files.
- **Fork vs. Fresh Agent**: Distinguishes between **Forks** (inherited context, directive prompts) and **Fresh Agents** (zero context, briefing prompts) for efficient task delegation.
- **XML-tagged Sections:** Prompts are constructed from multiple sections (mirrored in `systemPromptSections.ts`).
- **Context Bootstrapping:** The runtime builds a "context package" before each model turn.
- **Cyber Risk Instructions:** Explicit modules for handling security-sensitive operations (`cyberRiskInstruction.ts`).

---

## 5. Development Workflow (OmX)

The `claw-code` project itself was built using **oh-my-codex (OmX)**, demonstrating a multi-agent development workflow:
- **`$team` mode:** Coordinated parallel review and architectural feedback.
- **`$ralph` mode:** Persistent execution, verification, and completion discipline.
- **Codex-driven:** Layered on top of foundation models for architect-level verification.
