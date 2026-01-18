# F05: Verify Fix

## Context
Phase: **VALIDATE** (3 of 3)
Scenario: Fix applied, verifying bug is resolved

## Input Required
- Reproduction test from F01
- Fixed code from F03/F04
- Compiler output

## Prompt

```
TASK: Verify the fix eliminates the original bug.

INPUT:
{reproduction_test}
{fixed_code}

VERIFICATION STEPS:

1. COMPILE
   - Does fixed code compile?
   - Any new errors from contract additions?
   - Any type errors from code changes?

2. RUN REPRODUCTION TEST
   - Execute the test from F01
   - It should now PASS (bug eliminated)

3. VERIFY CORRECT BEHAVIOR
   - Not just "doesn't crash"
   - Actually produces correct result
   - Contracts pass

4. EDGE CASE CHECK
   - Test variations of the bug trigger
   - Similar inputs that should also work now

OUTPUT FORMAT:
COMPILATION:
    Status: [SUCCESS | FAILED]
    If FAILED: {errors and how to fix}

REPRODUCTION TEST:
    Status: [PASS | FAIL]

    If PASS:
        Bug eliminated: YES
        Correct result: {what it now returns/does}

    If FAIL:
        Bug still present: {what's still wrong}
        Action: Return to F04, fix is incomplete

EDGE CASE TESTS:
    Test: {variation 1}
    Result: [PASS | FAIL]

    Test: {variation 2}
    Result: [PASS | FAIL]

CONTRACT STATUS:
    Preconditions: [SATISFIED | VIOLATED: {which}]
    Postconditions: [SATISFIED | VIOLATED: {which}]
    Invariants: [SATISFIED | VIOLATED: {which}]

VERIFICATION RESULT: [FIXED | NOT_FIXED]

If FIXED: Proceed to F06
If NOT_FIXED: Return to F04 with new analysis
```

## Success Criteria
- Code compiles
- Reproduction test passes
- Correct behavior confirmed
- Contracts satisfied

## Next Step
→ F06-VERIFY-NO-REGRESSION.md (if fixed)
→ F04-FIX-IMPLEMENTATION.md (if not fixed)
