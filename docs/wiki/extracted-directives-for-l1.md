# Extracted Directives for L1 Compilation

**Purpose:** This document catalogs directives extracted from L0 framework files during the 2026-01-12 cleanup. These directives will be compiled into L1 templates by Agent-L1.

**Context:** Task B001 (Level Up Architecture) established that L0 contains principles and philosophy, L1 contains directives with placeholders, and L2 contains concrete implementations. This extraction preserves the "what to enforce" content removed from L0 for future L1 work.

---

## From AGENTS.md

### Unified Principles - Prompt Interaction

**Extracted directives:**
- MUST read corresponding prompt files
- MUST ignore all HTML comments in prompt files
- MUST request clarification if prompt content is insufficient using natural language guidance
- Prompt file path pattern: `.github/prompts/smaqit.[layer].prompt.md`

### Unified Principles - Template-Constrained Output

**Extracted directives:**
- MUST produce output following designated template
- MUST NOT add sections not defined in template
- MUST NOT omit required sections from template

### Unified Principles - Traceable References

**Extracted directives:**
- MUST reference input sources explicitly
- SHOULD use consistent reference format: `[LayerName](path/to/spec.md)`
- MUST NOT produce output that cannot be traced to input

### Unified Principles - Fail-Fast on Ambiguity

**Extracted directives:**
- MUST request clarification when input is ambiguous
- MUST NOT invent requirements not present in input
- SHOULD flag assumptions explicitly when clarification unavailable

### Unified Principles - Fail-Fast on Inconsistency

**Extracted directives:**
- MUST verify coherence across all input sources before producing output
- MUST stop and report when inputs contradict each other
- MUST NOT proceed with output while unresolved inconsistencies exist

### Unified Principles - Self-Validation Before Completion

**Extracted directives:**
- MUST validate output against completion criteria before finishing
- MUST NOT declare completion if any required criterion is unmet
- SHOULD iterate on output until validation passes

### Scope Boundaries

**Extracted directives:**
- MUST NOT execute work assigned to other phases
- MUST NOT execute work assigned to other layers (for specification agents)
- MUST NOT execute work assigned to other agents
- When user requests out-of-scope work:
  1. Stop immediately (do not plan, create todos, or execute)
  2. Respond clearly (state current scope and required agent)
  3. Suggest next step (provide prompt file or agent invocation command)

### Specification Agents - Input

**Extracted directives:**
- Prompt file path: `.github/prompts/smaqit.[layer].prompt.md` (primary source)
- Context specifications from previous layers for coherence (not requirements)
- MUST flag conflict when prompt requirements create incoherence with existing specs

### Specification Agents - Output

**Extracted directives:**
- Output location: `specs/{layer}/`
- MUST follow template from `templates/{layer}.template.md`

### Specification Agents - Directives

**Extracted directives:**
- MUST produce one specification file per distinct concept
- MUST generate YAML frontmatter with required fields: `id`, `status: draft`, `created`, `prompt_version`
- MUST capture git commit hash of prompt file at generation time for `prompt_version` field
- MUST include testable acceptance criteria in every specification
- MUST reference context specs used for coherence and traceability
- MUST validate output against layer template before completion
- MUST check for existing specs in same layer before creating new specs

- MUST NOT include implementation details (code, technology choices outside Stack layer)
- MUST NOT create inconsistencies with context layer specifications
- MUST NOT produce specs for layers outside their scope
- MUST NOT duplicate information present in existing specs

- SHOULD define explicit scope boundaries (inclusions vs exclusions)
- SHOULD use consistent terminology across layers
- SHOULD flag potential inconsistencies with context specs
- SHOULD update existing specs when adding to existing concept
- SHOULD create new specs only for distinct new concepts
- SHOULD reference existing specs for shared information using Foundation Reference (same-layer) or Implements/Enables (upstream)

### Specification Agent Mappings

**Extracted table:**
| Agent | Layer | Prompt File | Context (for coherence) | Output |
|-------|-------|-------------|---------------------------|--------|
| `smaqit.business` | Business | `smaqit.business.prompt.md` | None | `specs/business/*.md` |
| `smaqit.functional` | Functional | `smaqit.functional.prompt.md` | Business specs | `specs/functional/*.md` |
| `smaqit.stack` | Stack | `smaqit.stack.prompt.md` | Business and Functional specs | `specs/stack/*.md` |
| `smaqit.infrastructure` | Infrastructure | `smaqit.infrastructure.prompt.md` | Phase 1 specs | `specs/infrastructure/*.md` |
| `smaqit.coverage` | Coverage | `smaqit.coverage.prompt.md` | All layer specs | `specs/coverage/*.md` |

### Implementation Agents - Directives

**Extracted directives:**
- MUST determine which specs to process using `smaqit plan --phase=[PHASE]`
- MUST process only specs with `status: draft` or `status: failed` by default
- MUST support regeneration mode via `--regen` flag to process all specs regardless of status
- MUST report completion when no specs require processing and suggest `--regen` flag if appropriate
- MUST comply with all referenced specifications
- MUST trace every implementation decision to a specification
- MUST validate output against specification acceptance criteria
- MUST report deviations or impossibilities rather than silently diverge
- MUST update spec frontmatter status and timestamps during processing

**Frontmatter tracking table:**
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

- MUST NOT modify specifications (request changes through proper channels)
- MUST NOT implement features not defined in specifications
- MUST NOT skip validation steps defined in Coverage specs
- MUST NOT write state updates before all completion criteria are satisfied

- SHOULD prefer explicit over implicit behavior
- SHOULD document assumptions when specs are underspecified
- SHOULD request spec clarification before inventing solutions

### Cross-Layer Consolidation

**Extracted directives:**
- Implementation agents MUST consolidate specs before implementation:
  1. Coherence check: Verify specs across layers are compatible
  2. Conflict detection: Identify contradictions between layers
  3. Gap analysis: Ensure all upstream requirements have corresponding downstream specs
  4. Amendment request: If conflicts or gaps exist, request spec amendments before proceeding
