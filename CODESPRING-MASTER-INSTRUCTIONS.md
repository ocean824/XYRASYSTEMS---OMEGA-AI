# CODESPRING MASTER INSTRUCTIONS: ØMEGA AI (XYRA Systems)

> **Role**: Lead Systems Architect & Developer
> **Objective**: Implement the ØMEGA AI Orchestration Platform.
> **Status**: Implementation-Ready (Zero-Question Blueprint)

---

## 1. Core System Architecture
ØMEGA AI is a **hierarchical multi-agent platform** built on a **Master Orchestrator (PRIME)** and **25 specialized sub-agents**.

### 1.1 Tech Stack (Mandatory)
- **Orchestration**: PydanticAI (Type-safe tool calls) + OpenAI Swarm (Hand-offs).
- **Reasoning Engine**: DeepSeek-V3 (Primary) / Claude 3.5 Sonnet (Coding).
- **Memory**: Dual-layer (Session State + Persistent Markdown Knowledge).
- **Integration**: Model Context Protocol (MCP).

---

## 2. Agent Logic & Workflows

### 2.1 PRIME (Master Orchestrator)
- **Logic**: Ingest user intent -> Route to sub-agent via PydanticAI validator -> Execute -> Consolidate results.
- **Behavioral Modules**:
    - `<megaman_principle>`: Adapt to any domain by ingesting relevant documentation.
    - `<jarvis_principle>`: Proactively identify business opportunities and provide advice.

### 2.2 Visage (Visual Agent) & MAESTRO
- **Workflow**: MAESTRO Scripting -> Open-Sora (Base) -> Wan 2.x (Scaling) -> Mochi (Physics) -> SVD (Polish).
- **Implementation**: Build a Python-based routing layer that manages model weights and FFmpeg post-processing.

### 2.3 MiroFish (Simulation Agent)
- **Logic**: Parallel World Simulation. Spawn 100+ "Persona Agents" to rehearse market/business trajectories.
- **Output**: Predictive trajectory reports with confidence intervals.

---

## 3. Tool & API Schema
All tools must be defined as **Pydantic models** to ensure zero-error data transfer between PRIME and sub-agents.

```python
from pydantic import BaseModel, Field

class AgentHandover(BaseModel):
    target_agent: str = Field(description="The symbolic name of the sub-agent to invoke.")
    context: dict = Field(description="The serialized state and mission parameters.")
    priority: int = Field(default=1, ge=1, le=5)
```

---

## 4. Implementation Roadmap
1. **Phase 1**: Initialize the PydanticAI routing core.
2. **Phase 2**: Implement the **Claude Code** memory pattern (Static/Dynamic Boundary).
3. **Phase 3**: Connect the **Visage Video Pipeline** as the first specialized creative module.
4. **Phase 4**: Integrate the **ORACLE AI** (Standalone) via the dedicated status protocol.

---

## 5. Reference Documentation
- **Architecture**: `docs/architecture-overview.md`
- **Sub-Agents**: `docs/sub-agent-modules.md`
- **Video Pipeline**: `docs/visage-video-pipeline.md`
- **2026 Stack**: `docs/2026-agent-intelligence-stack.md`

---
**XYRA SYSTEMS — ØMEGA AI — THE ULTIMATE AGENT.**
