# X08: Surgical Fixes

## Context
Phase: **FIX** (2 of 4)
Purpose: Fix bugs minimally and precisely

## Mindset

**Fix the bug. Only the bug. Nothing but the bug.**

Surgical fixing means:
- Change as little code as possible
- Don't refactor
- Don't "improve" nearby code
- Don't add features
- Make the minimal change that fixes the issue

Every extra change is a risk of regression.

## Input Required
- Triage report from X07
- Source code
- Test suite

## Prompt

```
TASK: Fix prioritized bugs with minimal, surgical changes.

INPUT:
{triage_report_from_X07}
{source_files}
{test_suite}

RULE: For each fix, change the MINIMUM code necessary.

═══════════════════════════════════════════════════════════════
FIX PROCEDURE
For each bug in priority order
═══════════════════════════════════════════════════════════════

For each P1, then P2, then P3 finding:

1. READ the bug description
2. LOCATE the exact code causing the issue
3. UNDERSTAND the root cause
4. DESIGN the minimal fix
5. IMPLEMENT the fix
6. TEST that the specific bug is fixed
7. TEST that nothing else broke
8. DOCUMENT the fix

═══════════════════════════════════════════════════════════════
FIX: {FINDING-ID}
═══════════════════════════════════════════════════════════════

FINDING: {id}
  Description: {what's wrong}
  Priority: P{n}
  Location: {class}.{feature}:{line}

ROOT CAUSE ANALYSIS:
  The bug occurs because: {explanation}
  The fix needs to: {what change is needed}

MINIMAL FIX:

BEFORE:
```eiffel
{exact code being changed}
```

AFTER:
```eiffel
{fixed code}
```

CHANGE SUMMARY:
  Files changed: {count}
  Lines changed: {count}
  Nature of change: {description}

VERIFICATION:
  - [ ] Bug-specific test now passes
  - [ ] Existing tests still pass
  - [ ] Contract that caught it now satisfied
  - [ ] No new warnings introduced

═══════════════════════════════════════════════════════════════
FIX ANTI-PATTERNS
What NOT to do while fixing
═══════════════════════════════════════════════════════════════

FORBIDDEN during surgical fixing:

❌ "While I'm here, let me also..."
❌ "This variable name is bad, I'll rename it..."
❌ "This could be refactored to..."
❌ "I'll add some comments to clarify..."
❌ "This function is too long, I'll split it..."
❌ "I see another bug nearby..."

ALLOWED:

✓ Fix exactly what the bug report describes
✓ Add a test that catches this specific bug
✓ Add a contract that prevents this specific bug

═══════════════════════════════════════════════════════════════
FIX CATEGORIES
Different bugs need different fix approaches
═══════════════════════════════════════════════════════════════

CRASH FIX:
  - Add null check
  - Add bounds check
  - Add precondition
  - Handle error case

WRONG RESULT FIX:
  - Correct calculation
  - Fix off-by-one
  - Fix comparison operator
  - Add postcondition

STATE CORRUPTION FIX:
  - Ensure atomic update
  - Add state validation
  - Fix incomplete update
  - Add invariant

RESOURCE LEAK FIX:
  - Add cleanup in rescue
  - Ensure release on all paths
  - Add resource tracking

LIMIT FIX:
  - Document the limit
  - Add precondition for limit
  - Improve algorithm (if essential)

═══════════════════════════════════════════════════════════════
REGRESSION PREVENTION
Don't break what works
═══════════════════════════════════════════════════════════════

For each fix:

REGRESSION CHECK:
  - Run full test suite: [PASS | FAIL]
  - If FAIL: {what broke}
  - Fix caused regression: [YES | NO]
  - Regression fixed: [YES | N/A]

If a fix causes regression:
1. STOP
2. Reconsider the fix approach
3. Find alternative minimal fix
4. Never compromise existing functionality

═══════════════════════════════════════════════════════════════
FIX DOCUMENTATION
Record what was changed and why
═══════════════════════════════════════════════════════════════

For each fix:

FIX RECORD: {finding-id}
  Date: {date}
  Bug: {description}
  Root cause: {why it happened}
  Fix: {what was changed}
  Files: {list of files}
  Tests: {what tests verify the fix}
  Verified by: [TEST | CONTRACT | BOTH]

═══════════════════════════════════════════════════════════════
FIX BATCH PROCEDURE
When fixing multiple bugs
═══════════════════════════════════════════════════════════════

1. Fix ONE bug at a time
2. Test after EACH fix
3. Commit after each successful fix (if using VCS)
4. Never batch fixes together
5. If a fix breaks something, revert immediately

FIX SEQUENCE:
  1. Fix {id}: {status}
  2. Fix {id}: {status}
  ...

═══════════════════════════════════════════════════════════════
COMPILE AND TEST
After all fixes
═══════════════════════════════════════════════════════════════

FINAL VERIFICATION:
  - Code compiles: [YES | NO]
  - All original tests pass: [YES | NO]
  - All adversarial tests pass: [YES | NO]
  - All assault contracts satisfied: [YES | NO]

OUTPUT FORMAT:

# SURGICAL FIX REPORT: {library_name}

## Fix Summary
- Bugs fixed: {count}
- Lines changed: {count}
- Files modified: {count}
- Regressions caused: {count}
- Regressions fixed: {count}

## Fixes Applied

### FIX-001: {FINDING-ID}

**Bug**: {description}

**Root Cause**: {explanation}

**Fix**:
```eiffel
-- BEFORE:
{original code}

-- AFTER:
{fixed code}
```

**Changed**: {file}:{lines}

**Verified**: {how verified}

---

### FIX-002: ...

## Deferred Bugs
These bugs were NOT fixed (per triage):
- {FINDING-ID}: {reason}

## Test Results
- Original tests: {pass}/{total}
- Adversarial tests: {pass}/{total}
- All contracts satisfied: [YES | NO]

## Commit Information
{If using VCS, list commits}
```

## Implementation Notes

1. Work through bugs in triage priority order
2. One bug at a time
3. Minimal changes only
4. Test after each fix
5. Document everything
6. Never batch fixes

## Success Criteria
- All P1 bugs fixed
- All P2 bugs fixed (time permitting)
- No regressions introduced
- All fixes documented
- All fixes verified

## Next Step
→ X09-HARDEN-DEFENSES.md
