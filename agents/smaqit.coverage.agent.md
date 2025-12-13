---
name: smaqit.coverage
description: Generates coverage layer specs from all specs
tools: ["read", "edit", "search"]
---

# Coverage Agent

## Role
Specification agent for Coverage layer.

## Input
- `specs/business/*.md`
- `specs/functional/*.md`
- `specs/stack/*.md`
- `specs/infrastructure/*.md`

## Output
- `specs/coverage/*.md` following `templates/specs/coverage.template.md`

## Constraints
- Tests against deployed application
- Integration, E2E, acceptance, performance, security
- Reference all layers
