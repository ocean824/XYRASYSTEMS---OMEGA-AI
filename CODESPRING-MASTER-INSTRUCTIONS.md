# 🧠 CODESPRING MASTER INSTRUCTIONS: ØMEGA AI (XYRA SYSTEMS)

> **Role**: Lead Systems Architect & Developer
> **Objective**: Implement the ØMEGA AI Orchestration Platform.
> **Status**: Hyper-Detailed Implementation Blueprint (Zero-Question Target)

---

## 1. Core System Architecture & DNA
ØMEGA AI is a **recursive, hierarchical multi-agent platform** built on a **Master Orchestrator (PRIME)** and **25+ specialized sub-agents**.

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

### 2.2 Orchestration Logic (Recursive)
1. **Ingestion**: Use `agent_listing_delta` to keep the prompt cacheable.
2. **Routing**: Use `AgentTool` to spawn or fork sub-agents (e.g., `research_agent`, `trading_agent`).
3. **Validation**: Every output must pass through a `WARDEN` validation gate before being presented to the user.

---

## 3. Specialized Sub-Agent Blueprints

### 3.1 Visage (Visual Agent) & MAESTRO
**The 5-Stage Video Generation Pipeline:**
1. **MAESTRO (Director)**: Generates a technical script in JSON format.
2. **Open-Sora (Base)**: Core video generation from script.
3. **Wan 2.x (Fast Scaling)**: Rapid iterations for style/text-in-video.
4. **Mochi-1 (Physics)**: 10B parameter model for realistic motion and physics.
5. **SVD (Polish)**: Final cinematic transitions and upscaling.

**MAESTRO Script Schema:**
```json
{
  "scene_id": 1,
  "prompt": "...",
  "motion_bucket_id": 127,
  "fps": 24,
  "duration": 5.0,
  "physics_constraints": ["gravity: 9.8", "fluid_dynamics: active"]
}
```

### 3.2 MiroFish (Simulation Agent)
**The "Parallel World" Simulation Logic:**
- **Step 1**: Extract seed info from news/reports.
- **Step 2**: Spawn 100+ "Persona Agents" with diverse biases.
- **Step 3**: Rehearse future trajectories in a digital sandbox.
- **Step 4**: Aggregate results into a **Trajectory Report** with confidence intervals.

---

## 4. Advanced Engineering Patterns (Derived from Claude Code & Manus)

### 4.1 Memory Architecture (The "Claude Code" Pattern)
- **Static Boundary**: Keep the 20K+ token system prompt static for caching.
- **Dynamic Context**: Inject `agent_listing_delta` and `session_state` via attachments.
- **Persistence**: Use a background sub-agent (Fork) to distill session knowledge into `MEMORY.md` files.

### 4.2 Tool-Level Granularity (Pydantic Models)
All tools must be implemented as Pydantic models for zero-error data transfer.
```python
class AgentHandover(BaseModel):
    target_agent: str = Field(description="The symbolic name of the sub-agent to invoke.")
    context: dict = Field(description="The serialized state and mission parameters.")
    priority: int = Field(default=1, ge=1, le=5)
```

---

## 5. Security & Risk Governance
- **Rule 1**: No tool execution without `WARDEN` approval.
- **Rule 2**: Max daily credit/token limit (Hard-coded).
- **Rule 3**: All API keys must be isolated in a secure vault/sandbox.

---

## 6. Implementation Roadmap for CodeSpring
1. **Phase 1**: Initialize the PydanticAI routing core and the `PRIME` system prompt.
2. **Phase 2**: Implement the **Claude Code** memory pattern (Static/Dynamic Boundary).
3. **Phase 3**: Connect the **Visage Video Pipeline** and **MiroFish Simulation Core**.
4. **Phase 4**: Integrate the **ORACLE AI** (Standalone) via the dedicated status protocol.

---
**XYRA SYSTEMS — ØMEGA AI — THE ULTIMATE AGENT.**
