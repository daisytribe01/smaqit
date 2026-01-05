# Session: Task 049 - Fix Implementation Agent CLI Directives

**Date:** 2026-01-05  
**Tasks Completed:** 049 (primary), 051 (preventive), 052 (preventive)  
**Session Type:** Framework refinement (Level 0→1→2 cascade)

## Session Overview

Fixed weak CLI directive phrasing in all three implementation agents (Development, Deployment, Validation) by cascading changes from Level 1 template to Level 2 agents. Changed from instructional language ("Determine which specs...") to imperative language ("Execute X as first action and process ONLY...") to ensure agents programmatically query CLI for spec state rather than relying on implicit understanding.

## Problem Statement

**Root Cause:** Implementation agents had instructional directive: "Determine which specs to process using `smaqit plan --phase=[phase]`"

This weak phrasing allowed agents to satisfy the spirit without executing the command—they could determine specs through implicit understanding of the codebase rather than programmatic CLI query.

**Evidence:** Development phase report from E2E testing (Task 048) contained no `smaqit plan` or `smaqit status` commands in execution log, indicating agent processed specs without querying CLI.

**Impact:**
- Violates "CLI as single source of truth" principle
- Risk of processing wrong specs in complex scenarios (many specs, failed specs, already-implemented specs)
- Could lead to re-implementing completed specs or missing failed specs needing retry

**Why it matters:** In complex projects with many specs across multiple layers and phases, implicit understanding will fail. CLI must be programmatically queried to ensure correct filtering based on spec state (draft/failed vs implemented/deployed/validated).

## Implementation Approach

### Critical Assessment

**Question:** Should we update only Development agent (task scope) or all implementation agents?

**Answer:** All three agents + template.

**Rationale:**
1. **Pattern consistency** — Same weak phrasing exists in Deployment and Validation agents
2. **Preventive fix** — Tasks 051 and 052 identified same issue; address all now
3. **Level hierarchy** — Respect smaqit's own Level 0→1→2 workflow by updating template first
4. **Future-proof** — Template change prevents new agents from having same issue

**Trade-offs accepted:**
- Larger scope than originally planned (1 agent → 3 agents + template)
- More files to modify and test
- **Benefit:** Comprehensive fix prevents future bug reports, ensures consistency

### Workflow: Level 1 → Level 2

**Phase 1: Template (Level 1)**
- Updated `templates/agents/implementation-agent.template.md` first
- Changed directive from instructional to imperative
- Added output requirement for CLI command documentation

**Phase 2: Agents (Level 2)**
- Cascaded changes to Development, Deployment, Validation agents
- Maintained agent-specific phrasing (phase names, report contexts)
- Added output requirements to all three agents

**Phase 3: Validation**
- Built installer
- Tested installation in clean directory
- Verified installed agents contain imperative language
- Verified output requirements present

**Phase 4: Documentation**
- Updated task 049 with implementation details
- Marked tasks 051 and 052 as completed by 049
- Updated PLANNING.md to reflect completed tasks

## Changes Made

### Level 1: Template

**File:** `templates/agents/implementation-agent.template.md`

**Directive change (line 42):**
```diff
- - Determine which specs to process using `smaqit plan --phase=[PHASE]`
+ - Execute `smaqit plan --phase=[PHASE]` as the first action and process ONLY the specs returned by this command
```

**Output requirement added (line 37):**
```diff
  - Phase report MUST be written to `.smaqit/reports/[phase]-phase-report-YYYY-MM-DD.md` documenting phase outcomes
+ - Phase report MUST document the output of `smaqit plan --phase=[PHASE]` command execution
```

### Level 2: Development Agent

**File:** `agents/smaqit.development.agent.md`

**Directive change (line 49):**
```diff
- - Determine which specs to process using `smaqit plan --phase=develop`
+ - Execute `smaqit plan --phase=develop` as the first action and process ONLY the specs returned by this command
```

**Output requirement added (line 44):**
```diff
  - Development report MUST be written to `.smaqit/reports/development-phase-report-YYYY-MM-DD.md` and document build/test/run outcomes
+ - Development report MUST document the output of `smaqit plan --phase=develop` command execution
```

### Level 2: Deployment Agent

**File:** `agents/smaqit.deployment.agent.md`

**Directive change (line 51):**
```diff
- - Determine which specs to process using `smaqit plan --phase=deploy`
+ - Execute `smaqit plan --phase=deploy` as the first action and process ONLY the specs returned by this command
```

**Output requirement added (line 45):**
```diff
  - Deployment report MUST be written to `.smaqit/reports/deployment-phase-report-YYYY-MM-DD.md` with health status, endpoints, and scrubbed logs
+ - Deployment report MUST document the output of `smaqit plan --phase=deploy` command execution (if report is generated)
  - Configuration files following stack-specific conventions
```

Note: Deployment agent reports are optional, so qualifier added: "if report is generated"

### Level 2: Validation Agent

