 # Specifications

Specifications are the source of truth in smaqit. They are declarative, stating what must be true, whereas implementations are imperative, stating how to make it true. This document establishes the rules all specification documents MUST follow.

## Purpose

Specifications serve as contracts between layers:

- **Upstream → Downstream**: Each spec informs the next layer's work
- **Spec → Implementation**: Implementation agents consume specs as requirements, not suggestions
- **Spec → Validation**: Coverage specs translate requirements into verifiable tests

A specification is complete when another agent (or human) can implement or validate against it without requiring additional context.

## Requirement Identifiers

Every acceptance criterion MUST have a unique identifier for traceability.

### Format

```
{LAYER}-{CONCEPT}-{NUMBER}
```

| Component | Description | Examples |
|-----------|-------------|----------|
| `LAYER` | Three-letter layer prefix | `BUS`, `FUN`, `STK`, `INF`, `COV` |
| `CONCEPT` | Descriptive concept name | `LOGIN`, `AUTH`, `API-USER`, `DB-CONN` |
| `NUMBER` | Sequential number (3 digits) | `001`, `002`, `015` |

### Examples

| Layer | Requirement ID | Description |
|-------|----------------|-------------|
| Business | `BUS-LOGIN-001` | User can authenticate with valid credentials |
| Functional | `FUN-AUTH-TOKEN-001` | JWT token expires after 24 hours |
| Stack | `STK-FRAMEWORK-001` | Use React 18+ for frontend |
| Infrastructure | `INF-SCALING-001` | Auto-scale at 80% CPU threshold |
| Coverage | `COV-LOGIN-001` | Test case for BUS-LOGIN-001 |

### Rules

- IDs MUST be unique within the project
- IDs MUST NOT be reused after deletion (mark as deprecated instead)
- IDs MUST remain stable—never rename an ID, only deprecate and create new
- Related criteria SHOULD share the same `CONCEPT` segment

## Acceptance Criteria

Acceptance criteria define testable conditions that must be satisfied. They are written as checklists in specification layers, then translated to executable format in Coverage.

### Format

```markdown
## Acceptance Criteria

- [ ] {ID}: {Criterion statement}
- [ ] {ID}: {Criterion statement}
```

### Examples

```markdown
## Acceptance Criteria

- [ ] BUS-LOGIN-001: User can authenticate with valid email and password
- [ ] BUS-LOGIN-002: User receives error message for invalid credentials
- [ ] BUS-LOGIN-003: User account locks after 5 failed attempts
```

### Testability Requirements

Every criterion MUST be:

| Property | Definition | Good Example | Bad Example |
|----------|------------|--------------|-------------|
| **Measurable** | Has quantifiable outcome | "Response time < 2 seconds" | "Response is fast" |
| **Observable** | Can be verified externally | "Error message is displayed" | "System handles error gracefully" |
| **Unambiguous** | Single interpretation | "User sees 'Invalid password' text" | "User understands the error" |

### Untestable Criteria

Some requirements cannot be automatically validated. These MUST be flagged explicitly:

```markdown
## Acceptance Criteria

- [ ] BUS-UX-001: Dashboard loads within 2 seconds *(testable)*
- [ ] BUS-UX-002: Dashboard feels modern and engaging *(untestable)*
  - **Flag**: Subjective criterion—cannot be automatically validated
  - **Proposal**: Define measurable proxies:
    - Animation transitions present on state changes
    - Color palette matches design system
    - User satisfaction score > 4/5 in usability testing
  - **Resolution**: Defer to manual UX review; exclude from automated coverage
```

Untestable criteria:
- MUST be flagged with `*(untestable)*` marker
- MUST include a proposal for measurable alternatives or resolution
- MUST NOT block spec completion
- SHOULD be minimized through refinement

## Traceability

Specifications MUST reference their upstream sources explicitly.

### Reference Format

```markdown
## References

- [BUS-LOGIN](../business/login.md) — Originating use case
- [FUN-AUTH-TOKEN-001](../functional/authentication.md#fun-auth-token-001) — Token requirement
```

