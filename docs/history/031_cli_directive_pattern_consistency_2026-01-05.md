# Session: CLI Directive Pattern Consistency (Tasks 049, 051, 052)

**Date:** 2026-01-05  
**Session Focus:** Establish consistent CLI query pattern across all implementation agents
**Tasks Completed:** 049, 051, 052

---

## Session Overview

This session completed three related tasks (049, 051, 052) that addressed weak directive phrasing in implementation agents. All three implementation agents (Development, Deployment, Validation) now mandate explicit `smaqit plan --phase=[PHASE]` execution as first action, ensuring CLI is the authoritative source of truth for determining which specs require processing.

The work was surgical and focused—Level 2 (agents) and Level 1 (template) updates only, respecting the smaqit levels hierarchy.

---

## Problem Statement

### Root Cause

All three implementation agents had instructional rather than imperative directive phrasing:

**Before (weak):**
```markdown
- Determine which specs to process using `smaqit plan --phase=[PHASE]`
```

This phrasing invited interpretation ambiguity:
- Agent could satisfy "determine" by reading specs directly
- No requirement to actually execute the CLI command
- CLI state query was suggested, not mandated

### Evidence

E2E testing (task 048) revealed that Development and Validation agents processed specs without executing `smaqit plan` commands. Phase reports contained no CLI command execution logs.

**Impact:**
- Violated established phase workflow directive
- Undermined CLI as single source of truth for spec state
- Risk of processing wrong specs (skipping failed specs, re-implementing completed specs)

### Why This Matters

In complex projects with many specs across multiple layers, implicit understanding will fail. The CLI must be programmatically queried to ensure correct spec filtering based on state (draft/failed vs implemented/deployed/validated).

---

## Solution

### Pattern Established

**After (imperative, strong):**
```markdown
- Execute `smaqit plan --phase=[PHASE]` as the first action and process ONLY the specs returned
```

**Output requirement added:**
```markdown
- [Phase] report MUST document the output of `smaqit plan --phase=[PHASE]` command
```

**Key elements:**
1. **"Execute"** — Mandates action, not just consideration
2. **"as the first action"** — Establishes sequence priority
3. **"process ONLY the specs returned"** — Prevents implicit filtering
4. **Report documentation** — Ensures verifiable compliance

---

## Implementation

### Phase 1: Level 2 Updates (Agents)

Updated all three implementation agents with identical pattern:

**1. agents/smaqit.development.agent.md**
- Line 50: Directive changed to imperative
- Line 44: Added CLI command documentation requirement

**2. agents/smaqit.deployment.agent.md**
- Line 52: Directive changed to imperative
- Line 45: Added CLI command documentation requirement

**3. agents/smaqit.validation.agent.md**
- Line 48: Directive changed to imperative
- Line 40: Added CLI command documentation requirement

### Phase 2: Level 1 Updates (Template)

Updated implementation agent template to prevent future regression:

**4. templates/agents/implementation-agent.template.md**
- Line 42: Updated placeholder directive to imperative phrasing
- Line 37: Added placeholder for CLI command documentation requirement

### Phase 3: Validation

**Build test:**
```bash
cd installer && make build
# Result: ✓ Built: dist/smaqit
```

**Installation test:**
```bash
mkdir test-052 && cd test-052
../dist/smaqit init
# Result: ✓ Initialized successfully
```

**Verification:**
```bash
grep -n "Execute \`smaqit plan --phase=" .github/agents/smaqit.*.agent.md
# Result: All three agents have updated directive
```

---

## Key Decisions

### 1. Fix All Three Agents Together

**Decision:** Update Development, Deployment, and Validation agents in single session.

**Rationale:**
- All three had identical root cause
- Pattern consistency more important than task isolation
- Prevents partial fix (fixing one agent but leaving others weak)
- Task 052 explicitly mentioned "pattern consistency"

**Alternative rejected:** Fix only Deployment agent (task 052) and leave 049/051 for separate work. Would leave inconsistent pattern.

### 2. Imperative vs Instructional Language

**Decision:** Use imperative mood ("Execute X as the first action") rather than instructional ("Determine X using Y").

**Rationale:**
- Imperative leaves no room for interpretation ambiguity
- "Execute" mandates action rather than suggesting approach
- "as the first action" establishes clear sequencing
- "process ONLY" prevents implicit additions