- MUST NOT proceed with implementation while unresolved conflicts exist

### Tooling

**Extracted tool requirements:**

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

### Implementation Agent Mappings

**Extracted table:**
| Agent | Phase | Input | Output |
|-------|-------|-------|--------|
| `smaqit.development` | Develop | Business + Functional + Stack specs | Code |
| `smaqit.deployment` | Deploy | Code + Infrastructure specs | Running system |
| `smaqit.validation` | Validate | Deployed system + Coverage specs | Validation report |

### Orchestrator Agent - Directives

**Extracted directives:**
- Input prompt path: `prompts/smaqit.orchestrate.prompt.md`
- MUST execute pre-run validation before starting workflow (if requested)
- MUST invoke agents in correct dependency order: 5 spec agents → 3 implementation agents
- MUST verify each phase completion before proceeding to next phase
- MUST report all errors with context (phase, agent, input state)
- MUST respect user error handling preferences (stop on error vs continue)
- MUST validate workflow completion criteria before declaring success

- MUST NOT skip required phases without user approval
- MUST NOT proceed with missing upstream specifications
- MUST NOT silently ignore phase failures
- MUST NOT modify agent execution order to bypass dependencies
- MUST NOT bypass pre-run validation when user requested it

- SHOULD provide progress updates during long-running workflows
- SHOULD report estimated time remaining for multi-phase execution
- SHOULD suggest recovery actions when phases fail
- SHOULD document lessons learned for workflow optimization

### Orchestrator Tooling

**Extracted tool list:**
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

**Extracted table:**
| Agent | Purpose | Input | Output |
|-------|---------|-------|--------|
| `smaqit.orchestrator` | Coordinate workflow | Orchestrator prompt + all layer/implementation prompts | Orchestration report + workflow status |

### Validation - Completion Criteria

**Extracted checklists:**

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

### Validation - Quality Boundary

**Extracted directives:**
- Agents MUST stop iterating when:
  - All completion criteria are met, OR
  - A blocking issue prevents progress (flag and report), OR
  - Clarification is required from upstream (request and wait)

- Agents MUST NOT:
  - Iterate indefinitely without progress
  - Lower quality standards to force completion
  - Invent solutions to bypass blockers

### Validation - Failure Modes

**Extracted table:**
| Situation | Action |
|-----------|--------|
| Ambiguous input | Request clarification, do not guess |
| Conflicting requirements | Flag conflict, propose resolution options |
| Missing upstream spec | Stop, indicate which spec is needed |
| Impossible requirement | Report impossibility with rationale |

---

## From ARTIFACTS.md

### Requirement Identifiers - Format

**Extracted format:**
```
[LAYER_PREFIX]-[CONCEPT]-[NNN]
```

**Component table:**
| Component | Description | Examples |
|-----------|-------------|----------|
| `LAYER_PREFIX` | Three-letter layer code | `BUS`, `FUN`, `STK`, `INF`, `COV` |
| `CONCEPT` | Descriptive concept name | `LOGIN`, `AUTH`, `API-USER` |
| `NNN` | Sequential number (3 digits) | `001`, `002`, `015` |

**ID format examples:**
| Layer | Requirement ID Format | Description Pattern |
|-------|----------------------|---------------------|
| Business | `BUS-[CONCEPT]-001` | [Use case or actor goal description] |
| Functional | `FUN-[CONCEPT]-001` | [Behavior or data model requirement] |
| Stack | `STK-[CONCEPT]-001` | [Technology choice or tool requirement] |
| Infrastructure | `INF-[CONCEPT]-001` | [Deployment or scaling requirement] |
| Coverage | `COV-[CONCEPT]-001` | Test case for [upstream requirement ID] |

**Extracted rules:**
- IDs MUST be unique within the project
- IDs MUST NOT be reused after deletion (mark as deprecated instead)
- IDs MUST remain stable—never rename an ID, only deprecate and create new
- Related criteria SHOULD share the same `CONCEPT` segment

### Acceptance Criteria - Format

**Extracted format:**
```markdown
## Acceptance Criteria

- [ ] [ID]: [Criterion statement]
- [ ] [ID]: [Criterion statement]
```

**Testability table:**
| Property | Definition | Good Example | Bad Example |
|----------|------------|--------------|-------------|
| **Measurable** | Has quantifiable outcome | "Response time < 2 seconds" | "Response is fast" |
| **Observable** | Can be verified externally | "Error message is displayed" | "System handles error gracefully" |
| **Unambiguous** | Single interpretation | "User sees 'Invalid password' text" | "User understands the error" |

**Extracted directives:**
- Every criterion MUST be: measurable, observable, unambiguous

### Acceptance Criteria - Untestable Criteria

**Extracted format:**
```markdown
- [ ] BUS-UX-002: Dashboard feels modern and engaging *(untestable)*
  - **Flag**: Subjective criterion—cannot be automatically validated
  - **Proposal**: Define measurable proxies (animations, color palette, satisfaction score)
  - **Resolution**: Defer to manual UX review; exclude from automated coverage
```

**Extracted directives:**
- Untestable criteria MUST be flagged with `*(untestable)*` marker
- Untestable criteria MUST include a proposal for measurable alternatives or resolution
- Untestable criteria MUST NOT block spec completion

### Traceability - Reference Types

**Extracted table:**
| Type | Meaning | Use Case |
|------|---------|----------|
| **Prompt File** | Layer-specific prompt | Primary source for layer requirements |
| **Context** | Adjacent layer spec used for coherence | Ensures cross-layer coherence |

### Traceability - Context References

**Extracted table:**
| Reference Type | Meaning | Example |
|----------------|---------|---------|
| **Implements** | Feature spec with 1:1 mapping to upstream spec | Feature spec → Single upstream requirement |
| **Enables** | Foundation spec serving multiple upstream specs | Foundation spec → Multiple upstream requirements |
| **Foundation Reference** | Feature spec references foundation spec in same layer | Feature spec → Foundation spec for shared requirements |

