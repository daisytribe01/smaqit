# Phases

Phases are the sequential stages of software development in smaqit. Each phase includes both specification generation and implementation execution. The recommended workflow is to complete each phase before moving to the next, though specifications can be generated ahead if needed.

## Overview

smaqit operates in three sequential phases:

| Phase | Name | Specification Artifacts | Implementation Artifacts |
|-------|------|------------------------|-------------------------|
| Phase 1 | Develop | Business, Functional, Stack | Code and supporting documentation |
| Phase 2 | Deploy | Infrastructure | Running system and operational signals |
| Phase 3 | Validate | Coverage | Validation report and observed results |

Each phase moves from specification through consolidation to execution and verification. Sequencing keeps learning tight: Deploy follows a completed Develop, and Validate follows a completed Deploy, while remaining open to future refinement as the framework evolves.

### Implementation Phase Principles

**Status Cascade:** Implementation touches ripple upstream. When an implementation agent relies on a specification, that dependency is treated as part of the phase outcome so lifecycle state mirrors reality.

## Phase Definitions

### Develop — Build a Working Application

The Develop phase transforms user requirements into a working, tested application running in an isolated environment.

Business, Functional, and Stack perspectives set intent, behavior, and technology. Development weaves them together, produces code and tests, and exercises the result in an isolated environment until the application behaves as specified.

---

### Deploy — Run in Target Environment

The Deploy phase transforms a working application into a running system in a target environment.

Infrastructure specifications describe where and how the system should run. Deployment aligns stack and infrastructure intent, relies on trusted execution for sensitive operations, and verifies health in the target environment so running systems reflect the designed expectations.

---

### Validate — Verify Spec Compliance

The Validate phase verifies that the deployed system satisfies all specification requirements.

Coverage specifications translate upstream acceptance criteria into verification intent. Validation exercises deployed systems against those definitions, reports coverage and gaps, and reflects how closely the system matches specified behavior.

---

## Phase Transitions

### Develop → Deploy

Develop hands off when a working application exists and behavior aligns with specifications, providing context for infrastructure coherence.

---

### Deploy → Validate

Deploy hands off when the system is live and healthy in the target environment, supplying endpoints and environment context for validation.

---

### Validate → Feedback

Validation produces feedback that guides the next move: proceed when results are satisfactory, or return to earlier phases when gaps or failures appear.

---

## Failure Handling

### Retry Threshold

Failures are handled iteratively but with boundaries. Attempts are documented, and human judgment re-enters when progress stalls or risks increase.

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

Checkboxes inside specifications mark observed outcomes—unsatisfied, satisfied, or blocked. They act as a lightweight audit trail of what each phase achieved without altering the underlying requirements.

---

## Incremental Development

smaqit supports incremental workflows where specs are added and implemented iteratively.

Each specification carries a visible state—draft, implemented, deployed, validated—so incremental work focuses on what changed while preserving history. Planning and regeneration are guided by that state rather than a separate registry.

---

## Current Assumptions

These assumptions are explicitly stated and subject to revision per [SMAQIT](SMAQIT.md):

| Assumption | Status | Revision Trigger |
|------------|--------|------------------|
| Phases are strictly sequential | Active | Incremental deployment proves valuable |
| Validation failures require human decision | Active | Patterns emerge for automated routing |