**File:** `agents/smaqit.validation.agent.md`

**Directive change (line 47):**
```diff
- - Determine which specs to process using `smaqit plan --phase=validate`
+ - Execute `smaqit plan --phase=validate` as the first action and process ONLY the specs returned by this command
```

**Output requirement added (line 40):**
```diff
  - Markdown document written to `.smaqit/reports/validation-phase-report-YYYY-MM-DD.md` following validation report format (see below)
+ - Validation report MUST document the output of `smaqit plan --phase=validate` command execution
  - Maps test results to Coverage spec test cases
```

## Key Design Decisions

### 1. Imperative Language: "Execute...as first action"

**Decision:** Changed from "Determine using..." to "Execute...as the first action and process ONLY..."

**Rationale:**
- **Unambiguous:** "Execute X as first action" leaves no room for interpretation
- **Programmatic enforcement:** Agents must run command, not just satisfy spirit
- **Explicit sequencing:** "first action" mandates when command must run
- **Strict filtering:** "process ONLY" prevents processing additional specs beyond CLI output

**Alternative rejected:** "Should execute..." or "Must use..." — still allows skipping actual command execution

### 2. Output Documentation Requirement

**Decision:** Added MUST requirement that phase reports document CLI command output

**Rationale:**
- **Verification:** Enables checking whether agent actually executed command
- **Audit trail:** Reports show which specs were returned by CLI
- **Debugging:** If agent processes wrong specs, report reveals CLI output
- **Accountability:** Agent cannot claim it executed command without evidence in report

**Alternative rejected:** Optional recommendation — wouldn't ensure compliance

### 3. Template-First Approach

**Decision:** Updated Level 1 template before Level 2 agents

**Rationale:**
- **Respects hierarchy:** smaqit's own framework principle (Level 0→1→2→3)
- **Prevents recurrence:** Future agents generated from template won't have weak directive
- **Consistency:** Template defines structure that agents follow
- **Best practice:** Changes should flow downward through levels, not upward

**Alternative rejected:** Fix agents first, update template later — violates level dependency flow

### 4. Consistency Across All Implementation Agents

**Decision:** Applied fix to Development, Deployment, and Validation agents simultaneously

**Rationale:**
- **Prevent duplication:** Tasks 051 and 052 would need identical changes
- **Pattern consistency:** All implementation agents should follow same directive pattern
- **Efficiency:** Single comprehensive fix better than three separate fixes
- **User experience:** Consistent behavior across all phases reduces confusion

**Alternative rejected:** Fix only Development agent per task scope — leaves other agents with same issue

## Validation Results

### Build Verification

```bash
cd installer && make build
```

**Result:** ✅ Built successfully
- No compilation errors
- Installer version: ebe25a7-dirty
- Output: `dist/smaqit`

### Installation Test

```bash
mkdir test && cd test
../dist/smaqit init
```

**Result:** ✅ Installation successful
- `.smaqit/` directory created
- Templates copied
- Agent definitions installed to `.github/agents/`
- Prompt files installed to `.github/prompts/`

### Agent Directive Verification

```bash
grep "Execute \`smaqit plan" .github/agents/smaqit.*.agent.md
```

**Result:** ✅ All 3 agents have imperative directive
- `.github/agents/smaqit.development.agent.md:50`
- `.github/agents/smaqit.deployment.agent.md:52`
- `.github/agents/smaqit.validation.agent.md:48`

All contain: "Execute `smaqit plan --phase=[phase]` as the first action and process ONLY the specs returned by this command"

### Output Requirement Verification

```bash
grep "MUST document the output of" .github/agents/smaqit.*.agent.md
```

**Result:** ✅ All 3 agents have output documentation requirement
- `.github/agents/smaqit.development.agent.md:44`
- `.github/agents/smaqit.deployment.agent.md:45`
- `.github/agents/smaqit.validation.agent.md:40`

All contain requirement to document CLI command output in phase reports.

## Files Modified

**Level 1 (Template):**
1. `templates/agents/implementation-agent.template.md`

**Level 2 (Agents):**
2. `agents/smaqit.development.agent.md`
3. `agents/smaqit.deployment.agent.md`
4. `agents/smaqit.validation.agent.md`

**Documentation:**
5. `docs/tasks/049_fix_development_agent_cli_directive.md`
6. `docs/tasks/051_fix_validation_agent_cli_directive.md`
7. `docs/tasks/052_fix_deployment_agent_cli_directive.md`
8. `docs/tasks/PLANNING.md`

**Total: 8 files modified**

## Tasks Completed

| ID | Title | Status |
|----|-------|--------|
| 049 | Fix Development Agent CLI Directive | ✅ Completed |
| 051 | Fix Validation Agent CLI Directive | ✅ Completed (preventive) |
| 052 | Fix Deployment Agent CLI Directive | ✅ Completed (preventive) |

**Task 049** was the primary objective. Tasks 051 and 052 were addressed preventively as part of the comprehensive fix.