**Cross-layer reference format:**
```markdown
## References

### Implements
<!-- Feature spec: direct 1:1 implementation -->
- [BUS-[CONCEPT]-NNN](../business/[filename].md) — Implements [use case description]

### Enables  
<!-- Foundation spec: serves multiple business cases -->
- [BUS-[CONCEPT]-NNN](../business/[filename].md) — Enables [use case description]
- [BUS-[CONCEPT]-NNN](../business/[filename].md) — Enables [use case description]
```

**Foundation reference format:**
```markdown
## References

### Foundation Reference
<!-- Same-layer reference: feature spec extends foundation spec -->
- [STK-[FOUNDATION-CONCEPT]](./base-stack.md) — Shared requirements referenced here

### Implements
- [FUN-[CONCEPT]-NNN](../functional/feature.md) — Implements feature functionality
```

**Foundation reference rules:**
- Use when a feature spec extends a foundation spec in the same layer
- Foundation specs contain shared requirements that multiple feature specs depend on
- Example: Feature spec "[STK-CLI]" references foundation spec "[STK-PYTHON-BASE]" for base Python 3.8+ and development environment requirements
- Prefer updating existing spec over creating new spec with foundation reference when concept is not distinct

**Foundation specs without mapping format:**
```markdown
## References

### Enables
<!-- ⚠️ FOUNDATION WITHOUT MAPPING -->
**Justification:** [Why this foundation is needed before Business specs exist]
```

**Extracted directives:**
- Every spec (except Business) MUST have a References section
- References MUST use relative paths within `specs/`
- Orphaned foundations (no references, no justification) should be flagged by Coverage

**Traceability matrix example:**
| Business | Functional | Stack | Infrastructure | Coverage |
|----------|------------|-------|----------------|----------|
| BUS-[CONCEPT]-001 | FUN-[CONCEPT]-001 | STK-[CONCEPT]-001 | — | COV-[CONCEPT]-001 |

### Coverage Translation - Rules

**Extracted format:**

Source (Functional spec):
```markdown
- [ ] FUN-[CONCEPT]-001: [Behavior description]
```

Coverage translation:
```gherkin
# COV-[CONCEPT]-001: Maps to FUN-[CONCEPT]-001
Feature: [Feature Name]
  Scenario: [Scenario description]
    Given [precondition]
    When [action]
    Then [expected outcome]
```

**Extracted rules:**
- Each testable criterion MUST map to at least one test case
- Coverage IDs MUST reference their source requirement ID
- Untestable criteria MUST be listed with justification for exclusion
- Spec coverage % = (tested criteria / total testable criteria) × 100

### File Organization - Naming

**Extracted table:**
| Good | Bad |
|------|-----|
| `login.md` — Login use case | `authentication.md` — Login, logout, password reset, MFA |
| `user-registration.md` — Registration flow | `users.md` — Registration, profile, settings, deletion |

**Naming conventions:**
- Use lowercase with hyphens: `user-login.md`, `api-authentication.md`
- Match the primary concept name
- Avoid generic names: `misc.md`, `other.md`, `notes.md`

**Directory structure:**
```
specs/
├── business/
├── functional/
├── stack/
├── infrastructure/
└── coverage/
```

### Specification State - Frontmatter Schema

**Extracted format:**
```yaml
---
id: [LAYER_PREFIX]-[CONCEPT]
status: draft | implemented | deployed | validated | failed | deprecated
created: [ISO8601_TIMESTAMP]
implemented: [ISO8601_TIMESTAMP]
deployed: [ISO8601_TIMESTAMP]
validated: [ISO8601_TIMESTAMP]
prompt_version: [GIT_COMMIT_HASH]
---
```

**Required Fields:**
- `id`: Unique spec identifier (format: `BUS-LOGIN`, `FUN-AUTH`, etc.)
- `status`: Current lifecycle state
- `created`: Timestamp when spec was generated
- `prompt_version`: Git commit hash of prompt file at spec generation time

**Optional Fields (set by implementation agents):**
- `implemented`: When Development agent completed code generation
- `deployed`: When Deployment agent completed deployment
- `validated`: When Validation agent verified acceptance criteria

**State transition table:**
| From State | To State | Triggered By | Agent |
|------------|----------|--------------|-------|
| (none) | `draft` | Spec generation | Specification agents |
| `draft` | `implemented` | Code generated, tests pass | Development agent |
| `draft` | `failed` | Code generation failed | Development agent |
| `implemented` | `deployed` | Deployment succeeded | Deployment agent |
| `implemented` | `failed` | Deployment failed | Deployment agent |
| `deployed` | `validated` | All tests passed | Validation agent |
| `deployed` | `failed` | Tests failed | Validation agent |
| Any | `deprecated` | Feature removed | Manual/Specification agents |

### Acceptance Criteria State - Checkboxes

**Extracted format:**
```markdown
## Acceptance Criteria

- [x] BUS-LOGIN-001: User can authenticate with valid credentials
- [x] BUS-LOGIN-002: Invalid credentials show error message
- [!] BUS-LOGIN-003: Password complexity enforced (FAILED: regex bug)
```

**Checkbox states:**
- `[ ]` = Not yet implemented/validated
- `[x]` = Satisfied (implementation complete or test passed)
- `[!]` = Failed, untestable, or not satisfied

### State Aggregation - CLI Status

**Extracted example:**
```
Develop: 18 implemented, 2 failed
Deploy: 15 deployed, 3 draft
Validate: 12 validated, 5 draft
```

**Extracted directive:**
- Implementation agents update individual spec frontmatter
- CLI reads all specs and calculates aggregate counts
- No centralized state file

### Implementation Artifacts - The Anchoring Principle

**Extracted principle statement:**
> "Implementations MUST comply with industry standards for their stack, while satisfying spec-defined behavior. Two compliant implementations may differ internally, but MUST be structurally recognizable and behaviorally equivalent."

