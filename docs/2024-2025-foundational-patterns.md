# 2024-2025 Foundational Agentic Patterns

> **Status:** Core Methodology — ØMEGA AI Integration
> **Context:** "Classic" breakthroughs that define modern agentic behavior and orchestration.

---

## 1. Multi-Agent Orchestration Patterns

The 2024-2025 period established the "Big Three" patterns for agent coordination.

### 1.1 CrewAI: Role-Playing & Collaborative Intelligence
- **Core Pattern**: **Role-based Task Assignment**. Every agent has a specific `role`, `goal`, and `backstory`.
- **Omega Integration**: Enhance the **Marketing** and **Media** agents by giving them explicit "Backstories" (e.g., "You are a veteran copywriter with a focus on high-conversion hooks") to anchor their persona and improve output quality.
- **Key Feature**: **Hierarchical vs. Sequential Workflows**. Omega's PRIME orchestrator uses the Hierarchical pattern, while sub-agents (like Sonus/Visage) use Sequential hand-offs.

### 1.2 AutoGen (Microsoft): Conversational Multi-Agent Systems
- **Core Pattern**: **Multi-Agent Conversation**. Agents "talk" to each other to solve a problem.
- **Omega Integration**: Use for **Research (MiroFish)** where a "Critic" agent and a "Researcher" agent can debate findings before finalizing a report.
- **Key Feature**: **Human-in-the-Loop (HITL)**. Standardized patterns for when an agent should "pause" and ask for human approval (mirrored in `AskUserQuestionTool`).

### 1.3 LangGraph: Cyclic Workflows & Persistence
- **Core Pattern**: **Stateful Cyclic Graphs**. Agents can loop back to previous steps if a condition isn't met.
- **Omega Integration**: The **Coding Agent** uses this for "Test-Fix-Repeat" cycles. If a test fails, the agent loops back to the coding step with the error log.
- **Key Feature**: **Checkpoints & Time-Travel**. Ability to save the exact state of a multi-agent run and "rewind" to a specific node if a mistake is made.

---

## 2. Programmatic Prompting (DSPy)

### 2.1 The "Programming, Not Prompting" Shift
- **Core Pattern**: **Typed Signatures**. Instead of writing long strings, you define a signature: `Question -> Answer`.
- **Omega Integration**: Use for the **Operations Agent** to standardize internal data transformations.
- **Key Feature**: **Self-Improving Pipelines**. DSPy "compiles" prompts by testing them against examples and automatically optimizing the instructions. Omega can use this to "auto-tune" the system prompts for sub-agents based on successful task completions.

---

## 3. Computer Use & Visual Intelligence

### 3.1 Anthropic "Computer Use" API (Oct 2024)
- **Core Pattern**: **Visual Action Tokens**. The agent takes a screenshot, identifies coordinates, and "clicks" or "types."
- **Omega Integration**: This is the foundation for the **Desktop Agent (Vy)** and **Browser Agent (Runner H)**.
- **Key Feature**: **Feedback Loops**. The agent checks the screenshot *after* an action to verify the result before moving to the next step.

---

## 4. Coding Agent Breakthroughs (SWE-bench Era)

### 4.1 SWE-agent & OpenDevin (2024)
- **Core Pattern**: **Agentic Tool-Use for Repo-Level Fixes**. Moving beyond single-file edits to full repository context.
- **Omega Integration**: The **Coding Agent** adopts the "Plan-Execute-Verify" loop from Devin/SWE-agent.
- **Key Feature**: **Post-Mortem Analysis**. If a task fails, the agent generates a "Lessons Learned" entry in the persistent memory to avoid the same mistake in the future.

---

## 5. Summary of "Classic" Gems for Omega

| Pattern | Source | Omega Application |
|---------|--------|-------------------|
| **Role Backstories** | CrewAI | Improves persona consistency in Creative agents. |
| **Multi-Agent Debate** | AutoGen | Increases accuracy in Research and Trading agents. |
| **Typed Signatures** | DSPy | Standardizes communication between PRIME and sub-agents. |
| **Visual Verification** | Anthropic | Ensures reliability in Desktop and Browser automation. |
| **Post-Mortem Memory** | SWE-agent | Powers the "Learnings" section in persistent session memory. |
