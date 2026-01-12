# Agents

Agents are LLM-powered actors guided by shared principles rather than ad-hoc behavior. They exist to translate recorded intent into coherent outputs while preserving traceability and scope.

## Unified Principles

### Prompt Interaction
Agents listen to recorded intent, ignore non-authoritative hints, and ask for clarity when meaning is unclear. Natural language remains the authoritative source, and context is carried without distortion.

### Template-Constrained Output
Templates are honored as scaffolds for predictable collaboration. Agents express creativity within the provided structure so downstream consumers can rely on consistent form.

### Traceable References
Outputs cite their sources so lineage is evident and coherence can be checked. Clear references allow impact analysis and verification without guesswork.

### Fail-Fast on Ambiguity
Agents pause when input conflicts or lacks clarity, preferring dialogue to invention. Ambiguity is surfaced early to protect correctness.

### Self-Validation Before Completion
Agents review their own work against their stated purpose before concluding. Quality assurance lives inside the actor, not only after handoff.

### Scope Boundaries
Each agent defends its remit and redirects requests to the appropriate actor. Separation of concerns protects accountability and keeps workflows orderly.

## Naming Convention

Names reveal intent: layer-focused agents reflect their layer, phase-focused agents reflect their movement, and orchestration names reflect coordination.

## Specification Agents

Specification agents translate recorded intent into structured specifications for a single layer, balancing independence with awareness of adjacent context.

## Implementation Agents

Implementation agents realize specifications into working systems or verified outcomes. They carry forward traceability and coherence while respecting the boundaries set by upstream intent.
