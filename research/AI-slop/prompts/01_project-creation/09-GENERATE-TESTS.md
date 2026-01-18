# 09: Generate Test Cases (Build Test Layer)

## Context
Working Hat: **VERIFICATION** (Phase 3 of 4)
Layer Being Built: **TEST LAYER**

## Input Required
- Specification from step 01
- Contracts from steps 03-05
- Working implementation from step 06
- Problem domain description

## Prompt

```
TASK: Build the TEST LAYER by generating tests that verify ALL layers.

INPUT:
{specification}
{class_with_contracts}
{problem_domain}

═══════════════════════════════════════════════════════════════
TEST LAYER CONSTRUCTION
Tests must verify: Spec, Contracts, Code, AND Domain alignment
═══════════════════════════════════════════════════════════════

TEST CATEGORIES:

1. SPECIFICATION VERIFICATION TESTS
   - Test each spec'd behavior exists and works
   - Test spec'd edge cases are handled
   - Test spec'd error conditions produce correct results
   - Expected results come from SPEC, not code

2. CONTRACT BOUNDARY TESTS
   - Precondition: Just inside valid range (should pass)
   - Precondition: Just outside valid range (should violate)
   - Postcondition: Verify promised properties hold
   - Invariant: Stress with sequences that might break it

3. DOMAIN ALIGNMENT TESTS
   - Test domain rules are enforced
   - Test domain state transitions are valid
   - Test domain calculations are correct
   - Expected results come from DOMAIN KNOWLEDGE

4. EDGE CASES (from all layers)
   - Empty collections
   - Single element
   - Maximum capacity
   - Boundary integers (0, 1, -1, max_int)
   - Domain-specific boundaries

═══════════════════════════════════════════════════════════════
TEST-LAYER ALIGNMENT CHECKS
═══════════════════════════════════════════════════════════════

FOR EACH TEST:
[ ] Derives expected value from spec (not from code)?
[ ] Would fail if code violates spec?
[ ] Would fail if code violates domain rules?
[ ] Would fail if code violates contract?

MALFORMED TEST PATTERNS TO AVOID:
- Test that passes for wrong reasons (weak assertion)
- Test that expects implementation detail, not behavior
- Test that would pass even if feature is broken
- Test whose expected value came from reading the code

OUTPUT FORMAT (Eiffel test class):
class TEST_{CLASS_NAME}

inherit
    TEST_SET_BASE

feature -- Specification Tests

    test_{feature}_spec_behavior
        -- Tests spec: "{quoted spec text}"
        local
            l_obj: {CLASS_NAME}
        do
            create l_obj.make
            l_obj.{feature} (valid_args)
            assert ("spec_says_X", result_matches_spec)
        end

feature -- Contract Tests

    test_{feature}_precondition_boundary
        -- Tests precondition: {contract_tag}
        ...
    end

    test_{feature}_postcondition_holds
        -- Tests postcondition: {contract_tag}
        ...
    end

feature -- Domain Tests

    test_{feature}_domain_rule
        -- Tests domain rule: "{rule}"
        ...
    end

end

RULES:
- Comment each test with WHAT it verifies (spec, contract, domain)
- Expected values come from spec/domain, NEVER from code
- Test name describes the layer and property being verified
- Assert messages reference spec/contract/domain source
```

## Success Criteria
- Every spec'd behavior has a test
- Every precondition has boundary test
- Every postcondition verified in at least one test
- Domain rules have explicit tests
- Test expected values traceable to spec/domain (not code)

## Next Step
→ 10-ITERATE-REFINE.md
