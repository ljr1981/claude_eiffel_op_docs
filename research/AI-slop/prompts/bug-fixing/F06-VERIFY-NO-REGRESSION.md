# F06: Verify No Regression

## Context
Phase: **VALIDATE** (3 of 3)
Scenario: Fix verified, checking for side effects

## Input Required
- Fixed code
- Existing test suite
- Test results

## Prompt

```
TASK: Verify the fix doesn't break existing functionality.

INPUT:
{fixed_code}
{test_suite_results}

REGRESSION CHECK PROCESS:

1. RUN FULL TEST SUITE
   - All existing tests
   - With assertions enabled
   - Note any failures

2. ANALYZE FAILURES
   For each failure:
   - Was it passing before the fix?
   - Is it related to the fixed code?
   - Is it a true regression or test bug?

3. CATEGORIZE FAILURES

   TRUE REGRESSION:
   - Test was passing
   - Now fails due to fix
   - Fix broke something
   - Action: Revise fix

   REVEALED BUG:
   - Test was passing incorrectly
   - New contract catches it
   - Fix exposed latent bug
   - Action: Fix revealed bug too

   TEST BUG:
   - Test assumed buggy behavior
   - Fix is correct
   - Test needs update
   - Action: Update test

   UNRELATED:
   - Test fails but not due to fix
   - Flaky or environment issue
   - Action: Investigate separately

OUTPUT FORMAT:
TEST SUITE RESULTS:
    Total tests: {N}
    Passed: {N}
    Failed: {N}
    Previously failing: {N}

NEW FAILURES: {count}

For each new failure:

FAILURE 1:
    Test: {test_name}
    Error: {error message}
    Category: [TRUE_REGRESSION | REVEALED_BUG | TEST_BUG | UNRELATED]
    Analysis: {why it failed}
    Action: {what to do}

REGRESSION STATUS:
    [CLEAN: No regressions]
    [REGRESSIONS: {count} true regressions found]
    [REVEALED: {count} latent bugs exposed]

If CLEAN: Proceed to F07
If REGRESSIONS: Return to F04, revise fix
If REVEALED: Document additional bugs for separate fixes
```

## Success Criteria
- All existing tests run
- Failures categorized
- True regressions identified
- Plan for each failure

## Next Step
→ F07-ADD-REGRESSION-TEST.md (if clean)
→ F04-FIX-IMPLEMENTATION.md (if regressions)