### Implementation Artifacts - The Isolation Principle

**Extracted principle statement:**
> "Agents operate on references, never values. Secrets and credentials MUST remain outside the agent's context at all times—resolution happens in a trusted execution layer that returns only outcomes, never the sensitive data itself."

### Implementation Artifacts - Three Dimensions

**Extracted dimension rules:**

**Behavior (Invariant):**
- Defined by specifications, MUST be satisfied exactly
- No deviation permitted—behavior is the contract

**Structure (Consistent):**
- Follows industry standards for the chosen stack
- Implementations SHOULD be recognizable to practitioners

**Internals (Variable):**
- Variable names, helper functions, internal patterns
- May vary freely between implementations

### Implementation Artifacts - Traceability

**Extracted format:**
```csharp
/// <summary>
/// [Method description].
/// Implements: [REQ-ID-001], [REQ-ID-002]
/// </summary>
public async Task<Result> MethodName(Request request)
```

**Extracted rules:**
- Major components SHOULD reference the spec requirements they implement
- Traceability MUST be verifiable during validation phase

### Implementation Artifacts - Validation Requirements

**Extracted table:**
| Dimension | Verifiable? | How |
|-----------|-------------|-----|
| Behavior | MUST | Automated tests from Coverage specs |
| Structure | SHOULD | Static analysis, architectural tests |
| Internals | NOT REQUIRED | — |

### Implementation Artifacts by Phase - Develop

**Extracted outputs:**
- Source code, tests, configurations, build files
- README with build, test, and run instructions
- Development report in `.smaqit/reports/development-phase-report-YYYY-MM-DD.md` (build/test/run results)
- Spec frontmatter: `status: implemented`, `implemented: [ISO8601_TIMESTAMP]`
- Acceptance criteria checkboxes updated in Business, Functional, Stack specs: `[ ]` → `[x]` or `[!]`

**Extracted directives:**
- MUST satisfy all spec acceptance criteria
- MUST follow stack-specific standards

### Implementation Artifacts by Phase - Deploy

**Extracted outputs:**
- Infrastructure code (Terraform, etc.)
- Deployment manifests, environment configs
- Deployment report in `.smaqit/reports/deployment-phase-report-YYYY-MM-DD.md` with health status and endpoints
- Spec frontmatter: `status: deployed`, `deployed: [ISO8601_TIMESTAMP]`
- Acceptance criteria checkboxes updated in Infrastructure specs: `[ ]` → `[x]` or `[!]`

**Extracted directives:**
- MUST NOT hardcode secrets (Isolation Principle)

### Implementation Artifacts by Phase - Validate

**Extracted outputs:**
- Test results, coverage report in `.smaqit/reports/validation-phase-report-YYYY-MM-DD.md`, validation summary
- Spec frontmatter: `status: validated`, `validated: [ISO8601_TIMESTAMP]`

**Extracted directives:**
- MUST map results to Coverage spec test cases
- MUST include spec coverage percentage

**Frontmatter example:**
```yaml
---
id: BUS-LOGIN-001
status: validated
created: 2025-12-26T10:00:00Z
implemented: 2025-12-26T10:30:00Z
deployed: 2025-12-26T11:00:00Z
validated: 2025-12-26T11:30:00Z
prompt_version: abc123
---
```

**Extracted directive:**
- Agents use atomic writes (temp file + rename) to prevent corruption

**Validation report format:**
```markdown
# Validation Report

## Summary
- Specs Covered: 47/50 (94%)
- Tests Passed: 45/47 (96%)

## Coverage Gaps
| Requirement | Reason |
|-------------|--------|
| [REQ-ID] | Untestable: [reason] |

## Failures
| Test | Requirement | Result | Details |
|------|-------------|--------|---------|
| [TEST-ID] | [REQ-ID] | FAIL | [Failure description] |
```

---

## From LAYERS.md

### Layer Independence

**Extracted directives:**
- Each layer receives requirements from its prompt file: `.github/prompts/smaqit.[layer].prompt.md`
- Can be selected or swapped independently
- Must be coherent with adjacent layers
- Does not derive requirements from upstream layers

### Business Layer - Directives

**Extracted directives:**

**Business specs MUST:**
- Identify all actors and their goals
- Define measurable success metrics for each use case
- Include preconditions and postconditions
- Describe main and alternative flows in business terms

**Business specs MUST NOT:**
- Mention specific technologies, frameworks, or libraries
- Include implementation details or technical solutions
- Define data structures or API contracts
- Reference deployment or infrastructure concerns

**System Actor table:**
| Actor | Description | Goals |
|-------|-------------|-------|
| System | The application as a whole | [System-level properties stakeholders require] |

**Note:** System actor specs remain business-level (stakeholder-driven) and do not prescribe technical solutions.

### Functional Layer - Directives

**Extracted directives:**

**Functional specs MUST:**
- Define user flows that implement business use cases
- Specify data models with attributes and relationships
- Define API contracts (inputs, outputs, error conditions)
- Include state transitions where applicable
- Reference business specs for traceability using Implements (1:1 feature) or Enables (1:many foundation)
- Include justification when foundation spec has no Business references

**Functional specs MUST NOT:**
- Specify technology choices (languages, frameworks, databases)
- Include deployment or infrastructure concerns
- Define performance benchmarks (those belong in Infrastructure)
- Prescribe implementation patterns

**Foundation vs Feature Specs table:**
| Type | Purpose | Business Reference |
|------|---------|--------------------|
| **Feature specs** | Implement a specific business use case | 1:1 mapping (Implements) |
| **Foundation specs** | Enable multiple business use cases | 1:many mapping (Enables) |

**Foundation spec rules:**
- SHOULD reference all Business specs they enable
- MUST flag absence of Business references with justification

**Note:** Orphaned foundations (no Business references, no justification) indicate scope creep.

### Stack Layer - Directives

**Extracted directives:**

