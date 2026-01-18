# F02: Classify Fix Layers

## Context
Phase: **ANALYZE** (1 of 3)
Scenario: Bug reproduced, determining which layers need fixing

## Input Required
- Reproduction test from F01
- Root cause (if known from H07, or analyze now)
- Source code, specs, tests

## Prompt

```
TASK: Classify which layers need fixing to resolve this bug.

INPUT:
{reproduction_test}
{root_cause_if_known}
{source_code}
{specifications}
{tests}

═══════════════════════════════════════════════════════════════
LAYER ANALYSIS
For each layer, determine: ABSENT | MALFORMED | CORRECT
═══════════════════════════════════════════════════════════════

1. SPECIFICATION LAYER
   [ ] Is there a spec for this behavior?
   [ ] Does spec correctly describe expected behavior?
   [ ] Does spec match domain requirements?

   Status: [ABSENT | MALFORMED | CORRECT]
   If issue: {description}

2. CONTRACT LAYER
   [ ] Is there a contract for this feature?
   [ ] Does contract match spec?
   [ ] Does contract match domain?

   Status: [ABSENT | MALFORMED | CORRECT]
   If issue: {description}

3. CODE LAYER
   [ ] Does code satisfy contract?
   [ ] Does code match spec?
   [ ] Does code match domain?

   Status: [MALFORMED | CORRECT]
   If malformed: {description}

4. TEST LAYER
   [ ] Is there a test for this behavior?
   [ ] Does test expect correct behavior?
   [ ] Does test match spec/contract?

   Status: [ABSENT | MALFORMED | CORRECT]
   If issue: {description}

═══════════════════════════════════════════════════════════════
ROOT CAUSE IDENTIFICATION
Which layer is the SOURCE of the bug?
═══════════════════════════════════════════════════════════════

ROOT CAUSE LAYER: [SPEC | CONTRACT | CODE | TEST]

Explanation:
- If spec wrong → everything derived from it is wrong
- If contract wrong (spec correct) → contract is root
- If code wrong (contract correct) → code is root
- If test expects wrong thing (code correct) → test is root

═══════════════════════════════════════════════════════════════
FIX PLAN
What needs to change, in order?
═══════════════════════════════════════════════════════════════

FIX ORDER (top-down):
1. [If spec wrong] Fix specification
2. [If contract wrong/absent] Fix/add contract
3. [If code wrong] Fix code
4. [Always] Add/fix test

OUTPUT FORMAT:
LAYER STATUS:
  Specification: [ABSENT | MALFORMED | CORRECT]
  Contract: [ABSENT | MALFORMED | CORRECT]
  Code: [MALFORMED | CORRECT]
  Test: [ABSENT | MALFORMED | CORRECT]

ROOT CAUSE LAYER: {layer}
ROOT CAUSE: {description}

FIX PLAN:
  1. {first layer to fix}: {what to change}
  2. {next layer}: {what to change}
  ...
  N. Test: {add regression test}

SPECIAL CASES:
  [ ] Malformed test codifies bug (test expects buggy behavior)
  [ ] Multiple layers have same bug (cascading malformation)
  [ ] Fix in one layer will "break" tests (tests are wrong)
```

## Success Criteria
- All four layers assessed
- Root cause layer identified
- Fix order determined (top-down)
- Special cases flagged

## Next Step
→ F03-FIX-SPEC-CONTRACT.md (if spec or contract needs fixing)
→ F04-FIX-CODE.md (if only code needs fixing)
