# Layers

Layers are independent specification manifests that together form a coherent application. Each layer answers a specific question and receives requirements from its own prompt file. Upstream layers provide context for coherence and traceability, not requirements.

## Layer Independence

Each layer’s prompt is the sole source of its requirements. Layers can be selected or swapped independently while remaining coherent with adjacent layers, and do not derive requirements from upstream documents.

## Upstream References

Layers reference upstream specifications for these purposes:

| Purpose | Description |
|---------|-------------|
| **Coherence** | Implementation agents consolidate specs from multiple layers before execution. References ensure specs are compatible. |
| **Traceability** | Coverage maps requirements through all layers to ensure nothing is missed. |

Coherence validation happens at the end of each phase, when implementation agents consolidate the required specifications before execution.

## Layer Order

Layers are worked through in order within each phase:

**Phase 1 (Develop):** Business → Functional → Stack

**Phase 2 (Deploy):** Infrastructure (reads all Phase 1 specs for context)

**Phase 3 (Validate):** Coverage (reads all specs)

The order provides context accumulation, not requirement derivation:
- **Phase 1 layers** (Business through Stack): each provides cumulative context for subsequent layers
- **Infrastructure** (Phase 2): uses all Phase 1 specs as coherence context
- **Coverage** (Phase 3): validates against all layers

## Layer Definitions

### Business — Why?

The Business layer captures the intent, value, and goals of what is being built.

**Purpose:** Define use cases, actors, and measurable outcomes that justify the work.

**Input:** User requirements (stakeholder goals, use cases, success criteria)

**Context:** None (Business is the first layer)

Business specifications focus on actors, goals, measurable outcomes, and flow descriptions expressed in business language. They avoid technology choices, implementation solutions, data structures, and deployment concerns.

**System Actor:**

When stakeholders have requirements about system properties (availability, auditability, accessibility), use the **System** actor:

| Actor | Description | Goals |
|-------|-------------|-------|
| System | The application as a whole | [System-level properties stakeholders require] |

System actor specs remain business-level (stakeholder-driven) and do not prescribe technical solutions.

---

### Functional — What?

The Functional layer defines the behaviors, contracts, and data models required to fulfill business goals.

**Purpose:** Translate user experience requirements into precise behavioral specifications.

**Input:** User experience requirements (experience shape, behaviors, interactions)

**Context:** Business specs (for coherence and traceability)

Functional specifications translate experience requirements into behaviors, data models, contracts, and state transitions. They reference business context for traceability while remaining free of technology choices, deployment concerns, performance benchmarks, and implementation patterns.

**Foundation vs Feature Specs:**

Functional specs come in two categories:

| Type | Purpose | Business Reference |
|------|---------|--------------------|
| **Feature specs** | Implement a specific business use case | 1:1 mapping (Implements) |
| **Foundation specs** | Enable multiple business use cases | 1:many mapping (Enables) |

Foundation specs (shared components, cross-cutting concerns, common contracts) are legitimate engineering artifacts that serve multiple business goals.

**Foundation spec rules:**
Foundation specifications explain which business needs they enable and clarify when no business reference exists to avoid orphaned scope.

**Note:** Orphaned foundations (no Business references, no justification) indicate scope creep.

---

### Stack — With what?

The Stack layer selects and justifies the technologies used to implement functional requirements.

**Purpose:** Choose languages, frameworks, libraries, and tools that can deliver the specified behaviors.

**Input:** User technology preferences (languages, frameworks, constraints, team expertise)

**Context:** Business and Functional specs (for coherence and traceability)

Stack specifications capture technology selections, versions, libraries, build tools, and environment setup with rationale that aligns to functional intent. They avoid code examples, infrastructure topology, scaling or provider details, and anything that would contradict functional requirements.

**Foundation vs Feature Specs:**

Stack specs come in two categories:

| Type | Purpose | Functional Reference |
|------|---------|--------------------|
| **Feature specs** | Technology choices for a specific feature | 1:1 mapping (Enables) |
| **Foundation specs** | Base technologies enabling multiple features | 1:many mapping (Enables) |

Foundation specs (base language environments, shared build tools, common dependencies) are legitimate engineering artifacts that serve multiple functional requirements.

**Foundation spec rules:**
Foundation stack specifications clarify which functional needs they enable and call out any missing mappings to prevent orphaned tooling.

**Note:** Orphaned foundations (no Functional references, no justification) indicate scope creep.

---

### Infrastructure — Where?

The Infrastructure layer defines where and how the application runs in production.

**Purpose:** Specify compute, networking, observability, and operational concerns.

**Input:** User deployment requirements (environment, hosting, scaling, constraints)

**Context:** Phase 1 specs (Business, Functional, Stack) for coherence and traceability

Infrastructure specifications describe compute, networking, security boundaries, observability, scaling, and secrets approach in harmony with Phase 1 intent. They stay clear of business logic, functional behavior changes, application code, and test definitions.

**Foundation vs Feature Specs:**

Infrastructure specs come in two categories:

| Type | Purpose | Phase 1 Reference |
|------|---------|--------------------|
| **Feature specs** | Infrastructure for a specific feature/component | 1:1 mapping (Enables) |
| **Foundation specs** | Base infrastructure enabling multiple features | 1:many mapping (Enables) |

Foundation specs (base networking, shared security policies, common observability configuration) are legitimate operational artifacts that serve multiple application components.

**Foundation spec rules:**
Foundation infrastructure specifications explain how they support Phase 1 intent and surface any missing mappings to avoid unexplained foundations.

**Note:** Orphaned foundations (no Phase 1 references, no justification) indicate scope creep.

---

### Coverage — What's verified?

The Coverage layer ensures all requirements are testable and traceable. It reads from all upstream layers for traceability and coherence.

**Purpose:** Enumerate every acceptance criterion and map it to a verification test.

**Input:** User test requirements (test scope, test environment, integration points, acceptance thresholds)

**Context:** All layer specs (Business, Functional, Stack, Infrastructure) — source of upstream acceptance criteria to verify

Coverage specifications trace acceptance criteria across all upstream layers and translate them into verification approaches and expected outcomes. They highlight untestable items, cover integration and end-to-end scenarios, and focus on mapping rather than redefining requirements or introducing new criteria.

## Dependency Graph

```
                    ┌─────────────────────────────────────┐
                    │              Coverage               │
                    │          (What's verified?)         │
                    └─────────────────────────────────────┘
                        ↑         ↑         ↑         ↑
          ┌─────────────┘         │         │         └─────────────┐
          │                       │         │                       │
          │                       │         │                       │
    ┌─────┴─────┐           ┌─────┴─────┐   │                 ┌─────┴─────┐
    │  Business │           │ Functional│   │                 │  Infra    │
    │   (Why?)  │──────────→│  (What?)  │───┼─────────╮       │ (Where?)  │
    └─────┬─────┘           └─────┬─────┘   │         │       └───────────┘
          │                       │         │         │             ↑
          │                       │         │         │             │
          │                 ┌─────┴─────┐   │         │             │
          │                 │   Stack   │───┘         │             │
          │                 │(With what)│─────────────┼─────────────┘
          │                 └───────────┘             │
          │                                           │
          └───────────────────────────────────────────┘
```

**Phase 1 (Develop):** Business → Functional → Stack (linear)

**Phase 2 (Deploy):** Infrastructure reads all Phase 1 specs (cross-cutting)

**Phase 3 (Validate):** Coverage reads all specs including Infrastructure (cross-cutting)
