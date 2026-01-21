# H02: Analyze Semantic Alignment

## Context
Phase: **SURVEY** (1 of 3)
Scenario: Bug hunting - deep analysis of layer alignment

## Input Required
- Source code, tests, specifications
- Risk areas from H01
- Problem domain description

## Prompt

```
TASK: Analyze semantic alignment across all layers for each risk area.

INPUT:
{all_artifacts}
{risk_areas}
{problem_domain}

═══════════════════════════════════════════════════════════════
THE ALIGNMENT CHAIN
Each layer should derive from and align with the layer above.
═══════════════════════════════════════════════════════════════

    DOMAIN TRUTH (external reality)
           ↓
    SPECIFICATION (documented intent)
           ↓
    CONTRACT (machine-checkable intent)
           ↓
    CODE (implementation of intent)
           ↓
    TEST (verification of intent)

FOR EACH FEATURE, TRACE THE CHAIN:

1. DOMAIN TRUTH
   Q: What should this feature do in the real world?
   Q: What domain rules apply?

2. SPECIFICATION ALIGNMENT
   Q: Is there a spec? [YES | NO | PARTIAL]
   Q: Does spec match domain truth? [ALIGNED | MISALIGNED | ABSENT]
   If misaligned: {the gap}

3. CONTRACT ALIGNMENT
   Q: Are there contracts? [YES | NO | PARTIAL]
   Q: Do contracts match spec? [ALIGNED | MISALIGNED | ABSENT]
   Q: Do contracts match domain? [ALIGNED | MISALIGNED | ABSENT]
   If misaligned: {the gap}

4. CODE ALIGNMENT
   Q: Does code satisfy contracts? [YES | NO | UNKNOWN]
   Q: Does code match spec? [ALIGNED | MISALIGNED | ABSENT]
   Q: Does code match domain? [ALIGNED | MISALIGNED]
   If misaligned: {the gap}

5. TEST ALIGNMENT
   Q: Are there tests? [YES | NO | PARTIAL]
   Q: Do tests verify contracts? [YES | NO | PARTIAL]
   Q: Do tests expect correct behavior? [YES | NO | UNKNOWN]
   Q: Do tests match spec? [ALIGNED | MISALIGNED | ABSENT]
   If misaligned: {the gap}

═══════════════════════════════════════════════════════════════
MALFORMATION ANALYSIS BY LAYER
═══════════════════════════════════════════════════════════════

SPECIFICATION MALFORMATION:
- Ambiguous: Multiple interpretations possible
- Incomplete: Missing edge cases
- Contradictory: Conflicts with itself or other specs
- Impossible: Describes behavior that can't exist
- Outdated: No longer matches domain requirements

CONTRACT MALFORMATION:
- Too weak: Allows violations of spec/domain
- Too strong: Rejects valid inputs
- Wrong focus: Checks irrelevant properties
- Tautological: Always true, meaningless
- Contradictory: Conflicts with spec

CODE MALFORMATION:
- Violates contracts: Doesn't do what contracts say
- Violates spec: Doesn't do what spec says
- Violates domain: Breaks business rules
- Semantic drift: Has evolved away from purpose

TEST MALFORMATION:
- Wrong expectation: Asserts incorrect result
- Wrong target: Tests something other than spec/contract
- Incomplete: Missing assertions
- Fragile: Depends on implementation details
- Codified bug: Passes because both test and code are wrong

OUTPUT FORMAT:
FEATURE: {class.feature}

ALIGNMENT TRACE:
  Domain truth: {what it should do}
  Spec says: {what spec says, or ABSENT}
  Contract says: {what contracts say, or ABSENT}
  Code does: {what code actually does}
  Test expects: {what test asserts, or ABSENT}

ALIGNMENT STATUS:
  Domain → Spec: [ALIGNED | MISALIGNED | SPEC_ABSENT]
  Spec → Contract: [ALIGNED | MISALIGNED | CONTRACT_ABSENT]
  Contract → Code: [ALIGNED | MISALIGNED]
  Code → Test: [ALIGNED | MISALIGNED | TEST_ABSENT]

GAPS FOUND:
  Gap 1: Between {layer} and {layer}
    {layer_A} says: {X}
    {layer_B} says: {Y}
    Bug implication: {what could go wrong}

MALFORMATIONS FOUND:
  {layer}: {malformation type} - {description}

PRIORITY: [CRITICAL | HIGH | MEDIUM | LOW]
```

## Success Criteria
- Full alignment chain traced
- Every gap between layers identified
- Malformations categorized by type
- Bug implications articulated

## Next Step
→ H03-PROBE-EDGE-CASES.md (PHASE CHANGE: Probe)
