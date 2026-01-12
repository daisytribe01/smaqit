# smaqit Framework

Spec-driven agent orchestration where specifications are split into layers and phases. Users input requirements in prompt files, AI specification agents read from these prompt files to write specifications, then implementation agents generate outputs from those specifications.

## Core Principles

### Phases as Workflow Units

**Each phase includes specifications and implementation together.**

Phases are the primary workflow units in smaqit. Users can generate all specifications first, but the recommended approach is to complete each phase (specifications + implementation) before moving to the next. This provides faster feedback and validates the system incrementally.

### Specs Before Code

**Never write implementation without a corresponding specification.**

Specifications are not documentation—they are the source of truth. Implementation agents consume specs as contracts, not guidelines. This inverts the common pattern where code comes first and docs follow.

### Traceability Across Layers

Every output traces back to its originating prompt. Requirements live at the layer where users express them; upstream layers provide context rather than new demands. Code reflects specifications, and tests exercise those requirements.

### Layer Independence

**Each layer's prompt file is the sole source of requirements for that layer.**

Each layer has its own prompt file where users input requirements. Upstream layers provide context for coherence, not requirements. This ensures that user intent guides every layer without false derivation chains.

### Single Source of Truth

Each piece of information belongs in a single place. When it is needed elsewhere, reference the source rather than duplicate. Foundation specifications carry shared requirements so multiple feature specs stay consistent and conflicts are avoided.

### Specification Coverage

Requirements invite verification through traceable coverage. The Coverage layer follows acceptance criteria across upstream specs so omissions become visible gaps rather than silent misses.

### Self-Validating Agents

Agents validate their own output before declaring completion. Each agent carries completion criteria and checks itself first, pushing quality assurance into the act of production rather than a separate review step.

### Bounded Agents

**Agents execute only their designated layer or phase.**

Each agent has a single responsibility. Agents decline out-of-scope requests with clear redirection to the appropriate agent. This enforces separation of concerns and prevents scope creep across workflow boundaries.

### Template-Constrained Output

Templates are cognitive scaffolds rather than suggestions. They anchor consistent output, predictable inputs for downstream consumers, and lower variance in generated artifacts.

### Accept Mutability, Validate Behavior

Embrace non-determinism in artifacts while enforcing determinism in outcomes. Code, configurations, and documents may vary between runs, yet specifications define expected behavior and success is measured by meeting acceptance criteria, not identical output.

### Reproducible from Input Set

Identical input sets should lead to equivalent validated behavior. A full prompt set defines the workflow: the same prompts drive consistent acceptance results, changes surface explicitly, and the prompt set itself forms the audit trail.

### Progressive Refinement

Each layer addresses a distinct concern. Independence is balanced with coherence across Business (intent), Functional (behavior), Stack (tools), Infrastructure (environment), and Coverage (verification). Requirements arise from each layer’s prompt, while implementation agents check for cross-layer compatibility before execution.

### Stateful Specifications

Specifications carry lifecycle state through implementation phases. They move from draft to implemented, deployed, validated, or failed, reflecting the journey rather than remaining static documents.

### Explicit Over Implicit

When in doubt, make it explicit. Stated assumptions, clear scope boundaries, and explicit references prevent reliance on inference.

### Fail-Fast on Ambiguity

When input is unclear, pause for clarification. Avoid invented requirements and surface assumptions plainly.

## Quick Reference

### Layers

| Layer | Question | Purpose |
|-------|----------|---------|
| Business | Why? | Use cases, actors, goals |
| Functional | What? | Behaviors, contracts, flows |
| Stack | With what? | Languages, frameworks, libraries |
| Infrastructure | Where? | Compute, networking, observability |
| Coverage | Verified? | Integration, E2E, acceptance testing |

### Phases

| Phase | Spec Agents | Impl Agent | Output |
|-------|-------------|------------|--------|
| Develop | Business → Functional → Stack | Development | Working application |
| Deploy | Infrastructure | Deployment | Running system |
| Validate | Coverage | Validation | Validation report |

## See Also

Related frameworks deepen specific perspectives:

- [PROMPTS](PROMPTS.md) — Prompt structure, input records, agent interaction
- [LAYERS](LAYERS.md) — Layer definitions and their relationships
- [PHASES](PHASES.md) — Phase sequencing and workflows
- [TEMPLATES](TEMPLATES.md) — Template structure philosophy
- [AGENTS](AGENTS.md) — Agent behaviors and boundaries
- [ARTIFACTS](ARTIFACTS.md) — Artifact meaning and traceability