**Stack specs MUST:**
- Document technology choices with rationale
- Define language versions and framework versions
- Specify libraries and their purposes
- Include build tools and development environment setup
- Be consistent with Functional specs (validated at implementation)
- Reference Functional specs using Enables (foundation serving multiple) or direct reference (feature serving one)
- Include justification when foundation spec has no Functional references

**Stack specs MUST NOT:**
- Include code examples, implementation patterns, or architecture code blocks
- Define deployment topology or infrastructure
- Include compute, networking, or scaling decisions
- Specify cloud providers or hosting platforms
- Contradict functional requirements

**Foundation vs Feature Specs table:**
| Type | Purpose | Functional Reference |
|------|---------|--------------------|
| **Feature specs** | Technology choices for a specific feature | 1:1 mapping (Enables) |
| **Foundation specs** | Base technologies enabling multiple features | 1:many mapping (Enables) |

**Foundation spec rules:**
- SHOULD reference all Functional specs they enable
- MUST flag absence of Functional references with justification

**Note:** Orphaned foundations (no Functional references, no justification) indicate scope creep.

### Infrastructure Layer - Directives

**Extracted directives:**

**Infrastructure specs MUST:**
- Define compute resources (containers, serverless, VMs)
- Specify networking topology and security boundaries
- Include observability (logging, metrics, tracing)
- Define scaling policies and resource limits
- Specify secrets management approach
- Be consistent with Phase 1 specs regarding requirements and runtime constraints (validated at implementation)
- Reference Phase 1 specs using Enables (foundation serving multiple) or direct reference (feature serving one)
- Include justification when foundation spec has no Phase 1 references

**Infrastructure specs MUST NOT:**
- Redefine business logic or functional behaviors
- Override technology choices from Stack layer
- Include application code or configurations
- Define test cases (those belong in Coverage)

**Foundation vs Feature Specs table:**
| Type | Purpose | Phase 1 Reference |
|------|---------|--------------------|
| **Feature specs** | Infrastructure for a specific feature/component | 1:1 mapping (Enables) |
| **Foundation specs** | Base infrastructure enabling multiple features | 1:many mapping (Enables) |

**Foundation spec rules:**
- SHOULD reference all Phase 1 specs (Business, Functional, Stack) they enable
- MUST flag absence of Phase 1 references with justification

**Note:** Orphaned foundations (no Phase 1 references, no justification) indicate scope creep.

### Coverage Layer - Directives

**Extracted directives:**

**Coverage specs MUST:**
- Reference every acceptance criterion from upstream specs by ID
- Define a test case for each testable requirement
- Map: Requirement ID → Test Case → Expected Outcome
- Flag untestable requirements explicitly
- Include integration, E2E, and acceptance test definitions
- Report spec coverage (% of requirements with corresponding tests)

**Coverage specs MUST NOT:**
- Add acceptance criteria not present in upstream specs
- Skip upstream acceptance criteria without justification
- Modify or reinterpret upstream acceptance criteria
- Define unit tests (those are implementation details)

---

## From PHASES.md

### Develop Phase - Pre-Run Validation

**Extracted directives:**
- Before starting, Development agent validates all required prompt files are filled:
  - `.github/prompts/smaqit.business.prompt.md` has content
  - `.github/prompts/smaqit.functional.prompt.md` has content
  - `.github/prompts/smaqit.stack.prompt.md` has content
- If any prompt is empty or insufficient, agent halts and guides user: "Please fill [prompt file] with your [layer] requirements before starting development."

### Develop Phase - Workflow

**Extracted workflow steps:**
```
1. Business agent produces business specifications
2. Functional agent produces functional specifications
3. Stack agent produces stack specifications
4. Development agent:
   a. Consolidates specs (coherence check, conflict detection)
   b. Generates application code
   c. Generates unit tests
   d. Compiles/builds application
   e. Runs application in isolated environment
   f. Executes unit tests
   g. Verifies application works as specified
```

### Develop Phase - Completion Criteria

**Extracted checklist:**
- [ ] All three layer specs produced and complete (Business, Functional, Stack)
- [ ] All specs have `status: implemented` or higher
- [ ] Code generated and compiles without errors
- [ ] Unit tests pass
- [ ] Application runs successfully in isolated environment
- [ ] Behavior matches spec acceptance criteria
- [ ] README includes build, test, and run instructions
- [ ] Development report written to `.smaqit/reports/development-phase-report-YYYY-MM-DD.md`
- [ ] Spec frontmatter updated: `status: implemented`, `implemented: [ISO8601_TIMESTAMP]`
- [ ] Acceptance criteria checkboxes updated in Business, Functional, Stack specs: `[ ]` → `[x]` or `[!]`

### Deploy Phase - Pre-Run Validation

**Extracted directives:**
- Before starting, check `.github/prompts/smaqit.infrastructure.prompt.md` for content beyond template structure:
  - If empty or only contains comments: Halt with natural language guidance
  - Example guidance: "Please specify your target environment (cloud, on-premise, hybrid), hosting platform, and service topology requirements"
- If prompt has content, agents interpret free-style requirements and request clarification for ambiguities.

### Deploy Phase - User Input Required

**Extracted table:**
| Category | Purpose |
|----------|----------|
| Target environment | Where the system will run |
| Hosting platform | Provider or infrastructure type |
| Service topology | How the application is structured for deployment |
| Resource constraints | Compute, memory, storage limits |
| Scaling requirements | How the system should handle load |
| Geographic constraints | Location or data residency requirements |
| Budget constraints | Cost limits or optimization goals |
| Integration points | Existing systems to connect with |

### Deploy Phase - Workflow

**Extracted workflow steps:**
```
1. Infrastructure agent produces infrastructure specifications
2. Deployment agent:
   a. Consolidates specs (infrastructure + stack coherence)
   b. Generates Infrastructure as Code (configurations as references only, per Isolation Principle)
   c. Triggers trusted execution layer with environment parameter
   d. Receives outcome (success/failure, health status, endpoints)
   e. Verifies system health in target environment
```

