---
name: smaqit.infrastructure
description: Create infrastructure layer specifications from deployment requirements
agent: smaqit.infrastructure
---

# Infrastructure Prompt

This prompt captures infrastructure and deployment requirements for your project. These requirements will be used to generate infrastructure specifications.

## Requirements

### Target Environment
[Where will this system run?]

<!-- Example: "Local development machine - no cloud deployment" -->
<!-- Example: "Developer laptop (Windows/macOS/Linux)" -->

### Hosting Platform
[What platform or infrastructure?]

<!-- Example: "Local execution - no hosting required" -->

### Service Topology
[How is the system structured?]

<!-- Example: "Single-file Python script - no services" -->

### Resource Constraints
[What are the compute/memory/storage limits?]

<!-- Example: "Minimal - runs on any modern system with Python" -->

### Scaling Requirements
[How should the system handle load?]

<!-- Example: "No scaling needed - single execution per run" -->

### Geographic Constraints
[Are there location or data residency requirements?]

<!-- Example: "No constraints - runs locally" -->

### Budget Constraints
[What are the cost limits?]

<!-- Example: "$0 - no infrastructure costs" -->

### Integration Points
[What existing systems need to connect?]

<!-- Example: "None - standalone application" -->

## Addendum

Iterative refinements and amendments (auto-generated). Agents append refinement instructions here when users request modifications to existing specifications.

Format: `[YYYY-MM-DD HH:MM] [refinement instruction]`

<!-- Example: "[2025-12-28 14:30] Move from local execution to AWS Lambda" -->
<!-- Example: "[2025-12-28 15:45] Add CloudWatch logging for production monitoring" -->
