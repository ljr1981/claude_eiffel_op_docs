# H07: Confirm Root Cause

## Context
Phase: **VERIFY** (3 of 3)
Scenario: Exploit succeeded, trace to root cause

## Input Required
- Failing test from H06
- Malformation type identified
- Source code

## Prompt

```
TASK: Trace confirmed bug to its root cause, distinguishing symptom from source.

INPUT:
{failing_test}
{malformation_type}
{eiffel_source_code}

═══════════════════════════════════════════════════════════════
ROOT CAUSE BY MALFORMATION TYPE
═══════════════════════════════════════════════════════════════

IF CONTRACT ABSENT:
  Root: Missing contract at {location}
  Symptom: {what the test exposed}
  Fix type: ADD_CONTRACT

IF CONTRACT MALFORMED:
  Root: Contract is semantically wrong
  - Contract says: {what contract expresses}
  - Should say: {what domain requires}
  - Gap: {semantic difference}
  Fix type: FIX_CONTRACT

IF CODE MALFORMED:
  Root: Code violates domain semantics
  - Code does: {what code actually does}
  - Domain requires: {what it should do}
  - Gap: {semantic difference}
  Fix type: FIX_CODE

IF COMBINED:
  Root 1: Contract malformation at {location}
  Root 2: Code malformation at {location}
  Order: Fix contract first (defines correctness), then code
  Fix type: FIX_BOTH

═══════════════════════════════════════════════════════════════
SEMANTIC ANALYSIS
═══════════════════════════════════════════════════════════════

1. WHAT IS THE FEATURE'S TRUE PURPOSE?
   - Name suggests: {X}
   - Domain requires: {Y}
   - Contract states: {Z}
   - Code implements: {W}

   Alignment check: X = Y = Z = W?
   If not: Which is wrong?

2. WHAT IS THE CORRECT CONTRACT?
   Given the feature's true purpose:
   - Precondition should be: {correct require}
   - Postcondition should be: {correct ensure}

3. WHAT IS THE CORRECT CODE?
   Given the correct contract:
   - Implementation should: {correct behavior}

OUTPUT FORMAT:
FAILURE POINT: {class.feature:line}
MALFORMATION TYPE: [ABSENT | MALFORMED_CONTRACT | MALFORMED_CODE | COMBINED]

SEMANTIC TRACE:
  Feature purpose: {true purpose}
  Contract states: {what contract says, or "ABSENT"}
  Code does: {what code does}
  Misalignment: {where chain breaks}

ROOT CAUSE:
  Location: {class.feature:line}
  Category: {malformation type}

  IF CONTRACT ISSUE:
    Current contract: {existing or absent}
    Correct contract: {what it should be}
    Semantic gap: {why current is wrong}

  IF CODE ISSUE:
    Current behavior: {what code does}
    Correct behavior: {what domain requires}
    Semantic gap: {why current is wrong}

WHY IT'S WRONG (DOMAIN TERMS):
  {Explanation in problem domain language}

FIX DIRECTION:
  [ADD_CONTRACT | FIX_CONTRACT | FIX_CODE | FIX_BOTH]
  Primary fix: {the main change}
  Secondary fix: {if BOTH}
```

## Success Criteria
- Malformation type confirmed
- Semantic gap articulated
- Correct contract/code identified
- Fix direction clear

## Next Step
→ H08-DOCUMENT-FINDING.md
