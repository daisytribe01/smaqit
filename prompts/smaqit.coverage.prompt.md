---
name: smaqit.coverage
description: Create coverage layer specifications from verification preferences
agent: smaqit.coverage
---

# Coverage Prompt

This prompt captures verification strategy preferences for your project. The Coverage agent derives test requirements automatically from all upstream specifications (Business, Functional, Stack, Infrastructure).

## Verification Preferences

### Test Environment
[Where and how should tests run?]

<!-- Example: "GitHub Actions on push to main branch" -->
<!-- Example: "Local test suite with pytest" -->
<!-- Example: "Docker container with Node 20 runtime" -->

### Acceptance Thresholds
[What defines acceptable test coverage and results?]

<!-- Example: "100% of testable acceptance criteria must have test cases" -->
<!-- Example: "All integration tests must pass before deployment" -->
<!-- Example: "Performance tests within 10% of baseline" -->
