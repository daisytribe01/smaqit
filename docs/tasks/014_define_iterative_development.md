# Define Iterative Development Using smaqit

**Status:** Not Started  
**Created:** 2025-12-13

## Description

smaqit should embrace iterative development. New features should be backwards compatible, meaning specs need to be either modular or refactored upon feature introduction.

This task involves:
1. Identifying a new principle for iterative development
2. Defining a process for maintaining backwards compatibility in specs
3. Determining how specs should evolve as features are added

## Acceptance Criteria

- [ ] New principle documented in `framework/PRINCIPLES.md` covering iterative development
- [ ] Process defined for backwards compatibility when introducing new features
- [ ] Guidelines for modular vs. refactored specs documented
- [ ] Decision tree or rules for when to extend vs. refactor existing specs
- [ ] Framework documentation updated to reflect iterative workflow

## Notes

Key questions to address:
- When should a spec be extended (modular) vs. refactored?
- How do we version specs to track evolution? (git is your friendly versioning system)
- What triggers a spec refactor vs. a new spec?
- How do implementation agents handle spec changes?
