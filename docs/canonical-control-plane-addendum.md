# ØMEGA AI Canonical Control-Plane Addendum

**Status:** Additive clarification note  
**Scope:** Non-destructive naming and control-plane alignment for future implementation  
**Applies To:** All net-new ØMEGA AI documentation, code, prompts, schemas, and integration work

## Purpose

This document is an **additive clarification** to the existing ØMEGA AI architecture. It does **not** delete, replace, or invalidate any prior document. Its purpose is to standardize the naming of the top-level control entity for future implementation while preserving the intent of the current repository materials.[1] [2] [3] [4]

The canonical rule established here is simple: **ØMEGA AI** is the name of the overall multi-agent system, while **PRIME** is the canonical name of the top-level governing agent that commands, routes, approves, and supervises the subordinate agents. In prior documents where **DOMINION** is used to describe the same top-level orchestrator role, future implementation should interpret that reference as **PRIME**, unless a later document explicitly defines DOMINION as a distinct subsystem.[1] [2] [3]

## Canonical Naming Standard

| Layer | Canonical Name | Meaning | Implementation Guidance |
|---|---|---|---|
| System | **ØMEGA AI** | The full operating system containing all agents, memory, governance, integrations, and specialized engines | Use this as the umbrella platform name in product, architecture, and deployment materials |
| Top-level controller | **PRIME** | The supreme command agent responsible for orchestration, policy enforcement, prioritization, and cross-agent coordination | Use this in all new code, schemas, prompts, APIs, and diagrams |
| Legacy alias | **DOMINION** | Historical or alternate naming used in existing documents for the same top-level controller concept | Preserve in historical docs, but map to PRIME in all new implementation work |
| Trading subsystem peer | **LEVIATHAN AI / QUANTUM** | The specialized trading organism integrated into ØMEGA AI | Keep distinct from PRIME; PRIME governs orchestration, while LEVIATHAN governs market intelligence through ORCA AI and MEGALODON AI within its own trading stack |

## Interpretation Rule for Existing Documents

Many current documents use language such as “DOMINION,” “master orchestrator,” “supreme orchestrator,” or “main brain” for the top command role.[1] [2] [4] This addendum establishes the following forward-looking interpretation rule.

> If an existing ØMEGA AI document refers to the **single top-level agent** that governs the other agents, coordinates workflows, enforces high-level autonomy rules, and owns the main control loop, that role should now be implemented under the canonical name **PRIME**.

This rule is intentionally narrow. It avoids rewriting historical material while giving the implementation layer a single stable identifier.

## Implementation Mapping

| Existing wording found in repository | Forward implementation meaning |
|---|---|
| DOMINION | PRIME |
| Supreme Orchestrator | PRIME |
| Main Brain | PRIME |
| Master Orchestrator | PRIME |
| Ø (when used as the singular command agent) | PRIME |

## Authority Boundary

The repository already distinguishes between the overall platform and the command role, even when naming varies.[1] [2] [3] This addendum makes that separation explicit for implementation.

| Concern | Owned by PRIME | Owned by ØMEGA AI |
|---|---|---|
| Global task routing | Yes | Indirectly, through PRIME |
| Agent approval policy | Yes | As a system capability |
| System-wide memory access coordination | Yes | Through LOOM/ARCHIVE as subsystems |
| Kill-switch authority | Yes, in coordination with WARDEN | As a platform-level safeguard |
| Branding / system identity | No | Yes |
| Full ecosystem of agents and services | No | Yes |

In other words, **PRIME is not the whole system**. PRIME is the command intelligence inside the ØMEGA AI system.

## Additive Guidance for Future Files

All **net-new** implementation artifacts should use the following conventions.

| Artifact Type | Recommended Convention |
|---|---|
| Root orchestrator class | `PrimeOrchestrator` |
| Event source name | `prime` |
| Main policy namespace | `prime.policy.*` |
| Queue routing key | `prime.control` |
| Audit actor label | `PRIME` |
| Diagram label | `PRIME — Top-Level Controller` |
| Legacy compatibility note | `DOMINION (legacy naming in documentation)` |

## Practical Transition Policy

This repository should transition by **addition, not replacement**. Existing documents can remain exactly as they are. New work should simply normalize around PRIME and, where helpful, include a short compatibility note.

A safe example is the following:

> **PRIME** is the canonical top-level controller for ØMEGA AI. Some historical documents refer to this same role as **DOMINION**.

That single sentence is enough to preserve continuity without forcing destructive edits.

## First Functional Layer: Canonical Control-Plane Example

