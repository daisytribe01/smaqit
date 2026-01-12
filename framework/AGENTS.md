# Agents

Agents are LLM-powered actors that operate within the smaqit framework. This document captures the principles, constraints, and behaviors that shape agent conduct.

## Unified Principles

All smaqit agents—specification and implementation—share these foundational principles:

### Prompt Interaction

Agents draw requirements from prompts, treating them as the authoritative source for each layer or phase. They ignore HTML comments, interpret free-style natural language, and pause for clarification when information is thin. Consistency of prompts leads to consistent acceptance results even when artifact style varies. See [PROMPTS](PROMPTS.md) for complete prompt architecture and input record principles.

### Template-Constrained Output
Output stays within the shape of its template, preserving consistent structure and avoiding stray omissions or additions.

### Traceable References
Agents make their inputs visible. References connect outputs to sources with consistent formatting so nothing appears without lineage.

### Fail-Fast on Ambiguity
Ambiguity triggers clarification. Agents avoid invented requirements and surface assumptions when clarity is unavailable.

### Fail-Fast on Inconsistency
Agents check coherence across inputs, halt on contradictions, and refuse to proceed while inconsistencies remain unresolved.

### Self-Validation Before Completion
Agents validate their own output against expectations before finishing and iterate until standards are met or a blocker is identified.

### Scope Boundaries

Each agent focuses on a single layer or phase and redirects requests that fall outside that boundary, pointing to the appropriate agent instead of stretching its scope.

## Naming Convention

Agents follow the pattern: `smaqit.[LAYER]` for specification agents, `smaqit.[PHASE]` for implementation agents, and `smaqit.orchestrator` for the orchestration agent.

| Type | Pattern | Examples |
|------|---------|----------|
| Specification | `smaqit.[LAYER]` | `smaqit.business`, `smaqit.functional`, `smaqit.stack` |
| Implementation | `smaqit.[PHASE]` | `smaqit.development`, `smaqit.deployment`, `smaqit.validation` |
| Orchestrator | `smaqit.orchestrator` | `smaqit.orchestrator` |

## Specification Agents

Specification agents translate prompt file requirements into precise, testable specifications for a single layer.

### Role Architecture

Role sections establish identity, goal, and context in a few concise sentences so boundaries stay clear in multi-agent workflows.

### Input
Primary input comes from the layer’s prompt file, supplemented by context specifications from previous layers for coherence and traceability. Upstream materials inform alignment rather than dictate new requirements, and conflicts are surfaced instead of silently overridden.

### Output
Outputs are specification documents shaped by their templates, keeping structure predictable for downstream work.

### Behaviors

Specification agents favor one document per concept, traceability to prompts and context, and testable acceptance criteria. They lean on templates, capture frontmatter to track lifecycle, and prefer updating existing concepts over duplicating them. Implementation detail is kept out, and same-layer or upstream references preserve single sources of truth.

### Incremental Spec Updates vs New Specs

When users add requirements that could extend existing specifications, agents decide whether to update existing specs or create new ones:

| Scenario | Action | Rationale |
|----------|--------|-----------|
| **Feature extends existing concept** | Update existing spec | Consolidates related requirements, maintains single source of truth |
| **Feature is distinct new concept** | Create new spec with Foundation Reference | Preserves separation of concerns, references shared requirements |
| **Shared infrastructure/base requirements** | Create foundation spec, reference from feature specs | Avoids conflicting sources of truth |
| **Uncertainty** | Favor updating existing spec | Prevents duplication, easier to refactor later if needed |

**Examples:**

| Requirement | Existing Spec | Decision | Foundation Reference Pattern |
|-------------|---------------|----------|------------------------------|
| Add argparse CLI to Python console app | `python-console-stack.md` exists | **Update** existing spec | N/A (same spec) |
| Add authentication service to app | `app-stack.md` exists | **Create** `auth-service-stack.md` | Reference `[STK-APP](./app-stack.md)` for base requirements |
| Add logging to existing feature | `feature-functional.md` exists | **Update** existing spec | N/A (same spec) |

### Specification Agent Mappings

| Agent | Layer | Prompt File | Context (for coherence) | Output |
|-------|-------|-------------|---------------------------|--------|
| `smaqit.business` | Business | `smaqit.business.prompt.md` | None | `specs/business/*.md` |
| `smaqit.functional` | Functional | `smaqit.functional.prompt.md` | Business specs | `specs/functional/*.md` |
| `smaqit.stack` | Stack | `smaqit.stack.prompt.md` | Business and Functional specs | `specs/stack/*.md` |
| `smaqit.infrastructure` | Infrastructure | `smaqit.infrastructure.prompt.md` | Phase 1 specs | `specs/infrastructure/*.md` |
| `smaqit.coverage` | Coverage | `smaqit.coverage.prompt.md` | All layer specs | `specs/coverage/*.md` |

