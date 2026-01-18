# 08: Run and Analyze Contract Violations

## Context
Working Hat: **VERIFICATION** (Phase 3 of 4)

## Input Required
- Compiled executable
- Runtime contract violation output

## Prompt

```
TASK: Analyze contract violations and determine fix location.

INPUT:
{contract_violation_output}
{source_code}

VIOLATION TYPES:

1. PRECONDITION VIOLATION
   Message: "Precondition 'tag_name' violated"
   Location: CALLER is wrong
   Fix: Caller must satisfy precondition before calling

2. POSTCONDITION VIOLATION
   Message: "Postcondition 'tag_name' violated"
   Location: IMPLEMENTATION is wrong
   Fix: Feature body must achieve postcondition

3. INVARIANT VIOLATION
   Message: "Invariant 'tag_name' violated"
   Location: FEATURE that just completed is wrong
   Fix: Feature must restore invariant before exiting

4. CHECK VIOLATION
   Message: "Assertion 'tag_name' violated"
   Location: CHECK statement failed
   Fix: Logic leading to check is wrong

ANALYSIS FORMAT:
VIOLATION: [type] - [tag_name]
FEATURE: [class.feature]
CONTRACT: [the boolean expression]
ACTUAL: [what value/state caused failure]
ROOT CAUSE: [why the contract wasn't satisfied]
FIX LOCATION: [caller | implementation | specification]
FIX: [specific change]

DECISION TREE:
- Precondition violated → Fix caller OR weaken precondition if too strict
- Postcondition violated → Fix implementation OR strengthen postcondition if wrong
- Invariant violated → Fix feature OR fix invariant if wrong
- Contract itself wrong? → Return to specification phase (rare)
```

## Success Criteria
- All violations categorized
- Fix location identified (not just symptom)
- Implementation fixes preferred over contract changes

## Next Step
→ 09-GENERATE-TESTS.md
