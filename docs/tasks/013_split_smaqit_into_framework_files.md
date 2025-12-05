# Task: Split SMAQIT.md into Framework Files

**ID**: 013
**Status**: new

## Context

The current `framework/SMAQIT.md` contains all framework documentation in a single file. Split it into four focused files, each describing a core feature of smaqit:

1. **PHASES.md** — The development phases (Specification, Implementation, Validation)
2. **LAYERS.md** — The specification layers (Business, Functional, Stack, Infrastructure, Coverage)
3. **SPECS.md** — The specification format and structure rules
4. **AGENTS.md** — The agent definitions, roles, and how they map to layers/phases

This separation improves maintainability and allows agents to load only the context they need.

## Acceptance Criteria

- [ ] Create `framework/PHASES.md` — Phase definitions, ordering, and transitions
- [ ] Create `framework/LAYERS.md` — Layer definitions, dependencies, and agent mappings
- [ ] Create `framework/SPECS.md` — Specification format, templates, and validation rules
- [ ] Create `framework/AGENTS.md` — Agent roles, responsibilities, and layer/phase mappings
- [ ] Update `framework/SMAQIT.md` to be an index/overview that references the four files
- [ ] Update installer to copy all framework files (not just SMAQIT.md)
- [ ] Update copilot-instructions if needed

## Notes

- Keep SMAQIT.md as the entry point / table of contents
- Consider cross-references between the four files
- Installer may need to handle `framework/` as a directory copy