**Alternative rejected:** Keep instructional but add clarifying text. Still too ambiguous.

### 3. Add Output Requirement

**Decision:** Require phase reports to document CLI command output.

**Rationale:**
- Makes directive compliance verifiable
- Provides audit trail for troubleshooting
- Documents which specs were processed in each run
- Helps detect agent misbehavior

**Alternative rejected:** Trust agents to execute command without verification. Fails accountability principle.

### 4. Update Template Preventively

**Decision:** Update implementation agent template with same pattern.

**Rationale:**
- Level 1 feeds Level 2 (templates → agents)
- Future agent generation from template will have correct pattern
- Prevents regression when agents are regenerated
- Respects smaqit levels hierarchy

**Alternative rejected:** Only fix existing agents, leave template unchanged. Future regeneration would reintroduce weak phrasing.

---

## Problems Solved

### Problem 1: Interpretation Ambiguity

**Symptom:** Agents could satisfy "Determine which specs to process" without executing CLI.

**Root Cause:** Instructional rather than imperative directive phrasing.

**Solution:** Changed to "Execute ... as the first action and process ONLY the specs returned".

**Impact:** No ambiguity remains—agent MUST execute command.

### Problem 2: Missing Audit Trail

**Symptom:** Phase reports didn't document which specs were processed or why.

**Root Cause:** No requirement to document CLI command output.

**Solution:** Added output requirement that reports MUST document `smaqit plan` output.

**Impact:** Phase reports now provide verifiable compliance and troubleshooting context.

### Problem 3: Pattern Inconsistency

**Symptom:** Three implementation agents could have different filtering approaches.

**Root Cause:** No established pattern enforced across all implementation agents.

**Solution:** Applied identical directive pattern to all three agents simultaneously.

**Impact:** Pattern consistency ensures uniform behavior across all phases.

### Problem 4: Future Regression Risk

**Symptom:** Template still had weak phrasing that would reintroduce issue if agents regenerated.

**Root Cause:** Level 1 (template) not updated when Level 2 (agents) were fixed.

**Solution:** Updated implementation agent template with same pattern.

**Impact:** Future agent generation will have correct directive from the start.

---

## Technical Outcomes

### Directive Pattern Comparison

| Aspect | Before (Weak) | After (Strong) |
|--------|--------------|----------------|
| **Verb** | "Determine" | "Execute" |
| **Mandate** | Suggested approach | Required action |
| **Sequence** | Implied | "as the first action" |
| **Scope** | Ambiguous | "process ONLY the specs returned" |
| **Verification** | None | Report MUST document command output |

### Files Modified

**Level 2 (Agents): 3 files**
- agents/smaqit.development.agent.md
- agents/smaqit.deployment.agent.md
- agents/smaqit.validation.agent.md

**Level 1 (Template): 1 file**
- templates/agents/implementation-agent.template.md

**Documentation: 4 files**
- docs/tasks/049_fix_development_agent_cli_directive.md (status updated, implementation summary added)
- docs/tasks/051_fix_validation_agent_cli_directive.md (status updated)
- docs/tasks/052_fix_deployment_agent_cli_directive.md (status updated, implementation summary added)
- docs/tasks/PLANNING.md (moved 049, 051, 052 to Completed)

**Total: 8 files modified**

### Installer Impact

**Before:** Agents bundled with weak directive phrasing
**After:** All installed agents have imperative directives

**Verification:**
```bash
cd test-project
smaqit init
grep "Execute \`smaqit plan" .github/agents/*.agent.md
# Result: All three agents installed with updated directives
```

---

## Design Patterns Applied

### 1. Pattern Consistency Enforcement

Rather than fixing one agent at a time, applied identical pattern across all related components simultaneously. This ensures:
- Uniform behavior across all phases
- Easier to document and understand
- Reduces cognitive load for users
- Prevents partial fixes that leave inconsistencies

### 2. Level-Appropriate Changes

Made changes at appropriate levels in smaqit hierarchy:
- **Level 0 (Framework):** No changes needed (principles unchanged)
- **Level 1 (Templates):** Updated to prevent regression
- **Level 2 (Agents):** Fixed actual directive phrasing
- **Level 3 (Application):** User projects benefit from clearer directives

### 3. Imperative Over Instructional

