# Prompts

Prompts are the user-facing interface for smaqit workflows. They capture requirements as input records and invoke agents to generate specifications.

## Key Principles

**Input records:** Prompts capture user requirements for reproducibility and auditability.

**Natural language:** Free-style expression in users' own words with agents interpreting content.

**Example guidance:** HTML comment examples provide guidance without rigid enforcement.

**Single manifest:** One prompt file accumulates all requirements for a layer (unlike specs which are one-per-concept).

**Structure:** YAML frontmatter (name, description, agent) plus requirement sections with layer-specific sub-sections plus comment examples for guidance plus free-style user content in natural language.

**Prompt varieties:** Layer prompts (5) capture requirements for specification layers (business, functional, stack, infrastructure, coverage). Phase prompts (3) trigger phase implementation agents.

## Prompts as Input Records

**Versioned requirements:** Prompts are versioned input records capturing user requirements at each layer. Filled prompts should be committed to version control alongside specs. When requirements change, users edit prompt files and regenerate specs.

## Prompt Structure

**Natural language with suggested organization:** Prompts use GitHub Copilot prompt format with frontmatter specifying name, description, and agent. Free-style content follows suggested structure provided by templates.

**Single manifest consolidation:** Unlike specifications (one file per concept), prompts are single manifest files capturing all requirements for a layer. Business prompt contains all use cases, actors, goals for the project. Functional prompt contains all behaviors, data models, contracts for the project. Stack prompt contains all technology choices and rationale for the project. As projects evolve, users add requirements to existing prompts rather than creating new prompt files. This creates a consolidated input record for the entire project at each layer.

**Flexible structure:** Prompts are natural language inputs, not rigidly structured forms. Templates provide suggested structure (sections, sub-sections) but users write requirements in their own words. Agents interpret and request clarification if needed.

**Example convention:** Agents ignore HTML comments. Templates include examples wrapped in comment tags for user guidance only.

## Agent Interaction

### Reading Prompts

**Prompt consumption:** Agents read prompt files at execution start, locating corresponding prompt files, stripping all HTML comments before interpretation, parsing free-style content per layer expectations, and validating sufficiency.

### Validation Pattern

**Sufficiency checking:** Agents apply fail-fast on ambiguity when reading prompts. If prompt empty or insufficient: agent halts execution, suggests what's missing using natural language guidance, waits for user to fill prompt and re-invoke. Agents guide users naturally, not with template references or error codes. If prompt filled sufficiently: agent proceeds with spec generation, using prompt content as authoritative input.

## Amendment Workflow

**Requirement changes:** When requirements change, users edit prompts and regenerate specs. Prompts are the source, specs are derived. Agents always read from prompt files.

## Prompt Types

### Specification Prompts (Layer Prompts)

**Layer-specific input:** Each prompt captures requirements for single specification layer, covering use cases and actors and goals (business), behaviors and data and contracts (functional), technologies and tools and rationale (stack), deployment and scaling and observability (infrastructure), test scope and environment and thresholds (coverage).

### Implementation Prompts

**Phase execution parameters:** Trigger single implementation agent with optional execution parameters covering build options and output preferences (development), deployment target and verification (deployment), execution scope and failure handling (validation). Implementation prompts collect minimal runtime parameters (watch mode, verbosity, skip flags). Agents handle orchestration, validation, and error handling.

### Orchestrator Prompt

**Workflow coordination:** Coordinates full workflow from specifications through validation, capturing phase selection, pre-validation preferences, and error handling strategy. Orchestrator prompt collects execution parameters (which phases to run, validation preferences, error handling strategy). Orchestrator agent executes the workflow logic.

