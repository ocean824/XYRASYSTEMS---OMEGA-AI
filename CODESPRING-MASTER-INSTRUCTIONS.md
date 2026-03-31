# 🧠 CODESPRING MASTER INSTRUCTIONS: ØMEGA AI (XYRA SYSTEMS)

> **Role**: Lead Systems Architect & Developer
> **Objective**: Implement the ØMEGA AI Orchestration Platform.
> **Status**: Hyper-Detailed Implementation Blueprint (Zero-Question Target)

---

## 1. Core System Architecture & DNA
ØMEGA AI is a **recursive, hierarchical multi-agent platform** built on a **Master Orchestrator (PRIME)** and a **25+ specialized "Agent Army"**.

### 1.1 Tech Stack (Mandatory)
- **Orchestration**: PydanticAI (Type-safe tool calls) + OpenAI Swarm (Hand-offs).
- **Reasoning Engine**: DeepSeek-V3 (Primary) / Claude 3.5 Sonnet (Coding).
- **Memory**: Dual-layer (Session State + Persistent Markdown Knowledge via Claude Code pattern).
- **Integration**: Model Context Protocol (MCP) for all external tool access.

---

## 2. PRIME: The Master Orchestrator (The "Brain")
PRIME is the "Supreme Commander" that governs all sub-agents.

### 2.1 The "PRIME" System Prompt Schema
```markdown
# IDENTITY: PRIME (Master Orchestrator)
- **Goal**: Execute complex business automation by orchestrating specialized agents.
- **Constraints**: 
    - Never execute a task directly if a sub-agent is available.
    - Always use PydanticAI to validate tool-call data.
    - Implement the <megaman_principle>: Absorb capabilities from any ingested tool/doc.
    - Implement the <jarvis_principle>: Proactively identify and advise on business opportunities.
```

---

## 3. THE AGENT ARMY: 25+ Specialized Sub-Agents

Each sub-agent must be implemented as a specialized PydanticAI agent with its own system prompt, toolset, and domain logic.

### 3.1 Trading & Finance Cluster
1.  **Trading Agent (ORACLE AI)**: Market analysis, signal processing, and high-fidelity execution via the 4-layer Oracle stack.
2.  **Risk Manager**: Portfolio limits, drawdown controls, and regime detection.
3.  **Signal Analyst**: Validates external signals using Layer 1 (Data) + Layer 3 (Indicators).
4.  **Order Executor**: Final "GO/NO-GO" signal gate for execution.

### 3.2 Marketing & Creative Cluster
5.  **Marketing Agent**: Campaign management, audience targeting, and budget optimization.
6.  **Sonus (Audio Agent)**: Immersive audiobooks, emotion analysis, and granular voice control.
7.  **Visage (Visual Agent)**: High-fidelity video generation (Open-Sora, Wan 2.x, Mochi, SVD) and virtual humans.
8.  **Media Agent**: Music generation (YuE, DiffRhythm, ACE-Step).
9.  **Content Generator**: Automated ad copy and creative generation.
10. **MAESTRO (Director)**: Technical scripting and routing for the Visage video pipeline.

### 3.3 E-commerce & Operations Cluster
11. **E-commerce Agent**: Product management, inventory, and fulfillment tracking.
12. **Operations Agent**: Task management, scheduling, and team coordination.
13. **Product Manager**: Creation, updates, and dynamic pricing logic.
14. **Customer Service Agent**: Automated support, refunds, and returns management.
15. **Logistics Agent**: Fulfillment tracking and warehouse optimization.

### 3.4 Research & Intelligence Cluster
16. **Research Agent (MiroFish)**: Data collection, analysis, and swarm intelligence prediction.
17. **Simulation Agent (MiroFish)**: Parallel world simulation and GraphRAG construction.
18. **Persona Agent**: High-fidelity simulation agents for MiroFish swarm rehearsals.
19. **Report Agent**: Generates detailed simulation and research reports.
20. **Trend Analyzer**: Captures collective emergence from agent interactions.

### 3.5 Automation & Development Cluster
21. **Desktop Agent**: OS-level control (mouse/keyboard) via visual understanding.
22. **Browser Agent**: Web automation, form filling, and data extraction.
23. **Coding Agent**: Code generation, debugging, and refactoring (Claude Code pattern).
24. **Warden (Security)**: Validation gate for all autonomous actions.
25. **Audit Agent**: Maintains the immutable audit trail for all system decisions.

---

## 4. Agent Logic & Workflows

### 4.1 Visage (Visual Agent) & MAESTRO
**The 5-Stage Video Generation Pipeline:**
1. **MAESTRO (Director)**: Generates a technical script in JSON format.
2. **Open-Sora (Base)**: Core video generation from script.
3. **Wan 2.x (Fast Scaling)**: Rapid iterations for style/text-in-video.
4. **Mochi-1 (Physics)**: 10B parameter model for realistic motion and physics.
5. **SVD (Polish)**: Final cinematic transitions and upscaling.

### 4.2 MiroFish (Simulation Agent)
**The "Parallel World" Simulation Logic:**
- **Step 1**: Extract seed info from news/reports.
- **Step 2**: Spawn 100+ "Persona Agents" with diverse biases.
- **Step 3**: Rehearse future trajectories in a digital sandbox.
- **Step 4**: Aggregate results into a **Trajectory Report** with confidence intervals.

---

## 5. Advanced Engineering Patterns (Derived from Claude Code & Manus)

### 5.1 Memory Architecture (The "Claude Code" Pattern)
- **Static Boundary**: Keep the 20K+ token system prompt static for caching.
- **Dynamic Context**: Inject `agent_listing_delta` and `session_state` via attachments.
- **Persistence**: Use a background sub-agent (Fork) to distill session knowledge into `MEMORY.md` files.

### 5.2 Tool-Level Granularity (Pydantic Models)
All tools must be implemented as Pydantic models for zero-error data transfer.
```python
class AgentHandover(BaseModel):
    target_agent: str = Field(description="The symbolic name of the sub-agent to invoke.")
    context: dict = Field(description="The serialized state and mission parameters.")
    priority: int = Field(default=1, ge=1, le=5)
```

---

## 6. Security & Risk Governance
- **Rule 1**: No tool execution without `WARDEN` approval.
- **Rule 2**: Max daily credit/token limit (Hard-coded).
- **Rule 3**: All API keys must be isolated in a secure vault/sandbox.

---

## 7. Implementation Roadmap for CodeSpring
1. **Phase 1**: Initialize the PydanticAI routing core and the `PRIME` system prompt.
2. **Phase 2**: Implement the **Claude Code** memory pattern (Static/Dynamic Boundary).
3. **Phase 3**: Connect the **Visage Video Pipeline** and **MiroFish Simulation Core**.
4. **Phase 4**: Integrate the **ORACLE AI** (Standalone) via the dedicated status protocol.

---
**XYRA SYSTEMS — ØMEGA AI — THE ULTIMATE AGENT.**