### Rules

- Every spec (except Business) MUST have a References section
- References MUST use relative paths within `.smaqit/specs/`
- References MUST point to existing, accessible documents
- Anchor links (`#requirement-id`) SHOULD be used for criterion-level references

### Traceability Matrix

For complex projects, maintain traceability across layers:

| Business | Functional | Stack | Infrastructure | Coverage |
|----------|------------|-------|----------------|----------|
| BUS-LOGIN-001 | FUN-AUTH-001 | STK-JWT-001 | — | COV-LOGIN-001 |
| BUS-LOGIN-002 | FUN-AUTH-002 | — | — | COV-LOGIN-002 |

## Template Compliance

Specifications MUST follow their layer template exactly.

### Rules

- Agents MUST use the template from `.smaqit/templates/{layer}.template.md`
- Agents MUST NOT add sections not defined in the template
- Agents MUST NOT omit required sections from the template
- Agents MUST NOT leave placeholder text in completed specs

### Placeholder Handling

Templates may contain placeholder text like `{CONCEPT_NAME}` or `[describe here]`:

- All placeholders MUST be replaced with actual content
- If a section is not applicable, state "Not applicable: {reason}"
- Empty sections are not permitted

## Completeness Conditions

A specification is complete when:

- [ ] All template sections are filled (no placeholders remain)
- [ ] All acceptance criteria have unique IDs
- [ ] All acceptance criteria are testable (or flagged as untestable with proposals)
- [ ] All upstream references are valid and accessible
- [ ] Scope boundaries are explicitly stated (what's included and excluded)
- [ ] No implementation details are present (except in Stack layer for technology choices)

## Coverage Translation

The Coverage layer translates checklist-based acceptance criteria into executable test definitions using Gherkin format.

### Translation Example

**Source (Functional spec):**
```markdown
## Acceptance Criteria

- [ ] FUN-AUTH-001: User receives JWT token upon successful login
- [ ] FUN-AUTH-002: Token contains user ID and expiration timestamp
```

**Coverage translation:**
```gherkin
# COV-AUTH-001: Maps to FUN-AUTH-001
Feature: Authentication Token
  Scenario: Successful login returns JWT token
    Given a registered user with valid credentials
    When the user submits login request
    Then the response contains a JWT token
    And the response status is 200

# COV-AUTH-002: Maps to FUN-AUTH-002  
  Scenario: Token contains required claims
    Given a valid JWT token from login
    When the token payload is decoded
    Then the payload contains "user_id" claim
    And the payload contains "exp" claim
```

### Coverage Rules

- Each testable criterion MUST map to at least one test case
- Coverage IDs MUST reference their source requirement ID
- Untestable criteria MUST be listed with justification for exclusion
- Spec coverage percentage = (tested criteria / total testable criteria) × 100

## File Organization

### One Spec Per Concept

Each specification file should address a single cohesive concept:

| Good | Bad |
|------|-----|
| `login.md` — Login use case | `authentication.md` — Login, logout, password reset, MFA |
| `user-registration.md` — Registration flow | `users.md` — Registration, profile, settings, deletion |

### Naming Conventions

- Use lowercase with hyphens: `user-login.md`, `api-authentication.md`
- Match the primary concept name
- Avoid generic names: `misc.md`, `other.md`, `notes.md`

### Directory Structure

```
.smaqit/specs/
├── business/
│   ├── user-login.md
│   └── user-registration.md
├── functional/
│   ├── authentication.md
│   └── user-profile.md
├── stack/
│   └── technology-choices.md
├── infrastructure/
│   └── deployment.md
└── coverage/
    ├── authentication-tests.md
    └── user-flow-tests.md
```

## See Also

- [SMAQIT](SMAQIT.md) — Framework overview and principles
- [AGENTS](AGENTS.md) — Agent definitions and behaviors
- [LAYERS](LAYERS.md) — Layer definitions and dependencies
- [PHASES](PHASES.md) — Phase workflows and transitions
- [IMPLEMENTATIONS](IMPLEMENTATIONS.md) — Implementation artifacts
