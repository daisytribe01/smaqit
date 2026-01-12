# Agents

Agents are LLM-powered actors that operate within the smaqit framework. This document defines the principles governing agent behavior and interactions.

## Unified Principles

All smaqit agents—specification and implementation—share these foundational principles:

### Prompt Interaction

**Agents as prompt interpreters:** Agents receive requirements from prompt files, interpreting natural language input without rigid structure enforcement. Prompts serve as input records capturing user intent across layers.

**Comment blindness:** Example content wrapped in HTML comments exists for human guidance only. Agents process actual content, ignoring illustrative examples to prevent contamination.

**Sufficiency validation:** When prompt content lacks necessary information, agents request clarification using natural language guidance that reflects the specific gap rather than referencing template structure.

**Reproducible outcomes:** Given identical prompt sets across layers, acceptance criteria validation outcomes remain equivalent across runs, acknowledging that artifact internals may vary due to LLM non-determinism.

### Template-Constrained Output

**Templates as cognitive scaffolds:** Output structure follows designated templates exactly. Templates define required sections, acceptable formats, and structural boundaries that reduce variance and enable predictable downstream consumption.

**Section completeness:** Output includes all template-defined sections. Missing sections indicate incomplete work rather than optional omissions.

### Traceable References

**Explicit sourcing:** Every output element traces to its input source. References use consistent formats enabling downstream agents and humans to verify provenance.

**No invented content:** Output derives from input sources. When sources are absent or ambiguous, agents surface the gap rather than fabricating content.

### Fail-Fast on Ambiguity

**Clarification over guessing:** Ambiguous input triggers clarification requests rather than assumptions. Explicit assumption flagging occurs only when clarification is unavailable and progress requires making a choice.

### Fail-Fast on Inconsistency

**Coherence verification:** Agents verify consistency across all input sources before producing output. Contradictions halt progress until resolved.

**Conflict reporting:** When inputs contradict, agents report the conflict with context from each source, enabling informed resolution decisions.

### Self-Validation Before Completion

**Completion criteria checking:** Before declaring work complete, agents verify output against their completion criteria. Unmet criteria trigger iteration or escalation, never silent completion.

**Quality boundaries:** Agents iterate until criteria are met, blocking issues are surfaced, or clarification is required. Lowering quality standards to force completion violates self-validation.

### Scope Boundaries

**Single responsibility:** Each agent addresses one layer or phase. Scope boundaries prevent conflicting concerns within single executions.

**Boundary enforcement:** Out-of-scope requests receive clear redirection to the appropriate agent, preserving separation of concerns across the framework.

## Naming Convention

Agent names follow consistent patterns based on their type: layer-specific specification agents, phase-specific implementation agents, and a single orchestrator agent.

## Specification Agents

Specification agents translate prompt file requirements into precise, testable specifications for a single layer.

### Role Architecture

**Identity establishment:** Agent role sections open with direct identity statements, goal declarations, and context positioning. This upfront clarity prevents scope confusion and context pollution in multi-agent workflows.

**Concise framing:** Role descriptions remain brief (3-4 sentences), balancing clarity with cognitive load management.

### Input Sources

**Primary source:** Requirements come from layer-specific prompt files where users express their needs in natural language.

**Context sources:** Specifications from upstream layers provide coherence context and traceability references, without serving as requirements themselves.

**Layer independence with coherence:** Each layer reads requirements from its own prompt. Upstream layers inform coherence without dictating content. When prompt requirements conflict with existing upstream specs, agents surface the tension rather than silently resolving it.

### Output Artifacts

Specification documents follow layer-specific templates, with one document per distinct concept. Templates define structure, sections, and metadata requirements.

### Incremental Specification Updates

**Update vs create:** When requirements extend existing concepts, agents update existing specifications rather than creating duplicates. Distinct new concepts justify new specification documents.

**Foundation references:** Shared requirements across specifications live in foundation specs, referenced rather than duplicated. Feature specs reference foundations, avoiding conflicting sources of truth.

**Decision heuristic:** Uncertainty favors updating existing specs. Separation is easier to introduce later than consolidation after duplication.

### Specification Agent Types

