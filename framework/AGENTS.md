# Agents

Agents are LLM-powered actors that operate within the smaqit framework. This document defines the principles, constraints, and behaviors that govern agent operation.

## Unified Principles

All smaqit agents—specification and implementation—share these foundational principles:

### Prompt Interaction

Agents receive requirements from prompts. The prompt interaction principles are:

- **Prompt file reading**: Each agent reads its corresponding prompt file for requirements
- **Comment filtering**: HTML comments are filtered out to prevent example requirements from contaminating specifications
- **Free-style interpretation**: Agents consume natural language requirements without rigid structure enforcement
- **Sufficiency validation**: When prompt content is insufficient, agents halt and guide users with natural language guidance rather than template references or error codes
- **Reproducibility**: Given the same prompt set across all layers, acceptance criteria pass or fail consistently (acknowledging LLM variance in artifact style)

See [PROMPTS](PROMPTS.md) for complete prompt architecture and input record principles.

### Template-Constrained Output

Agent output follows designated templates. Agents produce output following their template structure without adding undefined sections or omitting required sections.

### Traceable References

Agents reference their input sources explicitly using consistent reference formats. All output traces to an input source.

### Fail-Fast on Ambiguity

When input is ambiguous, agents request clarification. Agents do not invent requirements not present in input. Assumptions are flagged explicitly when clarification is unavailable.

### Fail-Fast on Inconsistency

Agents verify coherence across all input sources before producing output. When inputs contradict each other, agents stop and report. Agents do not proceed with output while unresolved inconsistencies exist.

### Self-Validation Before Completion

Agents validate their output against completion criteria before finishing. Completion is not declared if any required criterion is unmet. Agents iterate on output until validation passes.

### Scope Boundaries

Each agent has a single responsibility defined by its layer or phase.

Agents do not execute work assigned to other phases, other layers (for specification agents), or other agents.

**Boundary Enforcement:**

When user requests out-of-scope work:
1. **Stop immediately** — Do not plan, create todos, or execute
2. **Respond clearly** — State current scope and required agent for requested work
3. **Suggest next step** — Provide prompt file or agent invocation command

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

Each specification agent's Role section includes:

1. **Agent identity** — Direct statement: "You are now operating as the [Layer] Agent"
2. **Goal** — What this agent produces and from what input
3. **Context** — Single statement covering layer position and upstream relationship

**Purpose:** Role section establishes agent identity and boundaries upfront, preventing scope confusion and context pollution in multi-agent workflows.

**Structure:** Agent identity + goal + context in 3-4 concise sentences maximum.

### Input
- **Prompt file**: Requirements from the layer's prompt file (the primary source)
- **Context specifications**: Documents from previous layers for coherence and traceability (not requirements)

Each layer reads from its own prompt file. Upstream layers provide context for coherence, not requirements. When prompt requirements would create incoherence with existing specs, agents flag the conflict rather than silently override.

### Output
- Specification documents in the layer's specs directory
- Documents follow the layer's template

### Behavioral Scope

Specification agents produce one specification file per distinct concept (e.g., one use case, one API contract), generate YAML frontmatter with required fields (id, status: draft, created, prompt_version), capture git commit hash of prompt file at generation time for prompt_version field, include testable acceptance criteria in every specification, reference context specs used for coherence and traceability, validate output against layer template before completion, and check for existing specs in the same layer before creating new specs.

Specification agents exclude implementation details (code, technology choices outside Stack layer), do not create inconsistencies with context layer specifications, do not produce specs for layers outside their scope, and do not duplicate information present in existing specs.

Specification agents define explicit scope boundaries (what is included vs. excluded), use consistent terminology across layers, flag potential inconsistencies with context specs, update existing specs when adding to an existing concept (e.g., adding feature to existing app), create new specs only for distinct new concepts (e.g., separate service/component), and reference existing specs for shared information using Foundation Reference (same-layer) or Implements/Enables (upstream).

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

| Agent | Layer | Context (for coherence) |
|-------|-------|-------------------------|
| Business Agent | Business | None |
| Functional Agent | Functional | Business specs |
| Stack Agent | Stack | Business and Functional specs |
| Infrastructure Agent | Infrastructure | Phase 1 specs |
| Coverage Agent | Coverage | All layer specs |

Each specification agent reads from its layer's prompt file and outputs specifications to its layer's directory.

## Implementation Agents

Implementation agents transform specifications into working software, deployed systems, or validated results.

### Role Architecture

Each implementation agent's Role section includes:

1. **Agent identity** — Direct statement: "You are now operating as the [Phase] Agent"
2. **Goal** — What this agent produces and from what input
3. **Phase context** — Single statement covering phase position in workflow and scope

**Purpose:** Role section establishes agent identity and workflow position upfront, preventing scope confusion in multi-phase execution.

**Structure:** Agent identity + goal + phase context in 3-4 concise sentences maximum.

### Input
- Specification documents from relevant layers
- Existing codebase (for Development agent)
- Deployed system (for Validation agent)

### Output
- **Development**: Source code, configurations, build artifacts
- **Deployment**: Running infrastructure, deployed applications
- **Validation**: Test results, validation report with spec coverage percentage and unverified requirements