### Deploy Phase - Trusted Execution Layer

**Extracted diagram:**
```
┌─────────────────────────────────────────────────────────────┐
│ Deployment Agent (no credentials in context)                │
│                                                             │
│  Generates: main.tf with ${secrets.AWS_ACCESS_KEY}          │
│  Calls: deploy(environment="staging")                       │
│                                                             │
│         ┌───────────────────────────────────────────┐       │
│         │ Trusted Execution Layer                   │       │
│         │ - Resolves ${secrets.X} from vault        │       │
│         │ - Runs: apply                             │       │
│         │ - Runs: health checks                     │       │
│         │ - Scrubs credentials from output          │       │
│         └───────────────────────────────────────────┘       │
│                                                             │
│  Receives: { status: "success", endpoint: "https://..." }   │
└─────────────────────────────────────────────────────────────┘
```

### Deploy Phase - Completion Criteria

**Extracted checklist:**
- [ ] Infrastructure specs produced and complete
- [ ] All infrastructure specs have `status: deployed` or higher
- [ ] IaC generated with reference-only secrets
- [ ] Deployment executed successfully
- [ ] Health checks pass
- [ ] System accessible at expected endpoints
- [ ] Deployment report written to `.smaqit/reports/deployment-phase-report-YYYY-MM-DD.md`
- [ ] Spec frontmatter updated: `status: deployed`, `deployed: [ISO8601_TIMESTAMP]`
- [ ] All referenced specs updated to `status: deployed` per Status Cascade principle (Business, Functional, Stack, Infrastructure)
- [ ] Acceptance criteria checkboxes updated in Infrastructure specs: `[ ]` → `[x]` or `[!]`

### Validate Phase - Pre-Run Validation

**Extracted directives:**
- Before starting, check `.github/prompts/smaqit.coverage.prompt.md` for content beyond template structure:
  - If empty or only contains comments: Halt with natural language guidance
  - Example guidance: "Please specify the test scenarios, validation criteria, and acceptance thresholds for your application"
- If prompt has content, agents interpret free-style requirements and request clarification for ambiguities.

### Validate Phase - Workflow

**Extracted workflow steps:**
```
1. Coverage agent:
   a. Reads all upstream specs (business, functional, stack, infrastructure)
   b. Enumerates all acceptance criteria by ID
   c. Produces test definitions (Gherkin format)
   d. Maps: Requirement ID → Test Case → Expected Outcome
   e. Flags untestable criteria

2. Validation agent:
   a. Executes tests against deployed system
   b. Collects pass/fail results per test case
   c. Calculates spec coverage percentage
   d. Produces validation report
```

### Validate Phase - Output

**Extracted format:**

Validation report in `.smaqit/reports/validation-phase-report-YYYY-MM-DD.md` containing:
- Spec coverage percentage
- Pass/fail status per requirement
- Unverified requirements with justification
- Failure details for failed tests

### Validate Phase - Completion Criteria

**Extracted checklist:**
- [ ] Coverage specs produced with all testable criteria mapped
- [ ] All coverage specs have `status: validated`
- [ ] Tests executed against deployed system
- [ ] Validation report written to `.smaqit/reports/validation-phase-report-YYYY-MM-DD.md`
- [ ] Spec coverage percentage calculated
- [ ] Untestable criteria documented with justification
- [ ] Spec frontmatter updated: `status: validated`, `validated: [ISO8601_TIMESTAMP]`

### Failure Handling - Retry Threshold

**Extracted table:**
| Phase | Default Retries | Rationale |
|-------|-----------------|-----------|
| Develop | 3 | Code/test fixes typically converge quickly |
| Deploy | 2 | Infrastructure issues often need investigation |
| Validate | 0 | Failures require human analysis |

### Failure Handling - Documentation

**Extracted directives:**

Each failure attempt MUST document:
- What was attempted
- What failed (error message, scrubbed if sensitive)
- What was changed before retry
- Final status after threshold exceeded

### Failure Handling - Escalation

**Extracted steps:**

When retry threshold is exceeded:
1. Agent stops iterating
2. Failure summary produced
3. Human review required to proceed or abort

### Spec Change Adaptation

**Extracted table:**
| Spec Changed | Required Re-runs |
|--------------|------------------|
| Business, Functional, Stack | Develop → Deploy → Validate |
| Infrastructure | Deploy → Validate |
| Coverage | Validate |

**Extracted note:** Coverage phase always re-runs when any upstream spec changes to ensure test coverage remains current.

### Acceptance Criteria Checkboxes - Responsibility

**Extracted table:**
| Phase | Agent | Updates Checkboxes In | Rationale |
|-------|-------|----------------------|-----------|
| Develop | Development | Business, Functional, Stack specs | Agent implements these requirements and confirms satisfaction |
| Deploy | Deployment | Infrastructure specs | Agent deploys to environment and confirms infrastructure requirements met |

**Checkbox states:**
- `[ ]` — Not yet implemented/validated
- `[x]` — Satisfied (implementation complete or test passed)
- `[!]` — Failed, untestable, or not satisfied

**Note:** Checkbox updates are implementation tracking, not specification modification. They reflect work done, not changes to requirements.

### Incremental Development - Determining Work

**Extracted table:**
| Mode | Command | Processes |
|------|---------|-----------|
| Incremental | `smaqit plan --phase=develop` | Only specs with `status: draft` or `status: failed` |
| Regeneration | `smaqit plan --phase=develop --regen` | All specs regardless of status |

### Incremental Development - Adding Features

**Extracted workflow:**
```
1. User adds requirements to prompt file
2. Spec agent generates new specs (status: draft)
3. Implementation agent runs `smaqit plan --phase=develop`
4. CLI returns only new draft specs (existing implemented specs skipped)
5. Agent processes returned paths
6. Tests validate new + existing functionality
```

### Incremental Development - Checking Status