**Layer-specific agents:** Each specification layer has a dedicated agent. Business agents work with stakeholder goals, functional agents with experience requirements, stack agents with technology preferences, infrastructure agents with deployment requirements, coverage agents with test requirements.

## Implementation Agents

Implementation agents transform specifications into working software, deployed systems, or validated results.

### Role Architecture

**Identity and workflow position:** Implementation agent roles establish identity, goal, and phase context upfront. This positions the agent within the sequential workflow and clarifies boundaries.

**Concise framing:** Role descriptions remain brief, stating what the agent produces from what inputs in 3-4 sentences.

### Input Sources

**Specification documents:** Agents consume specifications from relevant layers as contracts defining required behavior.

**Existing artifacts:** Development agents read existing codebases, validation agents access deployed systems.

### Output Artifacts

**Development outputs:** Source code, configurations, build artifacts, and test suites.

**Deployment outputs:** Running infrastructure, deployed applications, and operational endpoints.

**Validation outputs:** Test results, validation reports with coverage metrics, and unverified requirement documentation.

### Frontmatter State Tracking

**Lifecycle progression:** Implementation agents update specification frontmatter to reflect processing outcomes. Status fields and timestamps track each spec's progression through phases.

**State aggregation:** Phase status derives from scanning specification frontmatter. Agents update individual files; the system aggregates current state by reading all specs.

### Cross-Layer Consolidation

**Multi-layer coordination:** Implementation agents receive specifications from multiple layers and consolidate them before execution. This consolidation phase verifies coherence, detects conflicts, analyzes gaps, and surfaces needed amendments before proceeding.

**Conflict surface:** When specifications across layers contradict or omit necessary content, agents surface these issues rather than proceeding with flawed assumptions.

### Tool Access

**Specification vs implementation tooling:** Specification agents work with documentation tools (editing, searching, reference finding). Implementation agents additionally access execution capabilities (command running, compilation checking, test execution, git state inspection).

**Orchestrator capabilities:** The orchestrator agent combines implementation tools with agent invocation abilities, coordinating multi-agent workflows.

### Implementation Agent Types

**Phase-specific agents:** Each implementation phase has a dedicated agent. Development agents produce code from specifications, deployment agents create running systems in target environments, validation agents verify compliance against acceptance criteria.

## Orchestrator Agent

The orchestrator agent coordinates full workflow execution from specifications through validation.

### Input Sources

**Orchestrator configuration:** User preferences for workflow execution captured in the orchestrator prompt.

**Complete prompt set:** All layer prompts and implementation prompts inform pre-run validation when requested.

### Output Artifacts

**Orchestration documentation:** Reports document agent invocations, phase outcomes, and error contexts.

**Workflow status:** Complete, partial, or failed execution states with detailed logs.

### Orchestration Principles

**Dependency ordering:** Agents execute in correct sequence (specification agents before implementation, upstream layers before downstream).

**Phase boundary enforcement:** Each phase completes verification before the next begins.

**Error context:** Failures receive full context documentation including phase, agent, and input state.

**User preference respect:** Error handling follows user choices (halt on error versus continue with partial completion).

**Validation gate:** When pre-run validation is requested, workflow cannot start until all prompts pass sufficiency checks.

### Tool Access

Orchestrator agents access implementation tools plus agent invocation capabilities, enabling workflow coordination across multiple specialized agents.

## Validation

All agents perform self-validation before declaring completion.

### Self-Validation Loop

**Iterative refinement:** Agents produce output, check against completion criteria, iterate until criteria are met or blocking issues surface, then declare completion or escalate.

### Completion Criteria

**For specification agents:** Template sections filled completely, upstream references valid and accessible, acceptance criteria testable and measurable, scope boundaries explicitly stated, no implementation details leaked into specifications.

**For implementation agents:** All referenced specification requirements addressed, acceptance criteria satisfied, outputs traceable to input specifications, no unspecified features added, cross-layer consolidation completed without conflicts (development and deployment), specification coverage with unverified requirements identified (validation).

### Quality Boundary

**Stop conditions:** Agents stop iterating when completion criteria are met, blocking issues prevent progress, or clarification is required from upstream sources.

**Prohibited behaviors:** Infinite iteration without progress, lowering quality standards to force completion, inventing solutions to bypass blockers.