The first real working layer for ØMEGA AI is not all 25 agents at once. It is a **small control plane** in which PRIME can accept a task, validate the target agent, and emit a canonical audit event. That is the minimum layer that makes the naming standard operational rather than merely descriptive.[2] [3] [4]

| Function | Purpose | Why It Belongs in Layer 1 |
|---|---|---|
| `normalize_controller_name()` | Maps legacy orchestrator names to PRIME | Makes historical naming safe at runtime |
| `build_agent_registry_record()` | Produces canonical registry entries | Prevents agent identity drift |
| `build_task_envelope()` | Standardizes task routing payloads | Creates a stable handoff object |
| `emit_audit_event()` | Records PRIME actions consistently | Makes control-plane actions inspectable |

### Example: Name Normalization Function

```python
from typing import Literal

CanonicalController = Literal["PRIME"]


def normalize_controller_name(name: str) -> CanonicalController:
    normalized = name.strip().upper()
    aliases = {"PRIME", "DOMINION", "SUPREME ORCHESTRATOR", "MASTER ORCHESTRATOR"}
    if normalized not in aliases:
        raise ValueError(f"Unknown top-level controller alias: {name}")
    return "PRIME"
```

This is intentionally simple. It encodes the exact governance decision established by this addendum.

### Example: Agent Registry Record

```python
from pydantic import BaseModel, Field
from typing import Literal


class AgentRegistryRecord(BaseModel):
    agent_id: str = Field(..., examples=["prime"])
    canonical_name: str = Field(..., examples=["PRIME"])
    role: str = Field(..., examples=["Top-level controller"])
    tier: Literal["tier_0", "tier_1", "tier_2", "tier_3"]
    status: Literal["active", "disabled", "draft"] = "active"



def build_agent_registry_record() -> AgentRegistryRecord:
    return AgentRegistryRecord(
        agent_id="prime",
        canonical_name=normalize_controller_name("DOMINION"),
        role="Top-level controller",
        tier="tier_0",
    )
```

### Example: Task Envelope

```python
from pydantic import BaseModel
from typing import Literal
from uuid import uuid4


class TaskEnvelope(BaseModel):
    task_id: str
    originator: str
    target_agent: str
    priority: Literal["low", "normal", "high", "critical"]
    goal: str
    approval_mode: Literal["autonomous", "confirmation_required", "blocked"]



def build_task_envelope(goal: str, target_agent: str) -> TaskEnvelope:
    return TaskEnvelope(
        task_id=f"task_{uuid4().hex[:10]}",
        originator="PRIME",
        target_agent=target_agent,
        priority="high",
        goal=goal,
        approval_mode="confirmation_required",
    )
```

### Example: Audit Event

```python
from datetime import datetime, UTC
from pydantic import BaseModel


class AuditEvent(BaseModel):
    event_id: str
    actor: str
    action: str
    resource: str
    policy_result: str
    timestamp: str



def emit_audit_event(action: str, resource: str, policy_result: str) -> AuditEvent:
    return AuditEvent(
        event_id=f"evt_{uuid4().hex[:10]}",
        actor="PRIME",
        action=action,
        resource=resource,
        policy_result=policy_result,
        timestamp=datetime.now(UTC).isoformat(),
    )
```

### Example: Putting the First Layer Together

```python
def bootstrap_prime_control_plane(goal: str, target_agent: str) -> tuple[AgentRegistryRecord, TaskEnvelope, AuditEvent]:
    prime_record = build_agent_registry_record()
    task = build_task_envelope(goal=goal, target_agent=target_agent)
    event = emit_audit_event(
        action="route_task",
        resource=task.task_id,
        policy_result="pending_review",
    )
    return prime_record, task, event
```

This is the smallest real layer that proves the naming clarification is executable: **PRIME exists, PRIME routes, and PRIME leaves an auditable trail**.

## Recommended Next Additions

This naming clarification becomes most useful when paired with implementation-facing control-plane artifacts. The highest-value next additions are a canonical agent registry, a task-envelope schema, an audit-event schema, and a control-plane sequence diagram that all use **PRIME** consistently.[2] [3] [4]

## References

[1]: ../README.md "ØMEGA AI README"
[2]: ../CODESPRING-MASTER-INSTRUCTIONS.md "ØMEGA AI CodeSpring Master Instructions"
[3]: ./agent-structure.md "ØMEGA AI Agent Structure"
[4]: ../CODESPRING-INTEGRATION-MANIFEST.md "ØMEGA AI CodeSpring Integration Manifest"
