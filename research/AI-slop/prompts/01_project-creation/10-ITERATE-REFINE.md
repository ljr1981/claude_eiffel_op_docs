# 10: Iterate and Refine (Route to Correct Layer)

## Context
Working Hat: **ITERATION** (Phase 4 of 4)
Purpose: Identify which LAYER has the defect and route there

## Input Required
- Test results
- Contract violations encountered
- Implementation issues found
- Specification from step 01

## Prompt

```
TASK: Analyze failures and route to the correct LAYER for fixing.

INPUT:
{test_results}
{violations_encountered}
{implementation_issues}
{specification}

═══════════════════════════════════════════════════════════════
FAILURE LAYER IDENTIFICATION
Which layer is the SOURCE of the problem?
═══════════════════════════════════════════════════════════════

LAYER DIAGNOSIS MATRIX:

| Symptom | Root Layer | Action |
|---------|------------|--------|
| Spec ambiguous/wrong | SPECIFICATION | Fix spec → cascade changes |
| Contract contradicts spec | CONTRACT | Fix contract to match spec |
| Contract contradicts domain | CONTRACT | Fix contract to match domain |
| Code violates contract | CODE | Fix code to satisfy contract |
| Code wrong but contract passes | CONTRACT | Contract is too weak |
| Test expects wrong result | TEST | Fix test to expect spec'd result |
| Test passes but behavior wrong | TEST | Test is too weak |

═══════════════════════════════════════════════════════════════
CROSS-LAYER ALIGNMENT CHECKS
═══════════════════════════════════════════════════════════════

For each failure, check alignment:
[ ] Does spec match domain? (if not → fix spec)
[ ] Does contract match spec? (if not → fix contract)
[ ] Does code satisfy contract? (if not → fix code)
[ ] Does test expect what spec says? (if not → fix test)

═══════════════════════════════════════════════════════════════
FIX ORDER (always top-down)
═══════════════════════════════════════════════════════════════

1. SPECIFICATION issues first (return to step 01)
   - Ambiguous requirements
   - Missing domain rules
   - Wrong domain model

2. CONTRACT issues second (return to steps 03-05)
   - Contract doesn't match spec
   - Contract doesn't match domain
   - Contract too weak/strong

3. CODE issues third (return to step 06)
   - Code doesn't satisfy contract
   - Algorithm wrong for domain

4. TEST issues last (return to step 09)
   - Test expects wrong result
   - Test doesn't verify spec'd behavior
   - Test is too weak

OUTPUT FORMAT:
ISSUE: [description]
ROOT LAYER: [SPECIFICATION | CONTRACT | CODE | TEST]
ALIGNMENT BREAK: {layer X} contradicts {layer Y}
ACTION: [specific fix]
RETURN TO STEP: [01 | 03-05 | 06 | 09 | DONE]
RATIONALE: [why this layer, not another]

COMPLETION CRITERIA:
- All layers aligned
- All tests pass
- No contract violations at runtime
- Code compiles with zero warnings
- Spec, Contract, Code, Test all consistent
```

## Success Criteria
- Root layer identified for each failure
- Cross-layer alignments verified
- Top-down fix order followed
- All issues routed to correct step

## Completion
When all layers aligned and all tests pass → **DONE**

Return to:
- 01 for specification fixes
- 03-05 for contract fixes
- 06 for implementation fixes
- 09 for test fixes