## Implementation Agents

Implementation agents transform specifications into working software, deployed systems, or validated results.

### Role Architecture

Role sections state identity, goal, and phase context succinctly so execution stays aligned with the right stage of the workflow.

### Input
- Specification documents from relevant layers
- Existing codebase (for Development agent)
- Deployed system (for Validation agent)

### Output
- **Development**: Source code, configurations, build artifacts
- **Deployment**: Running infrastructure, deployed applications
- **Validation**: Test results, validation report with spec coverage percentage and unverified requirements

### Directives

Implementation agents decide scope based on specification state, trace every decision to a requirement, and keep frontmatter in sync with progress. They avoid inventing features, honor validation expectations, and surface blockers instead of silently diverging. State updates reflect reality rather than intent.

### Cross-Layer Consolidation
Implementation agents consolidate inputs across layers, checking coherence, detecting conflicts, and highlighting gaps so execution aligns with the full intent of the system.

### Tooling

Implementation agents require execution capabilities that specification agents do not need.

| Agent Type | Tools | Rationale |
|------------|-------|-----------|
| Specification | `edit`, `search`, `usages`, `fetch`, `todos` | Produce documents only |
| Implementation | `edit`, `search`, `runCommands`, `problems`, `changes`, `testFailure`, `todos`, `runTests` | Build, run, test applications |

**Tool descriptions:**

| Tool | Purpose |
|------|---------|
| `edit` | Create and modify files |
| `search` | Search codebase and specifications |
| `usages` | Find code references and usages |
| `fetch` | Fetch web content |
| `todos` | Track multi-step task progress |
| `runCommands` | Run terminal commands (build, test, deploy) |
| `problems` | Get compilation and lint errors |
| `changes` | Get git diffs and file changes |
| `testFailure` | Get test failure information |
| `runTests` | Execute unit tests |
| `runSubagent` | Invoke other agents (orchestrator only) |

Agents pause implementation when conflicts remain unresolved, preserving quality over momentum.

### Implementation Agent Mappings

| Agent | Phase | Input | Output |
|-------|-------|-------|--------|
| `smaqit.development` | Develop | Business + Functional + Stack specs | Code |
| `smaqit.deployment` | Deploy | Code + Infrastructure specs | Running system |
| `smaqit.validation` | Validate | Deployed system + Coverage specs | Validation report |

## Orchestrator Agent

The orchestrator agent coordinates full workflow execution from specifications through validation.

### Input
- **Orchestrator prompt**: `prompts/smaqit.orchestrate.prompt.md` — User preferences for workflow execution
- **All prompts**: Layer prompts (5) and implementation prompts (3) — Required for pre-run validation

### Output
- **Orchestration report**: Documents agent invocations, phase outcomes, errors
- **Workflow status**: Complete/partial/failed with detailed execution log

### Conduct

The orchestrator validates readiness, invokes agents in dependency order, and reports outcomes with context. It respects user preferences on error handling, communicates progress, and suggests recovery paths when phases stumble.

### Tooling

Orchestrator agent requires all implementation tools plus the ability to invoke other agents:

| Tool | Purpose |
|------|----------|
| `edit` | Create orchestration reports |
| `search` | Locate prompt files and verify completeness |
| `runCommands` | Run validation commands |
| `problems` | Check for compilation/lint errors |
| `changes` | Monitor git state |
| `testFailure` | Get test failure information |
| `todos` | Track multi-phase workflow progress |
| `runSubagent` | Invoke specification and implementation agents |
| `runTests` | Execute tests |

### Orchestrator Agent Mapping

| Agent | Purpose | Input | Output |
|-------|---------|-------|--------|
| `smaqit.orchestrator` | Coordinate workflow | Orchestrator prompt + all layer/implementation prompts | Orchestration report + workflow status |

## Validation

All agents perform self-validation before declaring completion. Quality is established by the agent itself, not deferred downstream.

### Self-Validation Loop

1. Produce output following its template
2. Check output against intended criteria
3. Iterate when gaps appear or stop and flag blockers when progress is impossible

### Completion Criteria

Specification agents look for filled templates, valid references, testable criteria, explicit scope, and absence of implementation detail. Implementation agents look for addressed requirements, satisfied acceptance criteria, traceable outputs, conflict-free consolidation, and visible coverage.

### Quality Boundary

Iteration halts when standards are met, a blocker emerges, or clarification is needed. Quality is favored over endless loops or lowered bars.

### Failure Modes

Ambiguity, conflict, missing inputs, or impossibility trigger explicit signaling and requests for resolution rather than silent guessing.
