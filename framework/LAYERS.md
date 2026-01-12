# Layers

Layers are independent specification manifests that together form a coherent application. Each layer answers a specific question and receives requirements from its own prompt file. Upstream layers provide context for coherence and traceability, not requirements.

## Layer Independence

**Prompt-driven requirements:** Each layer's prompt file serves as the sole source of requirements for that layer. Layers can be selected or swapped independently while maintaining coherence with adjacent layers. Upstream layers don't derive requirements for downstream layers.

## Upstream References

**Purpose of references:** Layers reference upstream specifications for coherence (ensuring compatibility across layers) and traceability (enabling coverage to map requirements without gaps). Implementation agents consolidate specifications from multiple layers before execution, validating coherence at phase boundaries.

## Layer Order

**Sequential workflow within phases:** Layers are worked through in order within each phase. Phase 1 progresses Business through Stack. Phase 2 handles Infrastructure reading all Phase 1 specs for context. Phase 3 handles Coverage reading all specs.

**Context accumulation:** The order provides context accumulation rather than requirement derivation. Phase 1 layers each provide cumulative context for subsequent layers. Infrastructure uses all Phase 1 specs as coherence context. Coverage validates against all layers.

## Layer Definitions

### Business — Why?

**Purpose:** The Business layer captures intent, value, and goals justifying the work.

**Content:** Use cases, actors, measurable outcomes defining success.

**Input:** User requirements covering stakeholder goals, use cases, and success criteria.

**Context:** None (Business is the first layer).

**Scope:** Business concerns only—no technologies, implementation details, data structures, API contracts, deployment concerns, or infrastructure references. System actor captures stakeholder requirements about system properties (availability, auditability, accessibility) while remaining at business level without prescribing technical solutions.

---

### Functional — What?

**Purpose:** The Functional layer defines behaviors, contracts, and data models required to fulfill business goals.

**Content:** User flows, data models, API contracts, state transitions.

**Input:** User experience requirements covering experience shape, behaviors, and interactions.

**Context:** Business specifications for coherence and traceability.

**Scope:** Behavioral specifications only—no technology choices, deployment concerns, performance benchmarks, or implementation patterns. References Business specs using Implements (1:1 feature mapping) or Enables (1:many foundation mapping) with justification when foundation specs lack Business references.

**Foundation distinction:** Feature specs implement specific business use cases (1:1 mapping). Foundation specs enable multiple business use cases (1:many mapping), serving as legitimate engineering artifacts for shared components and cross-cutting concerns. Orphaned foundations (no Business references, no justification) indicate scope creep.

---

### Stack — With what?

**Purpose:** The Stack layer selects and justifies technologies used to implement functional requirements.

**Content:** Language choices with versions, framework selections, library specifications, tool rationale, build tooling, development environment definitions.

**Input:** User technology preferences covering languages, frameworks, constraints, and team expertise.

**Context:** Business and Functional specifications for coherence and traceability.

**Scope:** Technology selections only—no code examples, implementation patterns, architecture code blocks, deployment topology, compute/networking/scaling decisions, cloud providers, or hosting platforms. References Functional specs using Enables (foundation serving multiple) or direct reference (feature serving one) with justification when foundation specs lack Functional references. Consistency with Functional specs validated at implementation.

**Foundation distinction:** Feature specs cover technology choices for specific features (1:1 mapping). Foundation specs cover base technologies enabling multiple features (1:many mapping), serving as legitimate engineering artifacts like base language environments and shared dependencies. Orphaned foundations indicate scope creep.

---

### Infrastructure — Where?

**Purpose:** The Infrastructure layer defines where and how the application runs in production.

**Content:** Compute resources, networking topology, security boundaries, observability (logging, metrics, tracing), scaling policies, resource limits, secrets management approach.

**Input:** User deployment requirements covering environment, hosting, scaling, and constraints.

**Context:** Phase 1 specifications (Business, Functional, Stack) for coherence and traceability.

**Scope:** Operational concerns only—no business logic redefinitions, technology choice overrides, application code, configurations, or test cases. References Phase 1 specs using Enables (foundation serving multiple) or direct reference (feature serving one) with justification when foundation specs lack Phase 1 references. Consistency with Phase 1 specs validated at implementation.

**Foundation distinction:** Feature specs cover infrastructure for specific features/components (1:1 mapping). Foundation specs cover base infrastructure enabling multiple features (1:many mapping), serving as legitimate operational artifacts like base networking and shared observability. Orphaned foundations indicate scope creep.

---

### Coverage — What's verified?

**Purpose:** The Coverage layer ensures all requirements are testable and traceable.

**Content:** Acceptance criterion mappings, test case definitions, requirement-to-test-to-outcome maps, untestable requirement flags, integration/E2E/acceptance test definitions, specification coverage percentages.

**Input:** User test requirements covering test scope, test environment, integration points, and acceptance thresholds.

**Context:** All layer specifications (Business, Functional, Stack, Infrastructure) as source of upstream acceptance criteria to verify.

**Scope:** Verification definitions only—no new acceptance criteria beyond upstream specs, no skipped criteria without justification, no modified or reinterpreted upstream criteria, no unit tests (those are implementation details).

## Dependency Graph

Layers connect through references forming a dependency structure supporting traceability and coherence:

- Coverage reads all layers (Business, Functional, Stack, Infrastructure)
- Infrastructure reads Phase 1 layers (Business, Functional, Stack)  
- Stack reads Business and Functional
- Functional reads Business
- Business reads prompt file only

Phase 1 (Develop) progresses linearly: Business → Functional → Stack.

Phase 2 (Deploy) adds Infrastructure reading all Phase 1 specs cross-cutting.

Phase 3 (Validate) adds Coverage reading all specs cross-cutting.
