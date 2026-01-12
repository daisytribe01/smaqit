# Phases

Phases are the sequential stages of software development in smaqit. Each phase includes both specification generation and implementation execution. The recommended workflow is to complete each phase before moving to the next, though specifications can be generated ahead if needed.

## Overview

smaqit operates in three sequential phases:

| Phase | Name | Specification Artifacts | Implementation Artifacts |
|-------|------|------------------------|-------------------------|
| Phase 1 | Develop | Business, Functional, Stack | Code, README, Development report in `.smaqit/reports/` |
| Phase 2 | Deploy | Infrastructure | Running system, Deployment report in `.smaqit/reports/` |
| Phase 3 | Validate | Coverage | Validation report in `.smaqit/reports/` |

Each phase:
1. **Specifies** — One or more specification agents produce layer specs
2. **Consolidates** — Implementation agent verifies cross-layer coherence
3. **Implements** — Implementation agent produces and executes artifacts
4. **Verifies** — Implementation agent confirms success before phase completion

Phases are strictly sequential. Deploy cannot begin until Develop completes. Validate cannot begin until Deploy completes. This constraint is subject to revision based on real-world usage (see [SMAQIT](SMAQIT.md)).

### Implementation Phase Principles

**Status Cascade:** Implementation agents update all specifications they reference, not just specs from their target layer.

When an implementation agent processes work, it identifies target specs, references upstream specs for coherence and context, and updates frontmatter in all referenced specs to reflect phase completion.

**Rationale:** If an implementation agent reads a specification for implementation context, that specification has been implemented/deployed/validated in that phase. Status must reflect reality for accurate lifecycle tracking.

## Phase Definitions

### Develop — Build a Working Application

The Develop phase transforms user requirements into a working, tested application running in an isolated environment.

**Specification Agents:**

| Agent | Layer | User Input | Context |
|-------|-------|------------|---------|
| Business Agent | Business | Stakeholder goals | None |
| Functional Agent | Functional | Experience shape | Business specs |
| Stack Agent | Stack | Technology preferences | Business and Functional specs |

**Implementation Agent:** Development Agent

**Pre-Run Validation:**

Before starting, the Development agent validates all required prompt files for Business, Functional, and Stack layers contain content. If any prompt is empty or insufficient, the agent halts and guides the user with natural language prompts to fill requirements.

**Workflow:**
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

**Environment:** Implicit — local developer machine or agent runner (e.g., GitHub Actions runner)

**Output:** Working, tested application in isolated environment

**Failure Handling:**
- Iterate on code/test failures up to retry threshold
- Document failure reasons at each attempt
- Escalate to human review when threshold exceeded

**Completion Criteria:**
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

---

### Deploy — Run in Target Environment

The Deploy phase transforms a working application into a running system in a target environment.

**Specification Agent:**

| Agent | Layer | User Input | Context |
|-------|-------|------------|---------|
| Infrastructure Agent | Infrastructure | Deployment requirements | Phase 1 specs |

**Implementation Agent:** Deployment Agent

**Pre-Run Validation:**

Before starting, the Deployment agent checks that the Infrastructure prompt file contains content beyond template structure. If empty or only contains comments, it halts with natural language guidance requesting target environment, hosting platform, and service topology requirements. If prompt has content, agents interpret free-style requirements and request clarification for ambiguities.

**User Input Required:**

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

**Workflow:**
```
1. Infrastructure agent produces infrastructure specifications
2. Deployment agent:
   a. Consolidates specs (infrastructure + stack coherence)
   b. Generates Infrastructure as Code (configurations as references only, per Isolation Principle)
   c. Triggers trusted execution layer with environment parameter
   d. Receives outcome (success/failure, health status, endpoints)
   e. Verifies system health in target environment
```

**Trusted Execution Layer:**
The deployment agent operates on credential references, never values. Actual deployment happens in a trusted execution layer:

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

See [ARTIFACTS](ARTIFACTS.md) for the Isolation Principle.

**Environment:** User-specified target (dev/staging/prod)

**Output:** Running system in target environment

**Failure Handling:**
- Iterate on deployment failures up to retry threshold
- Document failure reasons (scrubbed of sensitive data)
- Escalate to human review when threshold exceeded

**Completion Criteria:**
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

---

### Validate — Verify Spec Compliance

The Validate phase verifies that the deployed system satisfies all specification requirements.

**Specification Agent:**

| Agent | Layer | Input |
|-------|-------|-------|
| Coverage Agent | Coverage | All layer specs |

**Implementation Agent:** Validation Agent

**Pre-Run Validation:**

Before starting, the Validation agent checks that the Coverage prompt file contains content beyond template structure. If empty or only contains comments, it halts with natural language guidance requesting test scenarios, validation criteria, and acceptance thresholds. If prompt has content, agents interpret free-style requirements and request clarification for ambiguities.

**Workflow:**

The Coverage agent reads all upstream specs (business, functional, stack, infrastructure), enumerates all acceptance criteria by ID, produces test definitions, maps requirements to test cases to expected outcomes, and flags untestable criteria.

