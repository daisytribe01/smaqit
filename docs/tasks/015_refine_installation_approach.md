# Refine Installation Approach

**Status:** Not Started  
**Created:** 2025-12-13

## Description

Evaluate and decide on the smaqit installation approach. The current installer copies the entire `framework/` directory into user projects at `.smaqit/framework/`. 

The question: Should the framework remain as separate files that get copied, or should the framework content become embedded directly into the agents and templates?

### Current Approach
- Framework files are standalone documentation in `framework/`
- Installer copies them to `.smaqit/framework/`
- Agents/templates reference framework files

### Alternative Approach
- Embed framework knowledge directly into agent instructions
- Templates become self-contained with embedded principles
- No separate framework files needed in user projects

### Considerations
- **Maintainability**: Separate files are easier to update; embedded requires updating multiple agents
- **Context window**: Embedded agents are larger; separate files require file reads
- **User understanding**: Separate files let users read the methodology; embedded is opaque
- **Distribution size**: Embedded is self-contained; separate has more files

## Acceptance Criteria

- [ ] Analyze pros/cons of each approach
- [ ] Consider hybrid approaches (e.g., minimal framework + embedded essentials)
- [ ] Document decision rationale
- [ ] Update installer if approach changes
- [ ] Update copilot-instructions.md if structure changes

## Notes

This decision affects:
- `installer/main.go` - what gets copied where
- All agent files - how they reference framework
- Template files - self-contained vs referential
- User project structure - `.smaqit/` contents
