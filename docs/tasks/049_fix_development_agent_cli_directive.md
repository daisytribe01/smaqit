# Task 049: Fix Development Agent CLI Directive

**Status:** Completed  
**Priority:** High (Release Blocker)  
**Created:** 2026-01-05  
**Completed:** 2026-01-05  
**Related:** Task 048 (E2E Testing), Issue 4

## Problem

Development agent processed incremental implementation without executing `smaqit plan --phase=develop` to determine which specs require implementation. Agent relied on implicit understanding rather than explicit CLI state query.

**Evidence:** Development phase report (`development-phase-report-2026-01-04-luigi.md`) contains no `smaqit plan` or `smaqit status` commands in execution log.

**Root Cause:** Current directive is instructional ("Determine which specs to process using `smaqit plan --phase=develop`") rather than imperative ("Execute `smaqit plan --phase=develop` as first action").

**Impact:**
- Violates established phase workflow directive
- Undermines CLI as single source of truth for spec state
- Risk of processing wrong specs or missing specs in complex scenarios
- Could lead to re-implementing already-implemented specs or skipping failed specs

## Objective

Update Development agent directive to mandate explicit CLI command execution as first step, ensuring CLI is authoritative source for determining which specs require processing.

## Acceptance Criteria

- [x] Updated `agents/smaqit.development.agent.md` directive from instructional to imperative phrasing
- [x] Added output requirement that development report MUST document CLI command execution
- [x] Agent directive explicitly states command must be "first action"
- [x] Agent directive specifies "process ONLY the specs returned" by CLI
- [x] Verified directive change with test execution
- [x] Applied same fix to Deployment and Validation agents for consistency
- [x] Updated Level 1 template to prevent future occurrences

## Implementation Plan

1. **Update agent directive** (`agents/smaqit.development.agent.md` line ~49):
   - **From:** "Determine which specs to process using `smaqit plan --phase=develop`"
   - **To:** "Execute `smaqit plan --phase=develop` as the first action and process ONLY the specs returned"

2. **Add output requirement** (Output section):
   - Add: "Development report MUST document the output of `smaqit plan --phase=develop` command"

3. **Verify consistency:**
   - Check template (`templates/agents/implementation-agent.template.md`) for similar patterns
   - If template uses same weak phrasing, update it as well

## Files to Modify

- `agents/smaqit.development.agent.md` (directive + output requirements)
- `templates/agents/implementation-agent.template.md` (optional, if pattern exists)

## Testing

**Manual verification:**
1. Read updated directive
2. Confirm imperative phrasing is unambiguous
3. Confirm output requirement is clear

**Optional agent test:**
1. Run Development agent in test project
2. Verify agent executes `smaqit plan --phase=develop`
3. Verify report includes command output

## Estimated Effort

1 hour

## Dependencies

None (can be implemented independently)

## Blocks

- v0.5.0 release (this is a release blocker)

## Related Tasks

- Task 051: Fix Validation Agent CLI Directive (same issue, different agent)
- Task 052: Fix Deployment Agent CLI Directive (preventive)
- Task 048: E2E Agent Workflow Testing (discovered this issue)

## Notes

**Why this matters:** In complex projects with many specs across multiple layers, implicit understanding will fail. CLI must be programmatically queried to ensure correct spec filtering (draft/failed vs implemented/deployed/validated).

**Alternative considered:** Adding workflow section with numbered steps. Rejected because directive-based approach (MUST/MUST NOT) is established smaqit pattern.

**Success criteria:** Agent directive is so clear that no interpretation ambiguity remains. "Execute X as first action" leaves no room for "I satisfied the spirit without executing the command."

## Implementation Details

**Completed:** 2026-01-05

### Changes Made

**Level 1 (Template):**
- Updated `templates/agents/implementation-agent.template.md`:
  - Line 42: Changed from "Determine which specs to process using..." to "Execute `smaqit plan --phase=[PHASE]` as the first action and process ONLY the specs returned by this command"
  - Line 37: Added "Phase report MUST document the output of `smaqit plan --phase=[PHASE]` command execution"

**Level 2 (Agents):**
- Updated `agents/smaqit.development.agent.md`:
  - Line 49: Changed directive to imperative "Execute...as the first action"
  - Line 44: Added output requirement documenting CLI command execution
  
- Updated `agents/smaqit.deployment.agent.md`:
  - Line 51: Changed directive to imperative "Execute...as the first action"
  - Line 45: Added output requirement documenting CLI command execution (with "if report is generated" qualifier)
  
- Updated `agents/smaqit.validation.agent.md`:
  - Line 47: Changed directive to imperative "Execute...as the first action"
  - Line 40: Added output requirement documenting CLI command execution

### Design Decisions

**1. Template-First Approach:**
- Respected smaqit's Level 0→1→2→3 workflow by updating template before agents
- Prevents future agents from having same weak directive

**2. Consistency Across All Implementation Agents:**
- Applied fix to Development, Deployment, and Validation agents
- Tasks 051 and 052 no longer needed (addressed preventively)
- Consistent pattern improves predictability and reduces confusion

**3. Imperative Language:**
- "Execute X as the first action and process ONLY..." is unambiguous
- No room for interpretation or skipping command execution
- CLI becomes programmatically authoritative, not just suggested

**4. Report Documentation Requirement:**
- Added explicit requirement that phase reports MUST document CLI command output
- Enables verification that agents executed command (not just satisfied spirit)
- Future validation can check reports for command execution evidence

### Testing & Validation

**Build verification:**
```bash
cd installer && make build
# Result: ✓ Built successfully
```

**Installation test:**
```bash
mkdir test && cd test
../dist/smaqit init
# Result: ✓ All files copied, agents installed
```

**Agent verification:**
```bash
grep "Execute \`smaqit plan" .github/agents/smaqit.*.agent.md
# Result: ✓ All 3 implementation agents have imperative directive
```

**Output requirement verification:**
```bash
grep "MUST document the output of" .github/agents/smaqit.*.agent.md
# Result: ✓ All 3 agents have output documentation requirement
```

### Files Modified

1. `templates/agents/implementation-agent.template.md` (Level 1)
2. `agents/smaqit.development.agent.md` (Level 2)
3. `agents/smaqit.deployment.agent.md` (Level 2)
4. `agents/smaqit.validation.agent.md` (Level 2)
5. `docs/tasks/049_fix_development_agent_cli_directive.md` (documentation)

**Total: 5 files**

### Impact on Related Tasks

**Tasks addressed preventively:**
- Task 051: Fix Validation Agent CLI Directive → No longer needed (fixed in this task)
- Task 052: Fix Deployment Agent CLI Directive → No longer needed (fixed in this task)

Both tasks can be closed as "Completed by Task 049".
