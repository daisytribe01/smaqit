# Implementation

Implementations are the imperative artifacts produced by implementation agents. They satisfy spec-defined behavior while following industry standards for their stack. This document establishes the principles all implementation artifacts MUST follow.

## The Anchoring Principle

> "Implementations MUST comply with industry standards for their stack, while satisfying spec-defined behavior. Two compliant implementations may differ internally but MUST be structurally recognizable and behaviorally equivalent."
>
> — Claude Opus 4.5

## The Isolation Principle

> "Agents operate on references, never values. Secrets and credentials MUST remain outside the agent's context at all times—resolution happens in a trusted execution layer that returns only outcomes, never the sensitive data itself."
>
> — Claude Opus 4.5

## Three Dimensions of Implementation

Every implementation exists across three dimensions with different consistency requirements:

```
┌─────────────────────────────────────────────────────────────┐
│ BEHAVIOR (from Specs)                                       │
│ Invariant — MUST be identical across implementations        │
├─────────────────────────────────────────────────────────────┤
│ STRUCTURE (from Industry Standards)                         │
│ Consistent — SHOULD follow stack-specific best practices    │
├─────────────────────────────────────────────────────────────┤
│ INTERNALS (Implementation Freedom)                          │
│ Variable — MAY differ, no two implementations identical     │
└─────────────────────────────────────────────────────────────┘
```

### Behavior (Invariant)

Behavior is defined by specifications and MUST be satisfied exactly.

- Functional specs define what the system does
- Acceptance criteria define verifiable outcomes
- No deviation permitted—behavior is the contract

**Example:**
```
Spec says: "API returns JWT token on successful login"
Implementation MUST: Return JWT token on successful login
```

### Structure (Consistent)

Structure follows industry standards for the chosen stack.

- Stack specs define which technologies are used
- Industry standards define how those technologies are structured
- Implementations SHOULD be recognizable to practitioners of that stack

**Example:**
```
Stack says: ".NET 8 with Clean Architecture"
Implementation SHOULD: Separate Domain, Application, Infrastructure, API layers
```

### Internals (Variable)

Internals are implementation details that may vary freely.

- Variable names, helper functions, internal patterns
- Exact file organization within structural guidelines
- Performance optimizations, syntactic choices

**Example:**
```
Two .NET implementations may both satisfy the spec and follow Clean Architecture,
yet have different service class names, different LINQ expressions, different
null-handling patterns. This variance is expected and accepted.
```

## Traceability Requirements

All implementation artifacts MUST be traceable to their source specifications.

### Code Traceability

Implementation code SHOULD include references to specifications:

```csharp
/// <summary>
/// Authenticates user and returns JWT token.
/// Implements: FUN-AUTH-001, FUN-AUTH-002
/// </summary>
public async Task<AuthResult> Login(LoginRequest request)
{
    // Implementation...
}
```

### Traceability Rules

- Major components SHOULD reference the spec requirements they implement
- Traceability MAY be maintained via code comments, documentation, or metadata
- Traceability MUST be verifiable during validation phase

## Validation Requirements

Implementations MUST be verifiable against their specifications.

### What Must Be Verifiable

| Dimension | Verifiable? | How |
|-----------|-------------|-----|
| Behavior | MUST | Automated tests from Coverage specs |
| Structure | SHOULD | Static analysis, architectural tests |
| Internals | NOT REQUIRED | — |

### Validation Outputs

The Validation agent produces reports that verify:

- [ ] All spec acceptance criteria pass (behavior)
- [ ] Stack-specific standards are followed (structure)
- [ ] Spec coverage percentage (tested criteria / total testable criteria)
- [ ] List of unverified requirements with justification

## Stack-Specific Standards

IMPLEMENTATION.md defines the *principle* of following standards. Stack specs define *which standards* apply to a project.

### Standards Reference Pattern

```markdown
# In specs/stack/technology-choices.md (project-specific)

## Standards

### Backend (.NET 8)
- Architecture: Clean Architecture
- API Style: RESTful with OpenAPI
- Patterns: CQRS for complex domains, Repository for data access

### Infrastructure (Terraform)
- Structure: Module composition
- State: Remote state with locking
- Environments: Workspace-based separation

### Frontend (React 18)
- Architecture: Component composition
- State: React Query for server state, Zustand for client state
- Patterns: Hooks over classes, co-located styles
```

### When Standards Conflict with Specs

If industry standards conflict with spec requirements:

1. Spec requirements take precedence (behavior is invariant)
2. Flag the conflict in implementation notes
3. Document the deviation from standard with rationale

## Implementation Artifacts by Phase

### Develop Phase → Code

| Artifact | Description |
|----------|-------------|
| Source code | Application logic implementing functional specs |
| Configurations | App settings, environment configs |
| Build artifacts | Compiled outputs, packages |

**Code MUST:**
- Satisfy all referenced spec acceptance criteria
- Follow stack-specific standards from Stack specs
- Include traceability references to specs

**Code MUST NOT:**
- Implement features not defined in specifications
- Contradict functional requirements
- Deviate from Stack spec technology choices

### Deploy Phase → Infrastructure

| Artifact | Description |
|----------|-------------|
| Infrastructure code | Terraform, Pulumi, CloudFormation, etc. |
| Deployment manifests | Kubernetes manifests, Docker Compose, etc. |
| Environment configs | Secrets references, environment variables |

**Infrastructure MUST:**
- Satisfy Infrastructure spec requirements
- Follow infrastructure-as-code standards for chosen tools
- Enable observability as defined in specs

**Infrastructure MUST NOT:**
- Introduce components not specified
- Override security boundaries defined in specs
- Hardcode secrets or sensitive data

### Validate Phase → Reports

| Artifact | Description |
|----------|-------------|
| Test results | Pass/fail status for each test case |
| Coverage report | Spec coverage percentage and gaps |
| Validation summary | Overall compliance assessment |

**Reports MUST:**
- Map results to Coverage spec test cases
- Include spec coverage percentage
- List unverified requirements with justification
- Provide actionable feedback for failures

**Report Format:**
```markdown
# Validation Report

## Summary
- Specs Covered: 47/50 (94%)
- Tests Passed: 45/47 (96%)
- Tests Failed: 2

## Coverage Gaps
| Requirement | Reason |
|-------------|--------|
| BUS-UX-002 | Untestable: subjective criterion |
| INF-DR-003 | Deferred: requires production environment |

## Failures
| Test | Requirement | Result | Details |
|------|-------------|--------|---------|
| COV-AUTH-005 | FUN-AUTH-003 | FAIL | Token expiration is 48h, spec requires 24h |
```

## Completeness Conditions

An implementation is complete when:

- [ ] All referenced spec acceptance criteria are satisfied
- [ ] Stack-specific standards are followed
- [ ] Traceability to specs is documented
- [ ] No unspecified features were added
- [ ] Validation can verify behavior against specs

## See Also

- [SMAQIT](SMAQIT.md) — Framework overview and principles
- [AGENTS](AGENTS.md) — Agent definitions and behaviors
- [LAYERS](LAYERS.md) — Layer definitions and dependencies
- [PHASES](PHASES.md) — Phase workflows and transitions
- [SPECIFICATIONS](SPECIFICATIONS.md) — Specification artifacts
