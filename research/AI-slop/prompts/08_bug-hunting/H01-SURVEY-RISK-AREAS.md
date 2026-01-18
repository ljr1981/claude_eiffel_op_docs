# H01: Survey Risk Areas

## Context
Phase: **SURVEY** (1 of 3)
Scenario: Bug hunting in pre-existing code, bugs unknown

## Input Required
- Eiffel source code to analyze
- Test files (if any)
- Specifications/documentation (if any)
- Problem domain description

## Prompt

```
TASK: Identify high-risk areas across ALL layers: spec, contract, code, test.

INPUT:
{eiffel_source_code}
{test_files}
{specifications}
{problem_domain}

═══════════════════════════════════════════════════════════════
LAYER 1: SPECIFICATION RISKS
Bugs hide where specifications are absent or malformed.
═══════════════════════════════════════════════════════════════

ABSENT SPECIFICATION:
[ ] Feature with no documented purpose (no note, no comment)
[ ] Class with no description of responsibility
[ ] Complex behavior with no written requirements
[ ] Edge cases with no defined expected behavior

MALFORMED SPECIFICATION:
[ ] Spec contradicts domain rules
[ ] Spec is ambiguous (multiple interpretations possible)
[ ] Spec is internally inconsistent
[ ] Spec conflicts with other specs
[ ] Spec describes impossible behavior

═══════════════════════════════════════════════════════════════
LAYER 2: CONTRACT RISKS
Bugs hide where contracts are absent or malformed.
═══════════════════════════════════════════════════════════════

ABSENT CONTRACT:
[ ] Features with NO require clause
[ ] Features with NO ensure clause
[ ] Classes with NO invariant clause
[ ] Public features without preconditions
[ ] Commands without postconditions

MALFORMED CONTRACT:
[ ] Precondition doesn't match feature purpose
[ ] Postcondition doesn't guarantee claimed behavior
[ ] Invariant contradicts class semantics
[ ] Contract is tautology (always true)
[ ] Contract is impossible (always false)
[ ] Contract references wrong attributes
[ ] Contract contradicts specification

═══════════════════════════════════════════════════════════════
LAYER 3: CODE RISKS
Bugs hide where code is malformed.
═══════════════════════════════════════════════════════════════

MALFORMED CODE:
[ ] Algorithm doesn't match domain requirements
[ ] Code contradicts its own contracts
[ ] Code contradicts specification
[ ] Naming suggests X, code does Y
[ ] Domain rules violated in implementation
[ ] Data transformations lose domain meaning
[ ] State transitions violate domain model

STRUCTURAL RISKS:
[ ] Long features (>50 lines)
[ ] Deep nesting (>3 levels)
[ ] Array indexing without bounds check
[ ] Division without zero check

═══════════════════════════════════════════════════════════════
LAYER 4: TEST RISKS
Bugs hide where tests are absent or malformed.
═══════════════════════════════════════════════════════════════

ABSENT TEST:
[ ] Feature with no test coverage
[ ] Edge case with no test
[ ] Error path with no test
[ ] State transition with no test
[ ] Integration scenario with no test

MALFORMED TEST:
[ ] Test expects wrong result (codifies bug)
[ ] Test tests irrelevant property
[ ] Test doesn't actually exercise claimed behavior
[ ] Test passes for wrong reasons
[ ] Test contradicts specification
[ ] Test asserts implementation detail, not behavior

═══════════════════════════════════════════════════════════════
CROSS-LAYER MISALIGNMENT (Most Dangerous)
═══════════════════════════════════════════════════════════════

[ ] Spec says X, contract says Y
[ ] Contract says Y, code does Z
[ ] Code does Z, test expects W
[ ] Multiple layers disagree on expected behavior

OUTPUT FORMAT:
RISK AREA 1:
  Location: {class.feature or file}
  Layer: [SPEC | CONTRACT | CODE | TEST | CROSS-LAYER]
  Type: [ABSENT | MALFORMED | MISALIGNED]
  Severity: [CRITICAL | HIGH | MEDIUM | LOW]
  Finding: {specific issue}
  Domain Impact: {what goes wrong in real-world terms}

SUMMARY BY LAYER:
- Specification: {N} absent, {N} malformed
- Contract: {N} absent, {N} malformed
- Code: {N} malformed
- Test: {N} absent, {N} malformed
- Cross-layer: {N} misalignments

HUNT ORDER: {prioritized list - cross-layer misalignments first}
```

## Success Criteria
- All four layers surveyed
- Absent AND malformed issues identified
- Cross-layer misalignments flagged as highest priority
- Domain impact articulated

## Next Step
→ H02-ANALYZE-SEMANTICS.md