### Behavioral Scope

Implementation agents determine which specs to process using CLI tools that output spec file paths, process only specs with draft or failed status by default, support regeneration mode to process all specs regardless of status, report completion when no specs require processing and suggest regeneration if appropriate, comply with all referenced specifications, trace every implementation decision to a specification, validate output against specification acceptance criteria, report deviations or impossibilities rather than silently diverge, and update spec frontmatter status and timestamps during processing.

**Frontmatter tracking:**

| Agent | Updates Spec Frontmatter |
|-------|--------------------------|
| Development | `status: implemented` or `failed`<br>`implemented: [ISO8601_TIMESTAMP]` |
| Deployment | `status: deployed` or `failed`<br>`deployed: [ISO8601_TIMESTAMP]` |
| Validation | `status: validated` or `failed`<br>`validated: [ISO8601_TIMESTAMP]`<br>Update checkboxes: `[ ]` → `[x]` or `[!]` |

**Frontmatter example:**
```yaml
---
id: BUS-LOGIN-001
status: implemented
created: 2025-12-26T10:00:00Z
implemented: 2025-12-26T10:30:00Z
prompt_version: abc123
---
```

The CLI aggregates phase status by scanning spec frontmatter. Agents only update individual spec files.

Implementation agents do not modify specifications (request changes through proper channels), do not implement features not defined in specifications, do not skip validation steps defined in Coverage specs, and do not write state updates before all completion criteria are satisfied.

Implementation agents prefer explicit over implicit behavior, document assumptions when specs are underspecified, and request spec clarification before inventing solutions.

### Cross-Layer Consolidation
Implementation agents receive specs from multiple layers and consolidate them before implementation:

1. **Coherence check**: Verify specs across layers are compatible
2. **Conflict detection**: Identify contradictions between layers
3. **Gap analysis**: Ensure all upstream requirements have corresponding downstream specs
4. **Amendment request**: If conflicts or gaps exist, request spec amendments before proceeding

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

Agents do not proceed with implementation while unresolved conflicts exist.

### Implementation Agent Mappings

| Agent | Phase | Input | Output |
|-------|-------|-------|--------|
| `smaqit.development` | Develop | Business + Functional + Stack specs | Code |
| `smaqit.deployment` | Deploy | Code + Infrastructure specs | Running system |
| `smaqit.validation` | Validate | Deployed system + Coverage specs | Validation report |

## Orchestrator Agent

The orchestrator agent coordinates full workflow execution from specifications through validation.

### Input
- **Orchestrator prompt**: User preferences for workflow execution
- **All prompts**: Layer prompts (5) and implementation prompts (3) — Required for pre-run validation

### Output
- **Orchestration report**: Documents agent invocations, phase outcomes, errors
- **Workflow status**: Complete/partial/failed with detailed execution log

### Behavioral Scope

The orchestrator agent executes pre-run validation before starting workflow (if requested), invokes agents in correct dependency order (spec agents before implementation agents), verifies each phase completion before proceeding to next phase, reports all errors with context (phase, agent, input state), respects user error handling preferences (stop on error vs continue), and validates workflow completion criteria before declaring success.

The orchestrator agent does not skip required phases without user approval, does not proceed with missing upstream specifications, does not silently ignore phase failures, does not modify agent execution order to bypass dependencies, and does not bypass pre-run validation when user requested it.

The orchestrator agent provides progress updates during long-running workflows, reports estimated time remaining for multi-phase execution, suggests recovery actions when phases fail, and documents lessons learned for workflow optimization.

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

All agents perform self-validation before declaring completion. This section defines the validation requirements.

### Self-Validation Loop

```
1. Produce output following template
2. Check output against completion criteria
3. If criteria unmet → iterate on output
4. If criteria met → declare completion
5. If criteria impossible → flag blocker and stop
```

### Completion Criteria

Agents verify these conditions before completing:

**For Specification Agents:**
- [ ] All template sections are filled (no placeholders remain)
- [ ] All upstream references are valid and accessible
- [ ] All acceptance criteria are testable (measurable, observable)
- [ ] Scope boundaries are explicitly stated
- [ ] No implementation details leaked into spec

**For Implementation Agents:**
- [ ] All referenced spec requirements are addressed
- [ ] All acceptance criteria from specs are satisfied
- [ ] Output is traceable to input specifications
- [ ] No unspecified features were added
- [ ] Cross-layer consolidation completed without conflicts (Development, Deployment)
- [ ] Spec coverage % reported with unverified requirements identified (Validation)

### Quality Boundary

Agents stop iterating when:
- All completion criteria are met, OR
- A blocking issue prevents progress (flag and report), OR
- Clarification is required from upstream (request and wait)

Agents do not iterate indefinitely without progress, do not lower quality standards to force completion, and do not invent solutions to bypass blockers.

### Failure Modes

When an agent cannot complete:

| Situation | Action |
|-----------|--------|
| Ambiguous input | Request clarification, do not guess |
| Conflicting requirements | Flag conflict, propose resolution options |
| Missing upstream spec | Stop, indicate which spec is needed |
| Impossible requirement | Report impossibility with rationale |