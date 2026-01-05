# Session 027: Task 050 - Redesign Coverage Prompt

**Date:** 2026-01-05  
**Task:** 050 - Redesign Coverage Prompt  
**Status:** Completed  
**Previous Session:** [026_github_releases_installation_2026-01-02.md](026_github_releases_installation_2026-01-02.md)

## Objective

Redesign Coverage prompt to focus ONLY on verification preferences (test environment, tooling, thresholds), removing sections that ask for requirements already present in upstream specs. This resolves the contradiction between Coverage prompt structure (asking for requirements) and Coverage agent directives (MUST NOT add requirements).

## Context

**Problem Identified in Task 048 (E2E Testing):**

Coverage prompt asked users to specify:
- Performance Benchmarks
- Security Requirements
- Test Scope
- Integration Points
- Verification Requirements

But Coverage agent directives state:
- "MUST NOT add requirements not present in upstream specs"
- "Reference every acceptance criterion from upstream specs by ID"

**Evidence:** E2E testing proved Coverage agent generated 652-line spec with 100% traceability (92/92 testable requirements mapped) from minimal prompt (tooling/thresholds only). This validated that Coverage CAN work as pure traceability mapping layer.

**Root Cause:** Coverage layer design ambiguity - prompt suggested hybrid model (adds requirements) while framework mandated pure mapping (derives from upstream).

## Solution Architecture

**Principle Established:** Coverage is pure traceability mapping layer

- **Requirements Source:** ALL upstream specs (Business, Functional, Stack, Infrastructure)
- **Prompt Purpose:** Verification strategy preferences ONLY (tooling, environment, thresholds)
- **Agent Role:** Enumerate all acceptance criteria from upstream and map to test cases

**What Changed:**
- Prompt: From "requirements" to "verification preferences"
- Sections: From 7 (5 requirement + 2 preference) to 2 (preference only)
- Agent: Strengthened pure traceability directives
- Framework: Clarified Coverage input is preferences, context is source of requirements

## Implementation

### Level 0: Framework (LAYERS.md)

**Changed:**
```diff
- **Input:** User verification requirements (test scope, performance benchmarks, security requirements)
+ **Input:** User verification preferences (test environment, tooling, acceptance thresholds)

- **Context:** All layer specs (Business, Functional, Stack, Infrastructure)
+ **Context:** All layer specs (Business, Functional, Stack, Infrastructure) — source of all requirements
```

**Rationale:** Establish principle at foundation level - Coverage receives preferences, derives requirements from context.

### Level 1: Templates (specification-prompt.template.md)

**Removed Sections:**
- Test Scope
- Performance Benchmarks
- Security Requirements
- Integration Points
- Verification Requirements

**Kept Sections:**
- Test Environment
- Acceptance Thresholds

**Added Note:**
```markdown
**Note:** Coverage prompt is optional. The agent derives all test requirements from upstream specs 
(Business, Functional, Stack, Infrastructure). This prompt specifies ONLY verification strategy 
preferences (tooling, environment, thresholds), NOT requirements.
```

**Rationale:** Template guides future prompt creation with correct structure.

### Level 2: Source Prompt (prompts/smaqit.coverage.prompt.md)

**Structural Changes:**
- Header: "Requirements" → "Verification Preferences"
- Description: Updated to clarify agent derives from upstream specs
- Added: "This prompt is optional" guidance
- Removed: 5 requirement sections (37 lines → 27 lines)

**Example Updates:**
```markdown
<!-- Example: "100% of testable acceptance criteria must have test cases" -->
<!-- Example: "All integration tests must pass before deployment" -->
<!-- Example: "Performance tests within 10% of baseline" -->
```

**Rationale:** Users see simplified prompt, understand they don't need to duplicate requirements.

### Level 2: Agent (agents/smaqit.coverage.agent.md)

**Role Update:**
```diff
- Specification agent for the Coverage layer. Translates prompt file requirements into 
  precise, testable specifications.
+ Specification agent for the Coverage layer. Enumerates all acceptance criteria from 
  upstream specifications and maps each to executable test cases. Uses prompt file for 
  verification strategy preferences only.
```

