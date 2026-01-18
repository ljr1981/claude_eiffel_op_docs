# F03: Fix Specification and/or Contract

## Context
Phase: **FIX** (2 of 3)
Scenario: Bug's root cause is in SPECIFICATION or CONTRACT layer

## Input Required
- Classification from F02 (identifies root layer)
- Source code
- Specifications
- Problem domain

## Prompt

```
TASK: Fix the specification and/or contract layer issues identified in F02.

INPUT:
{classification_from_F02}
{specifications}
{eiffel_source_code}
{problem_domain}

═══════════════════════════════════════════════════════════════
FIX ORDER: Always top-down (Spec before Contract)
═══════════════════════════════════════════════════════════════

1. IF SPECIFICATION LAYER NEEDS FIX:
   - Clarify ambiguous requirements
   - Add missing domain rules
   - Correct wrong domain assumptions
   - Then cascade to contract and code

2. IF CONTRACT LAYER NEEDS FIX:
   - Add absent contract
   - Fix malformed contract
   - Align contract with spec and domain

═══════════════════════════════════════════════════════════════
SPECIFICATION FIXES
═══════════════════════════════════════════════════════════════

ABSENT SPEC → ADD:
  Document what the feature should do
  Document domain rules it must respect
  Document edge cases and error conditions

MALFORMED SPEC → CORRECT:
  Identify what spec says vs what domain requires
  Fix spec to match domain truth
  Document the correction

═══════════════════════════════════════════════════════════════
CONTRACT FIXES
═══════════════════════════════════════════════════════════════

CONTRACT ADDITION RULES:

1. PRECONDITION ADDITION
   Purpose: Reject input that causes bug
   Effect: Callers must change to satisfy

   Pattern:
       feature_name (arg: TYPE)
           require
               new_tag: new_condition  -- Prevents bug #{id}
               -- existing preconditions unchanged
           do
               ...
           end

2. POSTCONDITION ADDITION
   Purpose: Catch when feature fails to achieve goal
   Effect: Implementation must satisfy

   Pattern:
       feature_name: RESULT_TYPE
           do
               ...
           ensure
               new_tag: new_condition  -- Enforces correct behavior
               -- existing postconditions unchanged
           end

3. INVARIANT ADDITION
   Purpose: Prevent invalid state that caused bug
   Effect: All features must maintain

   Pattern:
       invariant
           new_tag: new_condition  -- Prevents state corruption

CONTRACT QUALITY CHECKLIST:
[ ] Tagged with descriptive name
[ ] Expression is BOOLEAN
[ ] Only references queries (no side effects)
[ ] Minimal: doesn't over-constrain
[ ] Sufficient: actually prevents the bug

OUTPUT FORMAT:
CONTRACT TYPE: [PRECONDITION | POSTCONDITION | INVARIANT]
LOCATION: {class.feature or class}

BEFORE:
```eiffel
{existing code}
```

AFTER:
```eiffel
{code with new contract}
```

CONTRACT ADDED:
    {tag}: {expression}

RATIONALE:
    This prevents the bug because: {explanation}

CALLER IMPACT (if precondition):
    Callers that may need updates:
    - {caller1}: {what they must do}
    - {caller2}: {what they must do}

COMPILE CHECK:
    [ ] Contract compiles
    [ ] Contract uses only exported queries
    [ ] Expression type is BOOLEAN
```

═══════════════════════════════════════════════════════════════
LAYER ALIGNMENT VERIFICATION
═══════════════════════════════════════════════════════════════

After fixing spec and/or contract:
[ ] Spec matches domain? (domain is truth)
[ ] Contract matches spec?
[ ] Contract matches domain?
[ ] Contract is machine-checkable form of spec?

OUTPUT FORMAT:
LAYER FIXED: [SPECIFICATION | CONTRACT | BOTH]

IF SPECIFICATION FIXED:
  BEFORE: {old spec or "absent"}
  AFTER: {new spec}
  DOMAIN JUSTIFICATION: {why this is correct}

IF CONTRACT FIXED:
  CONTRACT TYPE: [PRECONDITION | POSTCONDITION | INVARIANT]
  LOCATION: {class.feature or class}
  BEFORE: {existing code or "absent"}
  AFTER: {code with new/fixed contract}
  SPEC ALIGNMENT: {how contract implements spec}
  DOMAIN ALIGNMENT: {how contract enforces domain rule}

CASCADING CHANGES REQUIRED:
  [ ] Code needs update to satisfy new contract?
  [ ] Tests need update to verify new behavior?
  [ ] Other contracts affected?
```

## Success Criteria
- Spec and/or contract fixed
- Fixes align with domain truth
- Contract implements spec correctly
- Cascading changes identified

## Next Step
→ F04-FIX-CODE.md (if code layer also needs fixing)
→ F05-VERIFY-FIX.md (if only spec/contract needed fixing)
