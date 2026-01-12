# Templates

Templates define the structure that agents follow when producing output. This document establishes the principles for specification templates, agent templates, and prompt templates.

## Template Types

smaqit uses three types of templates:

| Type | Location | Purpose | Produces |
|------|----------|---------|----------|
| **Specification templates** | `templates/specs/` | Structure for spec documents | `specs/**/*.md` |
| **Agent templates** | `templates/agents/` | Structure for agent definitions | `agents/*.agent.md` |
| **Prompt templates** | `templates/prompts/` | Structure for prompt files | `.github/prompts/*.prompt.md` |

## Placeholder Convention

**Consistent placeholders:** All templates use bracket-wrapped SCREAMING_CASE format for customizable values. Common placeholders include layer names, phase names, prefixes, concepts, and sequential numbers. Agent templates add specialized placeholders for upstream spec paths, user input descriptions, layer-specific rules, phase-specific rules, and output formats.

## Specification Templates

Specification templates define the structure for spec documents produced by specification agents.

### Required Sections

**Core structure:** Every specification template includes frontmatter (YAML metadata with state tracking), title (concept name), references (upstream spec links except Business), scope (inclusions and exclusions), layer-specific content (varies by layer), and acceptance criteria (testable requirements with IDs).

**Frontmatter requirements:** All spec templates begin with YAML frontmatter containing required fields (identifier, initial draft status, creation timestamp, prompt version git hash) and optional fields added by implementation agents (implemented/deployed/validated timestamps). Specification agents generate frontmatter with required fields. Implementation agents update frontmatter as specs progress through phases.

### Compliance Principles

**Template adherence:** When producing specs from templates, agents use the template from the appropriate layer, produce consistent output structure across runs, avoid adding undefined sections, avoid omitting required sections, avoid leaving placeholder text, and minimize variance in generated artifacts.

### Placeholder Handling

All placeholders receive replacement with actual content. If a section is not applicable, state "Not applicable" with reason. Empty sections are not permitted.

## Agent Templates

Agent templates define the structure for agent definition files.

### Required Sections

**Core structure:** Every agent template includes YAML frontmatter (name, description, tools), role (agent's purpose including core responsibilities), framework reference (links to relevant framework files), input (upstream specs and user input), output (location, template, format), directives (behavioral rules), completion criteria (self-validation checklist), and failure handling (error response patterns).

### Agent Definition Format

Agent definitions use GitHub Custom Agent format beginning with YAML frontmatter specifying name, description, and tools, followed by markdown sections defining role, input, output, and other requirements.

## Prompt Templates

Prompt templates define the structure for prompt files that serve as input records and agent invocation interface.

### Required Sections

**Core structure:** Every prompt template includes YAML frontmatter (name, description, agent), purpose (what this prompt captures), requirements (sub-sections with suggested structure), and comment examples (guidance showing format).

### Prompt Template Format

Prompt templates use GitHub Copilot prompt format with YAML frontmatter followed by explanation and requirements sections with suggested structure.

### Free-Style with Structure

**Natural language input:** Prompts are free-style natural language inputs, not rigidly structured forms. Templates provide suggested structure (sections and sub-sections to guide users) and commented examples (showing good formats). No rigid enforcement—users write in their own words. Agents interpret natural language and request clarification if needed.

### Comment Convention

**Example guidance:** Templates and shipped prompts include examples wrapped in HTML comments. Agents ignore HTML comments to prevent example requirements from contaminating generated specs.

### Single Manifest Pattern

**Consolidated input:** Unlike specifications (one file per concept), prompts are single manifest files. One prompt per layer captures all requirements for that layer. Users add features to existing prompts as projects evolve. Prompts become consolidated input records for entire project.

## Template Completeness

**Complete templates:** A template is complete when all required sections are present, placeholders are clearly marked with bracket-SCREAMING_CASE format, section purposes are unambiguous, layer-specific rules from framework files are incorporated (for spec templates), and comment examples use HTML comment format (for prompt templates).