## Problems Solved

### Problem 1: Weak Directive Language

**Symptom:** Agents could interpret "Determine which specs to process using X" as guidance rather than command

**Root Cause:** Instructional phrasing ("Determine using...") allows satisfaction without execution

**Solution:** Imperative phrasing ("Execute X as first action and process ONLY...")

**Impact:** No interpretation ambiguity remains. Agents must execute CLI command programmatically.

### Problem 2: No Execution Verification

**Symptom:** No way to verify whether agent actually executed CLI command or just satisfied spirit

**Root Cause:** No requirement to document command execution in phase reports

**Solution:** Added MUST requirement that reports document CLI command output

**Impact:** Phase reports now provide evidence of CLI execution (or lack thereof).

### Problem 3: Pattern Inconsistency

**Symptom:** Same weak directive present in all three implementation agents

**Root Cause:** Template used instructional language that cascaded to all agents

**Solution:** Fixed template first, then cascaded to all agents

**Impact:** Consistent pattern across all implementation agents, future-proof for new agents.

## Lessons Learned

### 1. Instructional vs Imperative Language Matters

**Observation:** "Determine using X" sounds mandatory but allows interpretation. "Execute X as first action" is unambiguous.

**Lesson:** When specifying agent behavior, use imperative verbs (Execute, Run, Call) with explicit sequencing (as first action, before Y, after Z).

**Application:** Review other agent directives for instructional phrasing that could allow interpretation.

### 2. Verification Requirements Enable Compliance Checking

**Observation:** Without documentation requirement, no way to verify compliance short of code inspection.

**Lesson:** When mandating agent behavior, add output requirement that documents execution.

**Application:** Phase reports should include execution logs for all critical commands, not just outcomes.

### 3. Template-First Prevents Recurrence

**Observation:** Fixing agents without fixing template means next agent regeneration reintroduces issue.

**Lesson:** Always update templates (Level 1) before agents (Level 2) to ensure changes persist.

**Application:** When fixing agent issues, check if template is source and update accordingly.

### 4. Comprehensive Fix Better Than Incremental

**Observation:** Tasks 051 and 052 would require identical changes to different agents.

**Lesson:** When pattern problem affects multiple components, fix comprehensively in single pass.

**Application:** Before implementing task, assess whether similar issues exist elsewhere and address preventively.

## Next Steps

### Immediate

**Task 049 is complete.** All acceptance criteria met, implementation validated, documentation updated.

### Related Work

**Task 050: Redesign Coverage Prompt** — Next high-priority blocker
- Coverage prompt structure needs improvement based on E2E testing feedback
- Similar comprehensive approach may be needed

**Task 053: Fix Validation Frontmatter Updates** — Another high-priority blocker
- Validation agent may not be updating spec frontmatter correctly
- Could be related to same pattern of weak directives

**Task 048: E2E Agent Workflow Testing** — Original source of this discovery
- Continue E2E testing to discover other directive weaknesses
- Use testing feedback to identify similar issues proactively

### Future Considerations

**Audit all agent directives:** Review all 8 agents for other instances of instructional phrasing that could allow interpretation rather than mandating execution.

**Standardize directive patterns:** Consider creating directive writing guidelines in framework documentation to prevent weak phrasing in future agents.

## Session Metrics

**Duration:** ~90 minutes (session recap → implementation → validation → documentation)

**Tasks completed:** 3 (049, 051, 052)

**Files modified:** 8 (1 template, 3 agents, 4 documentation)

**Lines changed:** ~20 insertions, ~5 deletions

**Testing performed:**
- ✅ Installer build verification
- ✅ Installation test in clean directory
- ✅ Agent directive verification (grep)
- ✅ Output requirement verification (grep)

**Commits:** 2
1. Initial plan outlining approach
2. Complete implementation with validation

## Code Quality

**Strengths:**
- Respected smaqit's own Level 0→1→2 hierarchy
- Comprehensive fix prevents recurrence
- Clear imperative language eliminates ambiguity
- Added verification mechanism (output requirement)
- Thorough testing and validation
- Well-documented decisions and rationale

**Potential improvements:**
- Could add automated test checking that agents actually execute CLI commands during integration testing
- Could create linter rule to flag instructional phrasing ("Determine", "Use", "Consider") in agent MUST directives
- Could add explicit examples in agent documentation showing command execution patterns

## Reference

This session completes the pattern of strengthening agent directives discovered through E2E testing (Task 048). The fix ensures CLI is programmatically authoritative for spec state, not just suggested or implied.

**Pattern established:**
- Level 1 templates define structure and rules
- Level 2 agents follow templates without deviation
- Imperative language ("Execute X as first action") for mandatory behavior
- Output requirements provide verification mechanism

**Related history:**
- Task 048: E2E testing discovered issue
- Task 045: Implemented stateful specifications (enabled state-based filtering)
- Task 047: Implemented incremental processing (depends on correct state queries)
