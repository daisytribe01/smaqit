# smaqit Framework

Spec-driven agent orchestration where specifications express intent and implementations express realization. Layers and phases keep intent, behavior, tooling, environment, and verification distinct while remaining coherent.

## Core Principles

### Phases as Workflow Units
Phases pair specification and implementation so feedback arrives in small, end-to-end increments. Each phase is a complete loop that reveals coherence before moving forward.

### Specs Before Code
Specifications precede implementation and act as contracts. Code, deployments, and validations flow from articulated intent rather than improvisation.

### Traceability Across Layers
Every artifact points back to its originating intent, creating an unbroken line from requirement to verification. Context flows downstream to maintain coherence without replacing local intent.

### Layer Independence
Each layer listens to its own request stream and expresses its own perspective. Upstream material offers context, not commandments, preserving clarity of origin.

### Single Source of Truth
Information anchors once and is referenced elsewhere. Shared foundations carry common ideas; feature expressions lean on those anchors instead of duplicating them.

### Specification Coverage
Requirements and their verification remain linked. Coverage thinking treats untested intent as a visible gap rather than silent risk.

### Self-Validating Agents
Agents pause to evaluate their own output against their purpose before claiming completion. Quality lives inside the actor, not only after handoff.

### Bounded Agents
Each agent holds a focused charter and redirects out-of-scope demands. Clear boundaries prevent drift and protect accountability.

### Template-Constrained Output
Templates provide shape and vocabulary so outputs stay predictable and interoperable. Structure is a scaffold, not a suggestion.

### Accept Mutability, Validate Behavior
Surface form may vary across runs, yet expected behavior stays constant. Success is judged by fulfilled criteria rather than identical artifacts.

### Reproducible from Input Set
Equivalent input collections yield equivalent validated behavior. Changing any input makes lineage visible and invites deliberate reconsideration.

### Progressive Refinement
Layers address distinct concerns—intent, behavior, stack, environment, verification—allowing clarity within each while composing a whole.

### Stateful Specifications
Specifications acknowledge their journey through draft, implementation, deployment, and validation. State reflects lived progress rather than static description.

### Explicit Over Implicit
Assumptions surface in writing. Boundaries, references, and rationales are declared so decisions remain auditable.

### Fail-Fast on Ambiguity
Unclear input triggers a halt and a request for clarity. Invention without grounding is avoided in favor of explicit confirmation.

## Quick Reference

- **Layers** span from intent (Business) to verification (Coverage), with Stack and Infrastructure anchoring technology and environment along the way.
- **Phases** progress through developing, deploying, and validating, each completing a full loop before the next begins.

## See Also

Read SMAQIT.md first for framework overview. Consult these files as needed:

| File | Purpose | When to Consult |
|------|---------|-----------------|
| [PROMPTS](PROMPTS.md) | Prompt structure, input records, agent interaction | Understanding prompt philosophy |
| [LAYERS](LAYERS.md) | Five specification layers and their dependencies | Understanding layer perspectives |
| [PHASES](PHASES.md) | Three development phases and their workflows | Understanding phase flow |
| [TEMPLATES](TEMPLATES.md) | Template structure rules for prompts, specs, and agents | Understanding structural scaffolds |
| [AGENTS](AGENTS.md) | Agent behaviors (actors) | Understanding agent responsibilities |
| [ARTIFACTS](ARTIFACTS.md) | Artifact rules (outputs) | Understanding traceability and outputs |
