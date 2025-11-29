# smaqit Framework

**Version**: 0.1.0

## Layers

| Layer | Purpose |
|-------|---------|
| Business | Use cases |
| Functional | Behaviors, contracts, flows |
| Stack | Languages, frameworks, libraries |
| Infrastructure | Compute, networking, observability |
| Coverage | Integration, E2E, acceptance testing |

## Phases

| Phase | Spec Layers | Implementation Agent |
|-------|-------------|----------------------|
| Develop | Business → Functional → Stack | Development |
| Deploy | Infrastructure | Deployment |
| Validate | Coverage | Validation |

## Agents

### Specification Agents

| Agent | Layer | Input | Output |
|-------|-------|-------|--------|
| Business | Business | User description | `specs/business/*.md` |
| Functional | Functional | Business specs | `specs/functional/*.md` |
| Stack | Stack | Functional specs | `specs/stack/*.md` |
| Infrastructure | Infrastructure | Stack specs | `specs/infrastructure/*.md` |
| Coverage | Coverage | All specs | `specs/coverage/*.md` |

### Implementation Agents

| Agent | Phase | Input | Output |
|-------|-------|-------|--------|
| Development | Develop | Develop specs | Code |
| Deployment | Deploy | Code + Infra specs | Running system |
| Validation | Validate | Deployed app + Coverage specs | Validation report |

## Usage Rules

### Core Principle

**Specs before code.** Never write implementation without a corresponding specification.

### Layer Order

Always work through layers in order:

1. Business → 2. Functional → 3. Stack → 4. Infrastructure → 5. Coverage

### File Locations (in smaqit-enabled projects)

- Specs: `.smaqit/specs/{layer}/`
- Templates: `.smaqit/templates/`
- Framework: `.smaqit/SMAQIT.md`
- Agents: `.github/agents/`

### When Developing a Feature

1. Check if business spec exists in `.smaqit/specs/business/`
2. If not, write business spec first using `business.template.md`
3. Then functional spec using `functional.template.md`
4. Then stack spec using `stack.template.md`
5. Only then write code based on the output of the previous layers

### When Deploying

1. Check if infrastructure spec exists in `.smaqit/specs/infrastructure/`
2. If not, write infrastructure spec first using `infrastructure.template.md`
3. Only then deploy

### When Validating

1. Check if coverage spec exists in `.smaqit/specs/coverage/`
2. If not, write coverage spec first using `coverage.template.md`
3. Tests run against deployed app, not local

### Template Compliance

When writing specs, use the exact structure from `.smaqit/templates/{layer}.template.md`. Do not add or remove sections.