**Extracted example:**
```bash
# View aggregate phase status
smaqit status

# Shows per-phase spec counts:
Develop: 18 implemented, 2 failed
Deploy: 15 deployed, 3 draft
Validate: 12 validated, 5 draft
```

**Extracted note:** CLI aggregates status by scanning all spec frontmatter. No centralized state file.

### Current Assumptions

**Extracted table:**
| Assumption | Status | Revision Trigger |
|------------|--------|------------------|
| Phases are strictly sequential | Active | Incremental deployment proves valuable |
| Validation failures require human decision | Active | Patterns emerge for automated routing |

---

## From SMAQIT.md

### Single Source of Truth - Extracted Directives

**Extracted directives:**
- Agents MUST NOT duplicate information from existing specs—use Foundation Reference for same-layer or Implements/Enables for upstream
- Agents SHOULD update existing specs when extending a concept, create new specs only for distinct concepts
- Agents SHOULD reference foundation specs for shared requirements using Foundation Reference section

---

## From TEMPLATES.md

### Placeholder Convention - Common Placeholders

**Extracted table:**
| Placeholder | Description | Example |
|-------------|-------------|---------|
| `[LAYER]` | Lowercase layer name | `business` |
| `[LAYER_NAME]` | Title case layer name | `Business` |
| `[LAYER_PREFIX]` | 3-letter layer code | `BUS` |
| `[PHASE]` | Lowercase phase name | `development` |
| `[CONCEPT]` | Concept name in requirement ID | `LOGIN` |
| `[NNN]` | Sequential number (3 digits) | `001` |

### Placeholder Convention - Agent Template Placeholders

**Shared placeholders:**
| Placeholder | Description |
|-------------|-------------|
| `[UPSTREAM_SPEC_PATHS]` | Input spec paths |
| `[USER_INPUT_DESCRIPTION]` | What user input is accepted |

**Specification agent placeholders:**
| Placeholder | Description |
|-------------|-------------|
| `[LAYER]` | Lowercase layer name (e.g., `business`) |
| `[LAYER_NAME]` | Title case layer name (e.g., `Business`) |
| `[LAYER_PREFIX]` | 3-letter layer code (e.g., `BUS`) |
| `[LAYER_SPECIFIC_RULES]` | MUST/MUST NOT from LAYERS.md |

**Implementation agent placeholders:**
| Placeholder | Description |
|-------------|-------------|
| `[PHASE]` | Lowercase phase name (e.g., `development`) |
| `[PHASE_NAME]` | Title case phase name (e.g., `Development`) |
| `[AGENT_NAME]` | Agent display name (e.g., `Development Agent`) |
| `[UPSTREAM_SPEC_LAYERS]` | Which specification layers this agent consumes (e.g., `Business, Functional, and Stack`) |
| `[OUTPUT_ARTIFACTS_SUMMARY]` | Brief description of what this agent produces (e.g., `a working, tested application`) |
| `[PHASE_SEQUENCE_NOTE]` | Phase position in workflow (e.g., `Phase 1 of 3`) |
| `[PHASE_SPEC_LAYERS]` | Which spec layers are generated in this phase |
| `[PHASE_SPEC_SUMMARY]` | Brief summary of specs in this phase (e.g., `business, functional, stack specs`) |
| `[PHASE_SPECIFIC_RULES]` | MUST/MUST NOT from PHASES.md |
| `[ROLE_DETAILS]` | Phase-specific role description |
| `[OUTPUT_ARTIFACTS]` | What artifacts are produced |
| `[OUTPUT_FORMAT]` | Format of output artifacts |
| `[ADDITIONAL_COMPLETION_CRITERIA]` | Phase-specific completion checks |

### Specification Templates - Location

**Extracted directory structure:**
```
templates/specs/
├── business.template.md
├── functional.template.md
├── stack.template.md
├── infrastructure.template.md
└── coverage.template.md
```

### Specification Templates - Required Sections

**Extracted table:**
| Section | Purpose |
|---------|---------|
| Frontmatter | YAML metadata with state tracking |
| Title | Concept name |
| References | Upstream spec links (except Business) |
| Scope | What's included and excluded |
| [Layer-specific content] | Varies by layer |
| Acceptance Criteria | Testable requirements with IDs |

### Specification Templates - Frontmatter Requirements

**Extracted format:**
```yaml
---
id: [LAYER_PREFIX]-[CONCEPT]
status: draft
created: [TIMESTAMP]
prompt_version: [GIT_HASH]
---
```

**Required frontmatter fields:**
- `id`: Spec identifier (e.g., `BUS-LOGIN`, `FUN-AUTH-FLOW`)
- `status`: Initial state is always `draft`
- `created`: ISO8601 timestamp when spec generated
- `prompt_version`: Git commit hash of prompt file at generation

**Optional frontmatter fields (added by implementation agents):**
- `implemented`: Timestamp when Development agent completed
- `deployed`: Timestamp when Deployment agent completed
- `validated`: Timestamp when Validation agent completed

**Extracted directive:**
- Specification agents MUST generate frontmatter with required fields
- Implementation agents update frontmatter as specs progress through phases

### Specification Templates - Compliance Rules

**Extracted directives:**

When producing specs from templates:
- Agents MUST use the template from `templates/specs/[LAYER].template.md`
- Agents MUST produce consistent output structure across all runs
- Agents MUST NOT add sections not defined in the template
- Agents MUST NOT omit required sections from the template
- Agents MUST NOT leave placeholder text in completed specs
- Agents MUST minimize variance in generated artifacts

### Specification Templates - Placeholder Handling

**Extracted directives:**
- All placeholders MUST be replaced with actual content
- If a section is not applicable, state "Not applicable: [reason]"
- Empty sections are not permitted

### Agent Templates - Location

**Extracted directory structure:**
```
templates/agents/
├── specification-agent.template.md
├── implementation-agent.template.md
└── orchestrator-agent.template.md
```

### Agent Templates - Required Sections