The Validation agent executes tests against the deployed system, collects pass/fail results per test case, calculates spec coverage percentage, and produces the validation report.

**Environment:** Same target environment as Deploy phase

**Output:** Validation report containing spec coverage percentage, pass/fail status per requirement, unverified requirements with justification, and failure details for failed tests.

**Failure Handling:**
- Test failures do NOT trigger automatic retry
- Human decides next action:
  - Return to Develop (code/spec issue)
  - Return to Deploy (environment issue)
  - Investigate further
  - Accept with known issues

**Completion Criteria:**
- [ ] Coverage specs produced with all testable criteria mapped
- [ ] All coverage specs have `status: validated`
- [ ] Tests executed against deployed system
- [ ] Validation report written to `.smaqit/reports/validation-phase-report-YYYY-MM-DD.md`
- [ ] Spec coverage percentage calculated
- [ ] Untestable criteria documented with justification
- [ ] Spec frontmatter updated: `status: validated`, `validated: [ISO8601_TIMESTAMP]`

---

## Phase Transitions

### Develop → Deploy

**Trigger:** Develop phase completion criteria met

**Prerequisites:**
- Application compiles and runs
- Unit tests pass
- Behavior verified in isolated environment

**Handoff:**
- Working application code
- Phase 1 specs (context for Infrastructure coherence)

---

### Deploy → Validate

**Trigger:** Deploy phase completion criteria met

**Prerequisites:**
- System deployed to target environment
- Health checks pass
- Endpoints accessible

**Handoff:**
- Running system endpoint(s)
- Target environment identifier

---

### Validate → Feedback

**Trigger:** Validation report generated

**Outcomes:**

| Result | Action |
|--------|--------|
| All tests pass | Cycle complete ✓ |
| Tests fail | Human decides: Develop, Deploy, or investigate |
| Low coverage | Review Coverage specs for gaps |

**Note:** Automated feedback routing is deferred to future versions. Currently, validation failures require human decision.

---

## Failure Handling

### Retry Threshold

Implementation agents iterate on failures up to a configurable threshold:

| Phase | Default Retries | Rationale |
|-------|-----------------|-----------|
| Develop | 3 | Code/test fixes typically converge quickly |
| Deploy | 2 | Infrastructure issues often need investigation |
| Validate | 0 | Failures require human analysis |

### Failure Documentation

Each failure attempt documents what was attempted, what failed (error message, scrubbed if sensitive), what was changed before retry, and final status after threshold exceeded.

### Escalation

When retry threshold is exceeded:
1. Agent stops iterating
2. Failure summary produced
3. Human review required to proceed or abort

---

## Spec Change Adaptation

When any layer spec changes, downstream phases must re-run:

| Spec Changed | Required Re-runs |
|--------------|------------------|
| Business, Functional, Stack | Develop → Deploy → Validate |
| Infrastructure | Deploy → Validate |
| Coverage | Validate |

Coverage phase always re-runs when any upstream spec changes to ensure test coverage remains current.

---

## Acceptance Criteria Checkboxes

Each implementation agent updates checkboxes in the specs it processes as part of its self-validation process.

**Checkbox Responsibility by Phase:**

| Phase | Agent | Updates Checkboxes In | Rationale |
|-------|-------|----------------------|-----------|
| Develop | Development | Business, Functional, Stack specs | Agent implements these requirements and confirms satisfaction |
| Deploy | Deployment | Infrastructure specs | Agent deploys to environment and confirms infrastructure requirements met |

**Checkbox States:**

- `[ ]` — Not yet implemented/validated
- `[x]` — Satisfied (implementation complete or test passed)
- `[!]` — Failed, untestable, or not satisfied

**Self-Validation Principle:**

Checkbox updates are part of the implementation agent's self-validation process, confirming that requirements were addressed during execution. This creates an audit trail showing which phase satisfied which requirements.

**Note:** Checkbox updates are implementation tracking, not specification modification. They reflect work done, not changes to requirements.

---

## Incremental Development

smaqit supports incremental workflows where specs are added and implemented iteratively.

**Spec State Tracking:**

Each spec carries state through phases via frontmatter:
- Draft → Implemented → Deployed → Validated

**Determining Work:**

Implementation agents determine which specs need processing. In incremental mode, only specs with draft or failed status are processed. In regeneration mode, all specs are processed regardless of status.

**Adding Features:**

When users add new requirements to prompt files, specification agents generate new specs with draft status. Implementation agents identify and process only the new draft specs, skipping existing implemented specs. Tests validate both new and existing functionality.

**Status Aggregation:**

CLI aggregates phase status by scanning all spec frontmatter, showing per-phase spec counts. No centralized state file is maintained.

---

## Current Assumptions

These assumptions are explicitly stated and subject to revision per [SMAQIT](SMAQIT.md):

| Assumption | Status | Revision Trigger |
|------------|--------|------------------|
| Phases are strictly sequential | Active | Incremental deployment proves valuable |
| Validation failures require human decision | Active | Patterns emerge for automated routing |
