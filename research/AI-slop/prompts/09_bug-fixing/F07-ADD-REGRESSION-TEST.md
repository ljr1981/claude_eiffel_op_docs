# F07: Add Regression Test (Build/Strengthen Test Layer)

## Context
Phase: **VALIDATE** (3 of 3)
Scenario: Fix complete, adding permanent test to TEST LAYER

## Input Required
- Reproduction test from F01
- Specification (what correct behavior is)
- Fixed code
- Test suite
- Problem domain

## Prompt

```
TASK: Add regression test that verifies CORRECT BEHAVIOR (not just absence of bug).

INPUT:
{reproduction_test}
{specification}
{fixed_code}
{problem_domain}

═══════════════════════════════════════════════════════════════
TEST LAYER PRINCIPLE
Tests verify spec/domain behavior, not implementation details
═══════════════════════════════════════════════════════════════

REGRESSION TEST REQUIREMENTS:

1. TEST SPEC'D BEHAVIOR, NOT IMPLEMENTATION
   - Expected value from SPEC, not from code
   - Test what spec says should happen
   - Would fail if spec violated, regardless of how

2. TEST DOMAIN CORRECTNESS
   - Expected value from DOMAIN TRUTH
   - Test domain rules are enforced
   - Test domain-correct results

3. TEST THE BUG SCENARIO
   - Test should fail if bug returns
   - Test should pass with correct behavior
   - Independent of specific fix approach

4. MINIMAL AND FOCUSED
   - One bug, one test
   - Minimal setup
   - Clear assertion

5. SELF-DOCUMENTING
   - Test name describes spec'd behavior being verified
   - Comments explain what spec requires
   - Comments explain domain rule if applicable

6. BOUNDARY COVERAGE
   - Test the exact trigger
   - Test near-trigger (should pass)
   - Test clearly-invalid (precondition check)

OUTPUT FORMAT:
```eiffel
feature -- Regression Tests

    test_bug_{id}_{short_description}
            -- Regression test for bug #{id}: {description}
            -- Spec requires: {what spec says}
            -- Domain rule: {domain rule if applicable}
            -- Fixed: {date}
        local
            l_obj: {CLASS}
        do
            comment ("Verifies: {spec'd behavior}")

            -- Setup: minimal state to trigger scenario
            create l_obj.make
            {setup}

            -- Execute: the operation
            {operation}

            -- Verify: SPEC'D correct behavior (not implementation detail)
            -- Expected value from spec: {where expected value comes from}
            assert ("spec_requires_X", {expected_condition})
            assert ("domain_rule_Y", {domain_condition})
        end

    test_bug_{id}_boundary
            -- Boundary test: near-trigger should work per spec
            -- Spec says: {what spec says about boundary}
        local
            l_obj: {CLASS}
        do
            create l_obj.make
            -- Similar to bug trigger but valid per spec
            {boundary_operation}
            assert ("boundary_per_spec", {condition})
        end
```

TEST PLACEMENT:
    File: {test class file}
    Section: feature -- Regression Tests

DOCUMENTATION:
    - Bug ID: {id}
    - Date fixed: {date}
    - Root layer: {which layer had the bug}
    - Spec requirement: {what spec says}
    - Domain rule: {if applicable}
    - Contract added: {if any}

═══════════════════════════════════════════════════════════════
TEST LAYER QUALITY CHECK
═══════════════════════════════════════════════════════════════

[ ] Expected value derived from spec/domain (NOT from code)?
[ ] Would fail if spec'd behavior violated?
[ ] Comments explain WHAT is being verified?
[ ] Test is independent of implementation details?
[ ] Test would catch bug returning via different code path?

COMPLETE CHECKLIST:
    [ ] Test added to suite
    [ ] Test passes with fix
    [ ] Test would fail without fix (verified conceptually)
    [ ] Test verifies spec'd behavior (not just absence of symptom)
    [ ] Test documented with spec/domain references
    [ ] Bug report updated/closed
```

## Success Criteria
- Regression test added
- Test is minimal and focused
- Test is documented
- Bug cannot return silently

## Completion

Bug fix workflow complete:
1. ✓ Bug reproduced (F01)
2. ✓ Fix type classified (F02)
3. ✓ Contract added if needed (F03)
4. ✓ Code fixed if needed (F04)
5. ✓ Fix verified (F05)
6. ✓ No regressions (F06)
7. ✓ Regression test added (F07)

The bug is fixed and protected against recurrence.
