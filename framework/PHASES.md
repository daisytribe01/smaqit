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

**Status cascade:** Implementation agents update all specifications they reference, not just specs from their target layer.

**Reference-based updates:** When an implementation agent processes work, it identifies target specs, references upstream specs for coherence/context, and updates frontmatter in all referenced specs to reflect phase completion.

**Rationale:** If an implementation agent reads a specification for implementation context, that specification has been implemented/deployed/validated in that phase. Status reflects reality for accurate lifecycle tracking.

## Phase Definitions

### Develop — Build a Working Application

**Purpose:** The Develop phase transforms user requirements into a working, tested application running in an isolated environment.

**Specification agents:** Business agent translates stakeholder goals into business specifications. Functional agent translates experience shape into functional specifications. Stack agent translates technology preferences into stack specifications.

**Implementation agent:** Development agent consolidates specifications, generates application code and tests, builds the application, runs it in isolation, executes tests, and verifies behavior matches specifications.

**Pre-run validation:** Before starting, the Development agent validates all required prompt files contain content. If any prompt is empty or insufficient, agent halts and guides user to fill the specific prompt with needed requirements.

**Workflow:** Business agent produces specifications. Functional agent produces specifications. Stack agent produces specifications. Development agent consolidates specs (coherence check, conflict detection), generates code and tests, builds, runs in isolated environment, executes tests, verifies specifications.

**Environment:** Implicit—local developer machine or agent runner.

**Output:** Working, tested application in isolated environment.

**Failure handling:** Iterate on code/test failures up to retry threshold. Document failure reasons at each attempt. Escalate to human review when threshold exceeded.

**Completion criteria:** All three layer specs produced and complete (Business, Functional, Stack). All specs have implemented status or higher. Code generated and compiles without errors. Unit tests pass. Application runs successfully in isolated environment. Behavior matches spec acceptance criteria. README includes build, test, and run instructions. Development report written. Spec frontmatter updated with implementation status and timestamp. Acceptance criteria checkboxes updated in Business, Functional, Stack specs indicating satisfaction or failure.

---

### Deploy — Run in Target Environment

**Purpose:** The Deploy phase transforms a working application into a running system in a target environment.

**Specification agent:** Infrastructure agent translates deployment requirements into infrastructure specifications using all Phase 1 specs as context.

**Implementation agent:** Deployment agent consolidates specifications, generates Infrastructure as Code with reference-only secrets per Isolation Principle, triggers trusted execution with environment parameter, receives outcome, and verifies system health.

**Pre-run validation:** Before starting, check infrastructure prompt for content beyond template structure. If empty or only containing comments, halt with natural language guidance. If content present, interpret free-style requirements and request clarification for ambiguities.

**User input categories:** Target environment, hosting platform, service topology, resource constraints, scaling requirements, geographic constraints, budget constraints, integration points.

**Workflow:** Infrastructure agent produces specifications. Deployment agent consolidates specs (infrastructure + stack coherence), generates Infrastructure as Code (configurations as references only), triggers trusted execution layer with environment parameter, receives outcome (success/failure, health status, endpoints), verifies system health.

**Trusted execution layer:** Deployment agent operates on credential references, never values. Actual deployment happens in a trusted execution layer that resolves credential references from vault, executes deployment, runs health checks, scrubs credentials from output, and returns status with endpoints.

**Environment:** User-specified target (dev/staging/prod).

**Output:** Running system in target environment.

**Failure handling:** Iterate on deployment failures up to retry threshold. Document failure reasons (scrubbed of sensitive data). Escalate to human review when threshold exceeded.

**Completion criteria:** Infrastructure specs produced and complete. All infrastructure specs have deployed status or higher. IaC generated with reference-only secrets. Deployment executed successfully. Health checks pass. System accessible at expected endpoints. Deployment report written. Spec frontmatter updated with deployment status and timestamp. All referenced specs updated to deployed status per Status Cascade principle (Business, Functional, Stack, Infrastructure). Acceptance criteria checkboxes updated in Infrastructure specs indicating satisfaction or failure.

