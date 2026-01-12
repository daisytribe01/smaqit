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

**Traceable outputs:** Every output traces to a prompt file. Each layer receives requirements from its prompt file. Upstream layers provide context for coherence, not requirements. Code references specifications. Tests reference requirements.

### Layer Independence

**Prompt-driven layers:** Each layer's prompt file serves as the sole source of requirements for that layer. Upstream layers provide context for coherence without dictating content. This ensures user intent guides every layer without false derivation chains.

### Single Source of Truth

**Information uniqueness:** Each piece of information exists in exactly one place. When information is needed in multiple contexts, reference the source rather than duplicate. Foundation specs contain shared requirements that multiple feature specs depend on. This prevents conflicting sources of truth, reduces maintenance burden, and ensures consistency across specifications.

### Specification Coverage

**Verified requirements:** Every requirement receives verification through traceable test coverage. Traceability enables complete specification coverage: the Coverage layer traces requirements through all upstream specs to ensure nothing is missed. Untested requirements become explicit gaps, not silent omissions.

### Self-Validating Agents

**Agent self-checking:** Agents validate their own output before declaring completion. Each agent has completion criteria and verifies them before finishing. This shifts quality assurance left—into the agent itself, not a separate review step.

### Bounded Agents

**Single-responsibility execution:** Agents execute only their designated layer or phase. Each agent has a single responsibility. Agents decline out-of-scope requests with clear redirection to the appropriate agent. This enforces separation of concerns and prevents scope creep across workflow boundaries.

### Template-Constrained Output

**Templates as scaffolds:** Templates are cognitive scaffolds, not suggestions. Templates define the exact structure agents produce. This ensures consistent output across runs, predictable input for downstream consumers, and reduced LLM variance.

### Accept Mutability, Validate Behavior

**Embrace non-determinism in artifacts, enforce determinism in outcomes.**

LLMs rarely generate identical output twice. Rather than fighting this inherent variability, smaqit accepts it:

- **Mutable artifacts**: Code, configurations, and documents may vary between runs
- **Immutable behavior**: Specifications define expected outcomes, not implementation details
- **Validation over reproducibility**: Success is measured by passing acceptance criteria, not by identical output

### Reproducible from Input Set

**Identical input sets should produce equivalent validated behavior.**

The complete set of prompts across all layers defines a reproducible workflow. Given the same prompt set:

- **Equivalent outcomes**: Acceptance criteria pass or fail consistently
- **Traceable changes**: Modifying any prompt in the set reveals requirement changes explicitly
- **Audit trail**: Prompt sets document what was requested at each layer

### Progressive Refinement

**Each layer addresses a distinct concern.**

Layers are independent but must be coherent:
- Business (intent) | Functional (behavior) | Stack (tools) | Infrastructure (environment) | Coverage (verification)
- No layer derives requirements from another—each reads from its own prompt file
- Implementation agents validate cross-layer coherence before execution

### Stateful Specifications

**Specifications track their lifecycle state through implementation phases.**

Specs are not static documents—they evolve through phases with tracked states:
- **Draft**: Spec generated, ready for implementation
- **Implemented**: Code generated, tests pass
- **Deployed**: Running in target environment
- **Validated**: All acceptance criteria verified
- **Failed**: Processing failed at some phase

### Explicit Over Implicit

**When in doubt, make it explicit.**

- State assumptions rather than assume shared context
- Define scope boundaries rather than imply them
- Reference sources rather than expect inference

### Fail-Fast on Ambiguity

**When input is unclear, stop and request clarification.**

- Do not invent requirements
- Flag assumptions explicitly

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

## File Locations (in smaqit-enabled projects)

```
project/
├── .smaqit/
│   ├── framework/        # Framework files (this directory)
│   └── templates/        # Spec and prompt templates
├── .github/
│   ├── agents/           # Agent definitions
│   └── prompts/          # Prompt files (user input)
└── specs/
    ├── business/
    ├── functional/
    ├── stack/
    ├── infrastructure/
    └── coverage/
```

## See Also

Read SMAQIT.md first for framework overview. Consult these files as needed:

| File | Purpose | When to Consult |
|------|---------|-----------------|
| [PROMPTS](PROMPTS.md) | Prompt structure, input records, agent interaction | Understanding prompt files or agent invocation |
| [LAYERS](LAYERS.md) | Five specification layers and their dependencies | Generating or validating layer specs |
| [PHASES](PHASES.md) | Three development phases and their workflows | Orchestrating multi-agent workflows |
| [TEMPLATES](TEMPLATES.md) | Template structure rules for prompts, specs, and agents | Creating or validating templates |
| [AGENTS](AGENTS.md) | Agent behaviors (actors) | Understanding agent responsibilities |
| [ARTIFACTS](ARTIFACTS.md) | Artifact rules (outputs) | Understanding spec structure and traceability |
