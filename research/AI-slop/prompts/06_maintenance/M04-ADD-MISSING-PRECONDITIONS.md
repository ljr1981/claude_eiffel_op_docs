# M04: Audit Tests

## Context
Phase: **AUDIT** (1 of 3)
Scenario: Pre-existing codebase, improvement goal

## Input Required
- Eiffel test files
- Source files
- Specification audit from M01
- Contract audit from M02
- Problem domain description

## Prompt

```
TASK: Audit test coverage and correctness.

INPUT:
{test_files}
{source_files}
{specification_audit}
{contract_audit}
{problem_domain}

═══════════════════════════════════════════════════════════════
TEST COVERAGE AUDIT
What is tested?
═══════════════════════════════════════════════════════════════

FOR EACH PUBLIC FEATURE:
[ ] Has at least one test?
[ ] Normal case tested?
[ ] Edge cases tested?
[ ] Error cases tested?
[ ] Precondition boundaries tested?

ABSENT TEST PATTERNS:
- Feature with no test
- Edge case with no test
- Error path with no test
- State transition with no test
- Integration scenario with no test

═══════════════════════════════════════════════════════════════
TEST CORRECTNESS AUDIT
Do tests expect the right behavior?
═══════════════════════════════════════════════════════════════

FOR EACH TEST:
[ ] Test matches spec? (expects what spec says)
[ ] Test matches contract? (verifies postconditions)
[ ] Test matches domain? (expects domain-correct behavior)

MALFORMED TEST PATTERNS:
- Test expects wrong result (codifies bug)
- Test tests irrelevant property
- Test doesn't exercise claimed behavior
- Test passes for wrong reasons (weak assertion)
- Test contradicts specification
- Test contradicts contract
- Test asserts implementation detail, not behavior
- Test would pass even if feature is broken

═══════════════════════════════════════════════════════════════
TEST-SPEC ALIGNMENT
═══════════════════════════════════════════════════════════════

For each spec'd behavior:
[ ] Is there a test for it?
[ ] Does test expect what spec says?

For each test:
[ ] Does it correspond to a spec'd behavior?
[ ] Or is it testing undocumented behavior?

═══════════════════════════════════════════════════════════════
TEST-CONTRACT ALIGNMENT
═══════════════════════════════════════════════════════════════

For each contract:
[ ] Is there a test that would catch violation?

For each test assertion:
[ ] Does it verify a contract property?
[ ] Or is it testing something contracts don't cover?

OUTPUT FORMAT:
COVERAGE SUMMARY:
  Features with tests: {X}%
  Features without tests: {list}

TEST: {test_name}
  Tests feature: {class.feature}

  Correctness: [CORRECT | MALFORMED]
  If MALFORMED:
    Test expects: {what test asserts}
    Spec says: {what spec says, or N/A}
    Contract says: {what contract says, or N/A}
    Domain requires: {what domain requires}
    Problem: {the mismatch}

  Alignment:
    With spec: [ALIGNED | MISALIGNED | NO_SPEC]
    With contract: [ALIGNED | MISALIGNED | NO_CONTRACT]
    With domain: [ALIGNED | MISALIGNED]

TEST QUALITY SCORE: {X}/10

PRIORITY FIXES:
1. {most critical test issue}
2. {next priority}

ABSENT TESTS NEEDED:
1. {feature} - needs test for {scenario}
2. {feature} - needs test for {scenario}
```

## Success Criteria
- Coverage gaps identified
- Test correctness assessed
- Alignment with spec/contract/domain checked
- Malformed tests identified

## Next Step
→ M05-FIX-SPECIFICATIONS.md (PHASE CHANGE: Strengthen)
