# Templates

Templates shape how agents express their outputs. They provide consistent structure for specifications, agent definitions, and prompts.

## Template Types

smaqit relies on three template families:

| Type | Purpose | Produces |
|------|---------|----------|
| **Specification templates** | Structure for specification documents | Layered specs |
| **Agent templates** | Structure for agent definitions | Agent manifests |
| **Prompt templates** | Structure for prompt files | Layer and phase prompts |

## Placeholder Convention

All templates use `[PLACEHOLDER]` format (brackets, SCREAMING_CASE) for customizable values.

### Common Placeholders

| Placeholder | Description | Example |
|-------------|-------------|---------|
| `[LAYER]` | Lowercase layer name | `business` |
| `[LAYER_NAME]` | Title case layer name | `Business` |
| `[LAYER_PREFIX]` | 3-letter layer code | `BUS` |
| `[PHASE]` | Lowercase phase name | `development` |
| `[CONCEPT]` | Concept name in requirement ID | `LOGIN` |
| `[NNN]` | Sequential number (3 digits) | `001` |

### Agent Template Placeholders

**Shared placeholders:**

| Placeholder | Description |
|-------------|-------------|
| `[UPSTREAM_SPEC_PATHS]` | Input spec paths |
| `[USER_INPUT_DESCRIPTION]` | What user input is accepted |

**Specification agent placeholders:**

| Placeholder | Description |
|-------------|-------------|
| `[LAYER]` | Lowercase layer name (e.g., `business`) |
| `[LAYER_NAME]` | Title case layer name (e.g., `Business`) |
| `[LAYER_PREFIX]` | 3-letter layer code (e.g., `BUS`) |
| `[LAYER_SPECIFIC_RULES]` | Layer guardrails derived from LAYERS.md |

**Implementation agent placeholders:**

| Placeholder | Description |
|-------------|-------------|
| `[PHASE]` | Lowercase phase name (e.g., `development`) |
| `[PHASE_NAME]` | Title case phase name (e.g., `Development`) |
| `[AGENT_NAME]` | Agent display name (e.g., `Development Agent`) |
| `[UPSTREAM_SPEC_LAYERS]` | Which specification layers this agent consumes (e.g., `Business, Functional, and Stack`) |
| `[OUTPUT_ARTIFACTS_SUMMARY]` | Brief description of what this agent produces (e.g., `a working, tested application`) |
| `[PHASE_SEQUENCE_NOTE]` | Phase position in workflow (e.g., `Phase 1 of 3`) |
| `[PHASE_SPEC_LAYERS]` | Which spec layers are generated in this phase |
| `[PHASE_SPEC_SUMMARY]` | Brief summary of specs in this phase (e.g., `business, functional, stack specs`) |
| `[PHASE_SPECIFIC_RULES]` | Phase guardrails derived from PHASES.md |
| `[ROLE_DETAILS]` | Phase-specific role description |
| `[OUTPUT_ARTIFACTS]` | What artifacts are produced |
| `[OUTPUT_FORMAT]` | Format of output artifacts |
| `[ADDITIONAL_COMPLETION_CRITERIA]` | Phase-specific completion checks |

## Specification Templates

Specification templates define the structure for spec documents produced by specification agents.

### Required Sections

Specification templates share common sections to anchor clarity and traceability:

| Section | Purpose |
|---------|---------|
| Frontmatter | YAML metadata with state tracking |
| Title | Concept name |
| References | Upstream spec links (except Business) |
| Scope | What's included and excluded |
| [Layer-specific content] | Varies by layer |
| Acceptance Criteria | Testable requirements with IDs |

**Frontmatter Expectations:**

Spec templates open with YAML frontmatter capturing identity and lifecycle:

```yaml
---
id: [LAYER_PREFIX]-[CONCEPT]
status: draft
created: [TIMESTAMP]
prompt_version: [GIT_HASH]
---
```

**Required frontmatter fields:**
- `id`: Spec identifier (e.g., `BUS-LOGIN`, `FUN-AUTH-FLOW`)
- `status`: Initial state is `draft`
- `created`: ISO8601 timestamp when spec generated
- `prompt_version`: Git commit hash of prompt file at generation

**Optional frontmatter fields** (added during execution):
- `implemented`: Timestamp when development completes
- `deployed`: Timestamp when deployment completes
- `validated`: Timestamp when validation completes

Frontmatter reflects how specifications move through phases, capturing both origin and progress.

### Consistency Principles

Templates encourage consistent structure across runs, discourage stray sections or placeholders, and reduce variance so downstream consumers receive predictable documents.

### Placeholder Handling

Placeholders are replaced with concrete content. When a section truly does not apply, state the reason rather than leave it empty.

## Agent Templates

Agent templates define the structure for agent definition files.

### Required Sections

Agent templates share core sections that describe identity, purpose, inputs, outputs, directives, and completion thinking:

| Section | Purpose |
|---------|---------|
| YAML Frontmatter | name, description, tools |
| Role | Agent's purpose, including core responsibilities |
| Framework Reference | Links to relevant framework files |
| Input | Upstream specs and user input |
| Output | Location, template, format |
| Directives | Guidance and guardrails |
| Completion Criteria | Self-validation checklist |
| Failure Handling | Error response table |

### Agent Definition Format

Agent definitions use GitHub Custom Agent format:

```
---
name: smaqit.[layer]
description: [One-line description]
tools: ["read", "edit", "search"]
---

# [Layer] Agent

## Role
...

## Input
...

## Output
...
```

Note: The code fence above is for illustration only. Actual agent files start directly with the YAML frontmatter (`---`).

## Prompt Templates

Prompt templates define the structure for prompt files that serve as input records and agent invocation interface.

### Required Sections

Prompt templates share core sections:

| Section | Purpose |
|---------|---------|
| YAML Frontmatter | name, description, agent |
| Purpose | What this prompt captures |
| Requirements | Sub-sections with suggested structure |
| Comment Examples | `<!-- Example: ... -->` for guidance |

### Prompt Template Format

Prompt templates use GitHub Copilot prompt format:

```markdown
---
name: smaqit.[layer]
description: [One-line description]
agent: smaqit.[layer]
---

# [Layer] Prompt

[Brief explanation]

## Requirements

[Sub-sections with suggested structure]

<!-- Example: [Guidance showing format] -->

[User fills requirements here]
```

### Free-Style with Structure

Prompts are **free-style natural language inputs**, not rigidly structured forms. Templates provide:

- **Suggested structure**: Sections and sub-sections to guide users
- **Commented examples**: `<!-- Example: ... -->` showing good formats
- **No enforcement**: Users write in their own words

Agents interpret natural language and request clarification if needed. See [PROMPTS](PROMPTS.md) for complete principles.

### Comment Convention

Templates and shipped prompts include examples wrapped in HTML comments:

```markdown
### Actors

<!-- Example: "Mario Fan - Users who love Nintendo's Mario franchise" -->

[User writes actual actors here]
```

Agents treat HTML comments as guidance only, keeping example text out of generated specifications.

### Single Manifest Pattern

Unlike specifications (one file per concept), prompts are **single manifest files**:

- One prompt per layer captures all requirements for that layer
- Users add features to existing prompts as projects evolve
- Prompts become consolidated input records for entire project

## Template Completeness

A template feels complete when required sections are present, placeholders are clearly marked, purposes are unambiguous, and guidance remains inside comments rather than the main content.
