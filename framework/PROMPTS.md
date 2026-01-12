# Prompts

Prompts are the user-facing interface for smaqit workflows. They capture requirements as input records and invoke agents to generate specifications.

**Key Principles:**

- **Prompts are input records** — Capture user requirements for reproducibility and auditability
- **Free-style natural language** — Users write in their own words, agents interpret
- **HTML comment examples** — `<!-- Example: ... -->` provide guidance without rigid enforcement
- **Single manifest per layer** — One prompt file accumulates all requirements for a layer (unlike specs which are one-per-concept)

**Structure:**

- YAML frontmatter: `name`, `description`, `agent`
- Requirement sections with layer-specific sub-sections
- `<!-- Example: ... -->` comments for guidance (agents ignore these)
- Free-style user content in natural language

**Prompt types:**

- **Layer prompts** (5) — Capture requirements for specification layers (business, functional, stack, infrastructure, coverage)
- **Phase prompts** (3) — Trigger phase implementation agents

## Prompts as Input Records

**Prompts are versioned input records capturing user requirements at each layer.**

Filled prompts should be committed to version control alongside specs. When requirements change, users edit prompt files and regenerate specs.

## Prompt Structure

### Location

Prompts are organized in a dedicated prompts directory within the project. This location enables slash command invocation for agent activation.

### Format

**YAML Frontmatter + Free-Style Content**

Prompts use GitHub Copilot prompt format with frontmatter specifying name, description, and agent. Prompt templates define the structure.

### Single Manifest per Layer

Unlike specifications (one file per concept), prompts are **single manifest files** that capture all requirements for a layer:

- **Business prompt**: All use cases, actors, goals for the project
- **Functional prompt**: All behaviors, data models, contracts for the project
- **Stack prompt**: All technology choices and rationale for the project

As projects evolve, users add requirements to existing prompts rather than creating new prompt files. This creates a consolidated input record for the entire project at each layer.

### Free-Style with Suggested Structure

Prompts are **natural language inputs**, not rigidly structured forms. Templates provide suggested structure (sections, sub-sections) but users write requirements in their own words. Agents interpret and request clarification if needed.

### Comment Convention for Examples

**Agents ignore HTML comments** (`<!-- -->`). Templates include examples wrapped in `<!-- Example: ... -->` comments for user guidance only.

## Agent Interaction

### Reading Prompts

Agents read prompt files at the start of execution:

1. **Locate prompt**: Agent finds its corresponding prompt file
2. **Filter comments**: Agent strips all HTML comments before interpretation
3. **Parse requirements**: Agent interprets free-style content per layer expectations
4. **Validate sufficiency**: Agent checks if enough information provided

### Validation Pattern

Agents apply **Fail-Fast on Ambiguity** when reading prompts:

**If prompt empty or insufficient:**
- Agent halts execution
- Agent suggests what's missing using natural language guidance
- Agent waits for user to fill prompt and re-invoke

Agents guide users naturally, not with template references or error codes.

**If prompt is filled sufficiently:**
- Agent proceeds with spec generation
- Agent uses prompt content as authoritative input

## Amendment Workflow

When requirements change, users edit prompts and regenerate specs. Prompts are the source, specs are derived.

## Prompt Types

### Specification Prompts (Layer Prompts)

Capture requirements for single specification layer:

| Prompt Type | Layer | Captures | Invokes |
|-------------|-------|----------|---------|
| Business Prompt | Business | Use cases, actors, goals | Business Agent |
| Functional Prompt | Functional | Behaviors, data, contracts | Functional Agent |
| Stack Prompt | Stack | Technologies, tools, rationale | Stack Agent |
| Infrastructure Prompt | Infrastructure | Deployment, scaling, observability | Infrastructure Agent |
| Coverage Prompt | Coverage | Test scope, environment, thresholds | Coverage Agent |

### Implementation Prompts

Trigger single implementation agent with optional execution parameters:

| Prompt Type | Phase | Captures | Invokes |
|-------------|-------|----------|---------|
| Development Prompt | Development | Build options, output preferences | Development Agent |
| Deployment Prompt | Deployment | Deployment target, verification | Deployment Agent |
| Validation Prompt | Validation | Execution scope, failure handling | Validation Agent |

Implementation prompts collect minimal runtime parameters (watch mode, verbosity, skip flags). Agents handle orchestration, validation, and error handling.

### Orchestrator Prompt

Coordinates full workflow from specifications through validation:

| Prompt Type | Captures | Invokes |
|-------------|----------|---------|
| Orchestrator Prompt | Phase selection, pre-validation preferences, error handling | Orchestrator Agent |

Orchestrator prompt collects execution parameters (which phases to run, validation preferences, error handling strategy). Orchestrator agent executes the workflow logic.

