# Artifacts

Artifacts are the outputs produced by agents. This document defines the principles governing artifact structure and relationships.

There are two types of artifacts:
- **Specification artifacts** — Declarative documents stating what must be true
- **Implementation artifacts** — Imperative outputs stating how to make it true

---

## Specification Artifacts

Specifications serve as the source of truth in Spec Driven Development, functioning as contracts between layers.

**Completeness principle:** A specification provides sufficient information for another agent or human to implement or validate against it without requiring additional context.

### Requirement Identifiers

**Unique traceability:** Every acceptance criterion carries a unique identifier enabling traceability across layers and phases.

**Identifier structure:** Identifiers combine layer prefix, concept name, and sequential number. Layer prefixes use three-letter codes, concept names describe the domain area, sequential numbers ensure uniqueness within the concept.

**Identifier stability:** Once assigned, identifiers remain stable. Deprecated requirements retain their identifiers with deprecation markers rather than deletion or reassignment.

### Acceptance Criteria

Acceptance criteria define testable conditions that implementations satisfy.

**Testability properties:** Criteria exhibit measurability (quantifiable outcomes), observability (external verification), and unambiguity (single interpretation).

**Untestable criteria:** Some requirements resist automated validation (subjective qualities, aesthetic judgments). These receive explicit flags, alternative proposals, and resolution strategies rather than blocking specification completion.

### Traceability

**Explicit source attribution:** Specifications reference their sources explicitly, enabling provenance verification and impact analysis.

**Reference types:** Prompt files serve as primary sources for layer requirements. Adjacent layer specifications provide coherence context. The reference chain supports impact analysis and coverage mapping while preserving layer independence.

**Cross-layer coordination:** Layer independence doesn't mean isolation. References create explicit chains enabling impact analysis (when upstream specs change, affected downstream specs are identified) and coverage mapping (all requirements trace through layers to verification tests).

**Foundation patterns:** Specifications come in feature and foundation varieties. Feature specs map to specific upstream requirements. Foundation specs enable multiple upstream requirements, serving shared needs across features. Foundation specs without upstream mappings include justifications explaining their necessity.

**Orphan detection:** Specifications lacking upstream references and justifications indicate potential scope creep, surfaced by coverage analysis.

### Coverage Translation

The Coverage layer translates acceptance criteria into executable test definitions, mapping requirements to verification methods.

### File Organization

**One specification per concept:** Specifications focus on single concepts rather than aggregating multiple unrelated concerns. Naming follows lowercase-with-hyphens conventions matching primary concept names. Generic names like "misc" or "other" indicate organizational problems.

### Specification Completeness

**Complete specifications:** All template sections filled without placeholders, acceptance criteria carrying unique identifiers, criteria testable or flagged, upstream references valid, scope boundaries explicit, implementation details absent (except Stack layer).

### Specification State

**Lifecycle tracking:** Specifications carry state through phases via frontmatter metadata. States include draft (newly generated), implemented (code complete), deployed (running in environment), validated (criteria verified), failed (processing error), and deprecated (feature removed).

**State transitions:** Specification agents create drafts. Implementation agents advance states based on phase outcomes. Manual intervention handles deprecation. The system tracks completion timestamps per state.

**Acceptance criteria state:** Implementation agents mark criteria satisfaction using checkbox states indicating not yet addressed, satisfied, or failed/untestable status.

**State aggregation:** Phase status aggregates from scanning all specification frontmatter, calculating counts per state without centralized state files.

**Staleness:** Specifications become stale when content changes after implementation. Detection responsibility lies with users rather than automated systems.

---

## Implementation Artifacts

Implementations are imperative outputs produced by implementation agents, satisfying specification-defined behavior while following industry standards.

### The Anchoring Principle

**Standards plus specs:** Implementations comply with industry standards for their technology stack while satisfying specification-defined behavior. Two compliant implementations may differ internally yet remain structurally recognizable and behaviorally equivalent.

### The Isolation Principle

**Reference-only credentials:** Agents operate on credential references, never values. Secrets remain outside agent context at all times. Resolution happens in trusted execution layers that return outcomes without exposing sensitive data.

### Three Dimensions

**Behavior dimension (invariant):** Defined by specifications, satisfied exactly without deviation. Behavior serves as the contract between specification and implementation.

**Structure dimension (consistent):** Follows industry standards for chosen technology stacks. Practitioners recognize implementations as following established patterns and conventions.

**Internals dimension (variable):** Variable names, helper functions, internal patterns may vary freely between implementations. Internal variance doesn't affect external compliance.

### Traceability

Implementation code includes references to specifications, enabling verification that major components trace to their specification requirements.

### Validation Requirements

**Behavior validation:** Automated tests from Coverage specifications verify behavioral compliance.

**Structure validation:** Static analysis and architectural tests verify standards compliance.

**Internals:** No validation required for internal implementation choices.

### Implementation Artifacts by Phase

**Development phase:** Source code, tests, configurations, build files, README with instructions, development reports, specification frontmatter updates, acceptance criteria checkbox updates.

**Deployment phase:** Infrastructure code, deployment manifests, environment configurations, deployment reports with health and endpoints, specification frontmatter updates, acceptance criteria checkbox updates. Credentials appear as references only per Isolation Principle.

**Validation phase:** Test results, coverage reports, validation summaries, specification frontmatter updates. Results map to Coverage specification test cases including coverage percentages.

**State tracking:** Implementation agents update specification frontmatter. System status aggregates from scanning frontmatter across all specifications.