**Input Section Enhancement:**
- Clarified: "Prompt is optional"
- Added: "NOT requirements - all requirements come from upstream specs"
- Enhanced: Upstream specs labeled "source of ALL requirements"
- Added: "Critical" guidance block
- Updated: Conflict resolution (ignore prompt duplications)

**MUST Directives Strengthened:**
- Added: "Scan ALL upstream specs to enumerate every acceptance criterion by ID"
- Added: "Calculate coverage: (mapped criteria / total testable criteria) × 100%"
- Added: "Use prompt ONLY for verification strategy preferences"

**MUST NOT Added:**
- "Treat prompt content as source of requirements"

**Rationale:** Agent clearly instructed to derive from upstream, use prompt for preferences only.

## Files Modified

**Framework (Level 0):**
- `framework/LAYERS.md` - 2 lines changed

**Templates (Level 1):**
- `templates/prompts/specification-prompt.template.md` - Coverage section simplified (41 → 22 lines)

**Agents/Prompts (Level 2):**
- `prompts/smaqit.coverage.prompt.md` - Removed 5 sections (37 → 27 lines)
- `agents/smaqit.coverage.agent.md` - Enhanced directives (40 lines changed)

**Documentation:**
- `docs/tasks/050_redesign_coverage_prompt.md` - Marked completed
- `docs/tasks/PLANNING.md` - Moved task 050 to completed

**Total:** 4 source files + 2 documentation files

**Net Change:** -32 lines (simplified from requirement-adding to preference-only)

## Testing and Validation

### Build Validation
```bash
cd installer && make build
# Result: ✅ Build successful
```

### Installation Test
```bash
mkdir -p test_task50 && cd test_task50
../dist/smaqit init
# Result: ✅ Prompt file copied with updated structure
cat .github/prompts/smaqit.coverage.prompt.md
# Verified: Only 2 sections (Test Environment, Acceptance Thresholds)
```

### Grep Validation
```bash
grep -r "Performance Benchmarks\|Security Requirements\|Integration Points" \
  framework/ agents/ prompts/
# Result: Exit code 1 (no matches) - ✅ Requirement sections removed
```

### Consistency Check
- ✅ Framework (L0) establishes principle
- ✅ Template (L1) guides with correct structure
- ✅ Prompt (L2) follows template
- ✅ Agent (L2) aligns with framework

## Key Decisions

### 1. Coverage Layer Purpose

**Decision:** Coverage is pure traceability mapping, not requirement-adding layer

**Alternatives Considered:**
- Hybrid model: Coverage adds verification-specific requirements (performance, security)
- Current model: Coverage derives all requirements from upstream specs

**Rationale:**
- Framework LAYERS.md already states "MUST NOT add requirements not present in upstream specs"
- E2E testing validated agent CAN generate comprehensive coverage from upstream only
- Avoids requirement duplication/conflict risk
- Aligns with Coverage layer purpose: "Enumerate every acceptance criterion and map it to verification test"

### 2. Prompt Optionality

**Decision:** Make Coverage prompt optional, not required

**Rationale:**
- Agent can generate comprehensive coverage with minimal or no input
- All requirements exist in upstream specs
- Prompt useful for tooling preferences (pytest vs unittest, GitHub Actions vs local)
- Reduces cognitive load on users

### 3. What Belongs in Coverage Prompt

**Kept:**
- Test Environment (where/how tests run)
- Acceptance Thresholds (coverage % goals, pass criteria)

**Removed:**
- Performance Benchmarks → Business/Infrastructure specs
- Security Requirements → Functional/Infrastructure specs
- Test Scope → Derived from upstream acceptance criteria
- Integration Points → Infrastructure specs
- Verification Requirements → Derived from upstream acceptance criteria

**Rationale:** Prompt provides strategy preferences, not requirements. Requirements come from upstream.

### 4. Level Cascade Approach

**Decision:** Work Level 0 → Level 1 → Level 2