---

### Validate — Verify Spec Compliance

**Purpose:** The Validate phase verifies that the deployed system satisfies all specification requirements.

**Specification agent:** Coverage agent reads all upstream specs (business, functional, stack, infrastructure), enumerates acceptance criteria by ID, produces test definitions, maps requirements to test cases to expected outcomes, and flags untestable criteria.

**Implementation agent:** Validation agent executes tests against deployed system, collects pass/fail results per test case, calculates spec coverage percentage, and produces validation report.

**Pre-run validation:** Before starting, check coverage prompt for content beyond template structure. If empty or only containing comments, halt with natural language guidance. If content present, interpret free-style requirements and request clarification for ambiguities.

**Workflow:** Coverage agent reads all upstream specs, enumerates all acceptance criteria by ID, produces test definitions in standard format, maps requirement IDs to test cases to expected outcomes, flags untestable criteria. Validation agent executes tests against deployed system, collects pass/fail results per test case, calculates spec coverage percentage, produces validation report.

**Environment:** Same target environment as Deploy phase.

**Output:** Validation report containing spec coverage percentage, pass/fail status per requirement, unverified requirements with justification, and failure details for failed tests.

**Failure handling:** Test failures do not trigger automatic retry. Human decides next action: return to Develop (code/spec issue), return to Deploy (environment issue), investigate further, or accept with known issues.

**Completion criteria:** Coverage specs produced with all testable criteria mapped. All coverage specs have validated status. Tests executed against deployed system. Validation report written. Spec coverage percentage calculated. Untestable criteria documented with justification. Spec frontmatter updated with validation status and timestamp.

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

Implementation agents iterate on failures up to a configurable threshold. Development phase defaults to 3 retries (code/test fixes typically converge quickly). Deployment phase defaults to 2 retries (infrastructure issues often need investigation). Validation phase defaults to 0 retries (failures require human analysis).

### Failure Documentation

Each failure attempt documents what was attempted, what failed (error message, scrubbed if sensitive), what was changed before retry, and final status after threshold exceeded.

### Escalation

When retry threshold is exceeded: agent stops iterating, failure summary produced, human review required to proceed or abort.

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

**Implementation tracking:** Each implementation agent updates checkboxes in the specs it processes as part of its self-validation process. Development agent updates Business, Functional, Stack specs (implements these requirements, confirms satisfaction). Deployment agent updates Infrastructure specs (deploys to environment, confirms infrastructure requirements met).

**Checkbox states:** Not yet implemented/validated, satisfied (implementation complete or test passed), failed/untestable/not satisfied.

**Self-validation principle:** Checkbox updates are part of the implementation agent's self-validation process, confirming that requirements were addressed during execution. This creates an audit trail showing which phase satisfied which requirements.

**Distinction:** Checkbox updates are implementation tracking, not specification modification. They reflect work done, not changes to requirements.

---

## Incremental Development

smaqit supports incremental workflows where specs are added and implemented iteratively.

**Spec state tracking:** Each spec carries state through phases via frontmatter: Draft → Implemented → Deployed → Validated.

**Determining work:** Implementation agents determine which specs to process, focusing on specs requiring processing (draft or failed status by default). Regeneration mode processes all specs regardless of status.

**Adding features:** User adds requirements to prompt file. Spec agent generates new specs (draft status). Implementation agent processes new draft specs (existing implemented specs skipped). Tests validate new plus existing functionality.

**Checking status:** View aggregate phase status showing per-phase spec counts (implemented, failed, draft for each phase). System aggregates status by scanning all spec frontmatter. No centralized state file.

---

## Current Assumptions

These assumptions are explicitly stated and subject to revision per [SMAQIT](SMAQIT.md):

| Assumption | Status | Revision Trigger |
|------------|--------|------------------|
| Phases are strictly sequential | Active | Incremental deployment proves valuable |
| Validation failures require human decision | Active | Patterns emerge for automated routing |