Shifted from instructional guidance to imperative directives:
- "Determine X using Y" → "Execute X as first action"
- "Use CLI to find specs" → "Process ONLY the specs returned"
- Removes interpretation ambiguity
- Establishes clear sequencing and scope

### 4. Verifiable Compliance

Added output requirements to make directive compliance auditable:
- Reports MUST document command execution
- Provides troubleshooting context
- Enables pattern detection across multiple runs
- Makes agent misbehavior visible

---

## Session Metrics

**Duration:** ~1.5 hours (session recap → implementation → testing → documentation)

**Tasks Completed:** 3 (049, 051, 052)

**Files Created:** 1 (this history file)

**Files Modified:** 7
- 3 agent files (Development, Deployment, Validation)
- 1 template file (implementation-agent.template.md)
- 3 documentation files (task 049, 051, 052, PLANNING.md)

**Lines Changed:** ~12 insertions (directives + output requirements)

**Testing Performed:**
- Installer build verification ✓
- Installer init test ✓
- Installed agent verification ✓
- Directive consistency check ✓

**Commits:** 2
1. Initial plan with critical assessment
2. Complete implementation with all changes

---

## Key Learnings

### 1. Language Precision Matters

Small wording differences have large behavioral impact:
- "Determine" vs "Execute" = suggestion vs mandate
- "using X" vs "as first action" = approach vs sequence
- "process specs" vs "process ONLY returned specs" = ambiguous vs bounded

**Lesson:** In directive phrasing, every word matters. Be precise and imperative.

### 2. Pattern Consistency Over Task Isolation

Task 052 was originally "Deployment agent only" but critical assessment revealed all three agents had same issue. Fixing all together was better than sequential fixes:
- Prevents inconsistent interim states
- Easier to document and test
- Establishes clear pattern going forward

**Lesson:** When multiple components have identical root cause, fix them together rather than in isolation.

### 3. Template Updates Prevent Regression

Fixing agents without updating template would have meant next agent regeneration reintroduces the issue. Level 1 feeds Level 2, so both must be aligned.

**Lesson:** When fixing Level 2 artifacts, check if Level 1 source (template) needs updating too.

### 4. Verifiable Directives Are Better Directives

Adding output requirement (report MUST document command) makes compliance auditable. Without it, we'd have to trust agents execute the command without verification.

**Lesson:** For critical directives, add verification mechanism. Trust but verify.

### 5. Critical Assessment Catches Hidden Scope

Initial task (052) seemed like simple Deployment agent fix. Critical assessment revealed Development and Validation agents had identical issue, expanding scope appropriately.

**Lesson:** Always question premises. "Fix X" might really be "Fix X, Y, and Z for the same reason."

---

## Next Steps

**Immediate:**
- Tasks 049, 051, 052 are complete and closed
- Changes validated and tested
- Documentation updated

**Release Impact:**
- Removes three release blockers (049, 051, 052)
- v0.5.0 release moves closer to completion
- Remaining blockers: 050 (Coverage Prompt redesign), 053 (Validation frontmatter updates)

**Future Considerations:**
- Monitor phase reports to verify agents execute CLI commands correctly
- Consider adding automated verification (lint rule checking for command execution in reports)
- Watch for similar patterns in specification agents (do they need explicit directives?)

---

## Related Work

**Tasks completed in this session:**
- Task 049: Fix Development Agent CLI Directive
- Task 051: Fix Validation Agent CLI Directive
- Task 052: Fix Deployment Agent CLI Directive (Preventive)

**Related tasks:**
- Task 048: E2E Agent Workflow Testing (discovered the issue)
- Task 050: Redesign Coverage Prompt (remaining blocker)
- Task 053: Fix Validation Frontmatter Updates (remaining blocker)

**Pattern established:** ALL implementation agents MUST execute `smaqit plan --phase=[PHASE]` as first action with imperative phrasing and output documentation requirement.

---

## Reference

This session establishes the authoritative pattern for implementation agent CLI query directives. The pattern is now embedded in:
1. All three implementation agents (Level 2)
2. Implementation agent template (Level 1)
3. This history document (reference)

**Pattern:** `Execute \`smaqit plan --phase=[PHASE]\` as the first action and process ONLY the specs returned`

**Output requirement:** `[Phase] report MUST document the output of \`smaqit plan --phase=[PHASE]\` command`

**Applies to:** Development, Deployment, Validation agents (all implementation agents)