**Rationale:**
- Framework establishes principle first
- Templates encode structure guidance
- Agents/Prompts implement with specifics
- Ensures consistency across levels

## Impact Analysis

### User Experience
- **Before:** Users duplicated requirements in Coverage prompt (performance, security, etc.)
- **After:** Users specify minimal preferences (tooling, thresholds), agent derives rest
- **Improvement:** Reduced cognitive load, no duplication, clear separation of concerns

### Agent Behavior
- **Before:** Ambiguity - should agent use prompt requirements or upstream specs?
- **After:** Clear directive - derive from upstream, use prompt for preferences only
- **Improvement:** Eliminates contradiction, strengthens traceability

### Framework Alignment
- **Before:** Prompt structure contradicted agent directives and framework principles
- **After:** All levels (0, 1, 2) consistent - Coverage is pure traceability mapping
- **Improvement:** Framework integrity restored, no more design ambiguity

### Traceability
- **Before:** Unclear if tests verify prompt requirements or spec requirements
- **After:** Tests explicitly map to upstream acceptance criteria by ID
- **Improvement:** Complete traceability chain, audit trail preserved

## Lessons Learned

### 1. E2E Testing Reveals Design Issues

E2E testing (Task 048) exposed the Coverage prompt contradiction. Without actually running the agent with minimal input, the design ambiguity persisted unnoticed. **Takeaway:** Practical testing validates theoretical design.

### 2. Level Cascade Prevents Inconsistency

Working Level 0 → Level 1 → Level 2 ensured changes propagated correctly. Framework principle guided template structure, which guided agent directives. **Takeaway:** Respect smaqit's own level hierarchy when modifying smaqit.

### 3. Simplification Improves UX

Removing 5 sections from Coverage prompt didn't reduce capability - E2E testing proved agent works better with minimal input. **Takeaway:** More structure isn't always better; question every requirement field.

### 4. Framework Integrity Over Backward Compatibility

Coverage prompt structure existed since initial design, but contradicted framework principles. Fixing it required breaking existing prompt structure. **Takeaway:** Framework integrity takes precedence over preserving incorrect patterns.

## Related Tasks

- **Task 048** (Completed) - E2E Agent Workflow Testing (discovered this issue)
- **Task 049** (Active) - Fix Development Agent CLI Directive
- **Task 051** (Active) - Fix Validation Agent CLI Directive
- **Task 052** (Active) - Fix Deployment Agent CLI Directive

## Next Steps

With Task 050 complete, remaining release blockers are:
1. Task 049 - Development agent CLI directive
2. Task 051 - Validation agent CLI directive
3. Task 053 - Validation frontmatter updates

Once these are resolved, v0.5.0 release can proceed with stateful specifications + corrected Coverage layer.

## Session Metrics

- **Duration:** ~1.5 hours
- **Tasks Completed:** 1 (Task 050)
- **Files Modified:** 6 (4 source + 2 documentation)
- **Lines Changed:** -32 net (simplification)
- **Acceptance Criteria:** 12/12 met
- **Testing:** Build, installation, grep validation, consistency check - all passed
- **Release Impact:** Removes one blocker for v0.5.0

## Appendix: Correct Coverage Workflow

**User fills minimal Coverage prompt (optional):**
```markdown
## Verification Preferences

### Test Environment
GitHub Actions on push to main branch

### Acceptance Thresholds
100% of testable acceptance criteria must have test cases
```

**Coverage agent executes:**
1. Scan ALL upstream specs (specs/business/, specs/functional/, specs/stack/, specs/infrastructure/)
2. Extract ALL acceptance criteria with IDs (BUS-*, FUN-*, STK-*, INF-*)
3. For each criterion, define test case: COV-[ID] → Test → Expected Outcome
4. Calculate coverage: (mapped criteria / total testable criteria) × 100%
5. Flag untestable criteria with justification
6. Output: Coverage spec is comprehensive test plan proving 100% traceability

**Result:** 100% traceability, no requirement duplication, clear verification strategy.