**Extracted table:**
| Section | Purpose |
|---------|---------|
| YAML Frontmatter | name, description, tools |
| Role | Agent's purpose, including core responsibilities |
| Framework Reference | Links to relevant framework files |
| Input | Upstream specs and user input |
| Output | Location, template, format |
| Directives | MUST/MUST NOT/SHOULD rules |
| Completion Criteria | Self-validation checklist |
| Failure Handling | Error response table |

### Agent Templates - Format

**Extracted format:**
```
---
name: smaqit.[layer]
description: [One-line description]
tools: ["read", "edit", "search"]
---

# [Layer] Agent

## Role
...

## Input
...

## Output
...
```

**Note:** The code fence above is for illustration only. Actual agent files start directly with the YAML frontmatter (`---`).

### Prompt Templates - Location

**Extracted directory structure:**
```
templates/prompts/
├── specification-prompt.template.md
├── implementation-prompt.template.md
└── orchestrator-prompt.template.md
```

### Prompt Templates - Required Sections

**Extracted table:**
| Section | Purpose |
|---------|---------|
| YAML Frontmatter | name, description, agent |
| Purpose | What this prompt captures |
| Requirements | Sub-sections with suggested structure |
| Comment Examples | `<!-- Example: ... -->` for guidance |

### Prompt Templates - Format

**Extracted format:**
```markdown
---
name: smaqit.[layer]
description: [One-line description]
agent: smaqit.[layer]
---

# [Layer] Prompt

[Brief explanation]

## Requirements

[Sub-sections with suggested structure]

<!-- Example: [Guidance showing format] -->

[User fills requirements here]
```

### Prompt Templates - Comment Convention

**Extracted example:**
```markdown
### Actors

<!-- Example: "Mario Fan - Users who love Nintendo's Mario franchise" -->

[User writes actual actors here]
```

**Critical extracted directive:**
- Agents MUST ignore HTML comments to prevent example requirements from contaminating generated specs

### Template Completeness

**Extracted checklist:**
- [ ] All required sections are present
- [ ] Placeholders are clearly marked with `[PLACEHOLDER]` format
- [ ] Section purposes are unambiguous
- [ ] Layer-specific rules from LAYERS.md are incorporated (for spec templates)
- [ ] Comment examples use `<!-- Example: ... -->` format (for prompt templates)

---

## From PROMPTS.md

### Prompt Structure - Location

**Extracted directory structure:**
```
project/
└── .github/
    └── prompts/
        ├── smaqit.business.prompt.md
        ├── smaqit.functional.prompt.md
        ├── smaqit.stack.prompt.md
        ├── smaqit.infrastructure.prompt.md
        ├── smaqit.coverage.prompt.md
        ├── smaqit.development.prompt.md
        ├── smaqit.deployment.prompt.md
        ├── smaqit.validation.prompt.md
        └── smaqit.orchestrate.prompt.md
```

### Agent Interaction - Reading Prompts

**Extracted workflow steps:**
1. **Locate prompt**: Agent finds corresponding prompt file (e.g., Business Agent reads `smaqit.business.prompt.md`)
2. **Ignore comments**: Agent strips all HTML comments before interpretation
3. **Parse requirements**: Agent interprets free-style content per layer expectations
4. **Validate sufficiency**: Agent checks if enough information provided

### Agent Interaction - Validation Pattern

**Extracted directives:**

**If prompt empty or insufficient:**
- Agent halts execution
- Agent suggests what's missing using natural language guidance
- Agent waits for user to fill prompt and re-invoke

**Note:** Agents guide users naturally, not with template references or error codes.

**If prompt is filled sufficiently:**
- Agent proceeds with spec generation
- Agent uses prompt content as authoritative input

### Specification Prompts

**Extracted table:**
| Prompt | Layer | Captures | Invokes |
|--------|-------|----------|---------|
| `smaqit.business.prompt.md` | Business | Use cases, actors, goals | Business Agent |
| `smaqit.functional.prompt.md` | Functional | Behaviors, data, contracts | Functional Agent |
| `smaqit.stack.prompt.md` | Stack | Technologies, tools, rationale | Stack Agent |
| `smaqit.infrastructure.prompt.md` | Infrastructure | Deployment, scaling, observability | Infrastructure Agent |
| `smaqit.coverage.prompt.md` | Coverage | Test scope, environment, thresholds | Coverage Agent |

### Implementation Prompts

**Extracted table:**
| Prompt | Phase | Captures | Invokes |
|--------|-------|----------|---------|
| `smaqit.development.prompt.md` | Development | Build options, output preferences | Development Agent |
| `smaqit.deployment.prompt.md` | Deployment | Deployment target, verification | Deployment Agent |
| `smaqit.validation.prompt.md` | Validation | Execution scope, failure handling | Validation Agent |

**Extracted note:** Implementation prompts collect minimal runtime parameters (watch mode, verbosity, skip flags). Agents handle orchestration, validation, and error handling.

### Orchestrator Prompt

**Extracted table:**
| Prompt | Captures | Invokes |
|--------|----------|----------|
| `smaqit.orchestrate.prompt.md` | Phase selection, pre-validation preferences, error handling | Orchestrator Agent |

**Extracted note:** Orchestrator prompt collects execution parameters (which phases to run, validation preferences, error handling strategy). Orchestrator agent executes the workflow logic.

---

## Summary

**Total directives extracted:** 200+ distinct directives across all L0 files

**Next steps for L1 compilation:**
1. Agent-L1 reads this extraction document
2. Agent-L1 reads L0 framework files (principles)
3. Agent-L1 compiles directives into L1 templates with placeholders
4. Agent-L1 ensures L1 templates are self-contained and traceable to L0 principles

**Validation:**
- L0 files now contain zero MUST/SHOULD/MUST NOT statements
- L0 files read as pure principles and philosophy
- All implementation details (paths, commands, formats) removed from principles
- Extracted directives preserved for L1 compilation
