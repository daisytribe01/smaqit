# Layers

Layers are independent specification manifests that together form a coherent application. Each layer answers a specific question and receives requirements from its own prompt file. Upstream layers provide context for coherence and traceability, not requirements.

## Layer Independence

**Each layer's prompt file is the sole source of requirements for that layer.**

Each layer receives requirements from its prompt file, can be selected or swapped independently, maintains coherence with adjacent layers, and does not derive requirements from upstream layers.

## Upstream References

Layers reference upstream specifications for these purposes:

| Purpose | Description |
|---------|-------------|
| **Coherence** | Implementation agents consolidate specs from multiple layers before execution. References ensure specs are compatible. |
| **Traceability** | Coverage maps requirements through all layers to ensure nothing is missed. |

Coherence validation happens at the end of each phase, where the implementation agent consolidates the required specs before execution.

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

**Content Scope:**

Business specifications identify all actors and their goals, define measurable success metrics for each use case, include preconditions and postconditions, and describe main and alternative flows in business terms.

Business specifications remain technology-agnostic, excluding specific technologies, frameworks, libraries, implementation details, technical solutions, data structures, API contracts, deployment concerns, or infrastructure.

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

**Content Scope:**

Functional specifications define user flows that implement business use cases, specify data models with attributes and relationships, define API contracts (inputs, outputs, error conditions), and include state transitions where applicable. They reference business specs for traceability using Implements (1:1 feature) or Enables (1:many foundation), and include justification when foundation specs have no Business references.

Functional specifications remain technology-neutral, excluding technology choices (languages, frameworks, databases), deployment or infrastructure concerns, performance benchmarks, and implementation patterns.

**Foundation vs Feature Specs:**

Functional specs come in two categories:

| Type | Purpose | Business Reference |
|------|---------|--------------------|
| **Feature specs** | Implement a specific business use case | 1:1 mapping (Implements) |
| **Foundation specs** | Enable multiple business use cases | 1:many mapping (Enables) |

Foundation specs (shared components, cross-cutting concerns, common contracts) are legitimate engineering artifacts that serve multiple business goals.

Foundation specs reference all Business specs they enable, and flag absence of Business references with justification to surface potential scope creep.

**Note:** Orphaned foundations (no Business references, no justification) indicate scope creep.

---

### Stack — With what?

The Stack layer selects and justifies the technologies used to implement functional requirements.

**Purpose:** Choose languages, frameworks, libraries, and tools that can deliver the specified behaviors.

**Input:** User technology preferences (languages, frameworks, constraints, team expertise)

**Context:** Business and Functional specs (for coherence and traceability)

**Content Scope:**

Stack specifications document technology choices with rationale, define language and framework versions, specify libraries and their purposes, and include build tools and development environment setup. They maintain consistency with Functional specs (validated at implementation), reference Functional specs using Enables (foundation serving multiple) or direct reference (feature serving one), and include justification when foundation specs have no Functional references.

Stack specifications exclude implementation code, code examples, implementation patterns, architecture code blocks, deployment topology, infrastructure decisions, compute or networking or scaling decisions, cloud providers, and hosting platforms. Stack specifications never contradict functional requirements.

**Foundation vs Feature Specs:**

Stack specs come in two categories:

| Type | Purpose | Functional Reference |
|------|---------|--------------------|
| **Feature specs** | Technology choices for a specific feature | 1:1 mapping (Enables) |
| **Foundation specs** | Base technologies enabling multiple features | 1:many mapping (Enables) |

Foundation specs (base language environments, shared build tools, common dependencies) are legitimate engineering artifacts that serve multiple functional requirements.

Foundation specs reference all Functional specs they enable, and flag absence of Functional references with justification to surface potential scope creep.

**Note:** Orphaned foundations (no Functional references, no justification) indicate scope creep.

---

### Infrastructure — Where?

The Infrastructure layer defines where and how the application runs in production.

**Purpose:** Specify compute, networking, observability, and operational concerns.

**Input:** User deployment requirements (environment, hosting, scaling, constraints)

**Context:** Phase 1 specs (Business, Functional, Stack) for coherence and traceability

**Content Scope:**

Infrastructure specifications define compute resources (containers, serverless, VMs), specify networking topology and security boundaries, include observability (logging, metrics, tracing), define scaling policies and resource limits, and specify secrets management approach. They maintain consistency with Phase 1 specs regarding requirements and runtime constraints (validated at implementation), reference Phase 1 specs using Enables (foundation serving multiple) or direct reference (feature serving one), and include justification when foundation specs have no Phase 1 references.

Infrastructure specifications exclude business logic redefinitions, functional behavior overrides, technology choice overrides from Stack layer, application code or configurations, and test case definitions (those belong in Coverage).

**Foundation vs Feature Specs:**

Infrastructure specs come in two categories:

| Type | Purpose | Phase 1 Reference |
|------|---------|--------------------|
| **Feature specs** | Infrastructure for a specific feature/component | 1:1 mapping (Enables) |
| **Foundation specs** | Base infrastructure enabling multiple features | 1:many mapping (Enables) |

Foundation specs (base networking, shared security policies, common observability configuration) are legitimate operational artifacts that serve multiple application components.

Foundation specs reference all Phase 1 specs (Business, Functional, Stack) they enable, and flag absence of Phase 1 references with justification to surface potential scope creep.

**Note:** Orphaned foundations (no Phase 1 references, no justification) indicate scope creep.

---

### Coverage — What's verified?

The Coverage layer ensures all requirements are testable and traceable. It reads from all upstream layers for traceability and coherence.

**Purpose:** Enumerate every acceptance criterion and map it to a verification test.

**Input:** User test requirements (test scope, test environment, integration points, acceptance thresholds)

**Context:** All layer specs (Business, Functional, Stack, Infrastructure) — source of upstream acceptance criteria to verify

**Content Scope:**

Coverage specifications reference every acceptance criterion from upstream specs by ID, define a test case for each testable requirement, map requirements to test cases to expected outcomes, flag untestable requirements explicitly, include integration, E2E, and acceptance test definitions, and report spec coverage (percentage of requirements with corresponding tests).

Coverage specifications do not add acceptance criteria not present in upstream specs, do not skip upstream acceptance criteria without justification, do not modify or reinterpret upstream acceptance criteria, and exclude unit test definitions (those are implementation details).

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
