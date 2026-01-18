# F01: Reproduce Bug

## Context
Phase: **ANALYZE** (1 of 3)
Scenario: Known bug, need to confirm before fixing

## Input Required
- Bug report (from H08 or user-provided)
- Source code
- Steps to reproduce

## Prompt

```
TASK: Reproduce the reported bug to confirm it exists as described.

INPUT:
Bug Report:
{bug_description}

Steps to Reproduce:
{reproduction_steps}

Source Code:
{eiffel_source_code}

REPRODUCTION PROCESS:

1. UNDERSTAND THE REPORT
   - What is the expected behavior?
   - What is the actual behavior?
   - What triggers it?

2. CREATE REPRODUCTION TEST
   - Translate steps into Eiffel test
   - Minimal setup
   - Clear assertion of failure

3. EXECUTE AND OBSERVE
   - Does it fail as described?
   - Same failure mode?
   - Same location?

4. CONFIRM OR DISPUTE

OUTPUT FORMAT:
BUG SUMMARY: {one line}

REPRODUCTION TEST:
```eiffel
test_reproduce_bug_{id}
        -- Reproduces: {bug summary}
    local
        l_obj: {CLASS}
    do
        -- Setup
        create l_obj.make
        {setup_steps}

        -- Trigger
        {triggering_operation}

        -- This should fail if bug exists
        assert ("expected_behavior", {condition})
    end
```

EXECUTION RESULT:
  Status: [REPRODUCED | NOT REPRODUCED | DIFFERENT FAILURE]

  If REPRODUCED:
    Failure matches report: YES
    Ready to proceed to F02

  If NOT REPRODUCED:
    Possible reasons:
    - {reason 1}
    - {reason 2}
    Action: Clarify with reporter / check environment

  If DIFFERENT FAILURE:
    Expected: {from report}
    Actual: {what happened}
    Action: Report discrepancy, may be different bug
```

## Success Criteria
- Bug reproduced exactly as described
- Test case captures the bug
- Failure confirmed

## Next Step
→ F02-CLASSIFY-FIX-TYPE.md (if reproduced)
→ Return to reporter (if not reproduced)
