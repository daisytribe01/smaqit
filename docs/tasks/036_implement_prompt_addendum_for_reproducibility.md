# Implement Prompt Addendum for Reproducibility

**Status:** Complete  
**Created:** 2025-12-28  
**Completed:** 2025-12-28  
**Source:** User Testing Report Issue #5 (2025-12-27)

## Description

When users refine specs iteratively (e.g., "fix the stack spec"), the additional instructions are not captured in prompt files, breaking reproducibility. Implement "Addendum" section to capture all iterative refinement instructions.

**ASSESSMENT COMPLETED**: Evaluated two options:
- **Option 1 (Implemented):** Addendum section - append-only log of refinements
- **Option 2 (Rejected):** Update existing prompt content - merge refinements

**Decision:** Option 1 aligns with "Prompts are input records" principle and "Reproducible from Input Set" core principle. Input records should capture ALL user instructions, not just final merged state.

## Acceptance Criteria

- [x] Prompt templates updated with optional `## Addendum` section
- [x] Section documented as: "Iterative refinements and amendments (auto-generated)"
- [x] Agent instructions updated to detect spec modification requests
- [x] Agents append refinement instructions to prompt file with timestamp
- [x] Addendum format: `[YYYY-MM-DD HH:MM] [refinement instruction]`
- [x] Specification agent templates (Level 1) include addendum appending logic
- [x] All 5 specification agents (Level 2) implement addendum behavior
- [x] Framework documentation (PROMPTS.md) explains addendum principle

## Implementation Details

### Files Modified (12 files)

**Level 0 (Framework):**
- `framework/PROMPTS.md` - Added "Iterative Refinement with Addendum" section

**Level 1 (Templates):**
- `templates/prompts/specification-prompt.template.md` - Added Addendum section structure

**Level 2 (Prompts - 5 files):**
- `prompts/smaqit.business.prompt.md`
- `prompts/smaqit.functional.prompt.md`
- `prompts/smaqit.stack.prompt.md`
- `prompts/smaqit.infrastructure.prompt.md`
- `prompts/smaqit.coverage.prompt.md`

**Level 2 (Agents - 5 files):**
- `agents/smaqit.business.agent.md`
- `agents/smaqit.functional.agent.md`
- `agents/smaqit.stack.agent.md`
- `agents/smaqit.infrastructure.agent.md`
- `agents/smaqit.coverage.agent.md`

### Addendum Format

```markdown
## Addendum

Iterative refinements and amendments (auto-generated). Agents append refinement instructions here when users request modifications to existing specifications.

Format: `[YYYY-MM-DD HH:MM] [refinement instruction]`

<!-- Example: "[2025-12-28 14:30] Change from Python to Go for better performance" -->
```

### Agent Behavior

Added MUST directive to all 5 specification agents:

> **Detect spec modification requests**: When user requests modifications to existing specifications (e.g., "fix the [layer] spec", "refine X"), append the refinement instruction to `.github/prompts/smaqit.[layer].prompt.md` under the `## Addendum` section with timestamp format: `[YYYY-MM-DD HH:MM] [user refinement instruction]`

## Validation

- [x] Installer builds successfully (v833bd14)
- [x] Test installation verified all 5 prompts contain Addendum section
- [x] All 5 agents contain addendum appending instructions
- [x] Code review passed with no issues
- [x] Security scan: N/A (markdown documentation only)

## Impact

**Severity:** High (now resolved)  
**User Impact:** Restores reproducibility principle. Prompt files now maintain complete input record. Regenerating specs from prompts (including addendum entries) produces equivalent behavioral outcomes.

## Notes

**FRAMEWORK LEVEL CHANGE**. Critical for maintaining input record completeness. Agents now detect when they're modifying existing specs vs creating new specs and capture refinement instructions accordingly.

The implementation follows the Level 0→1→2 architecture:
1. Framework principle documented in PROMPTS.md
2. Template structure defined in specification-prompt.template.md
3. Concrete implementations in 5 prompts + 5 agents
