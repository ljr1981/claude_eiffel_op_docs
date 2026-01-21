# R08: Validate Design

## Context
Phase: **SYNTHESIS** (2 of 2)
Purpose: Verify the specification is complete, correct, and implementable

## Mindset

**A specification is only valuable if it can be implemented correctly.**

Validate that:
- All requirements are addressed
- Design principles are followed
- Contracts are implementable
- No contradictions exist
- Specification is complete

## Input Required
- Complete specification from R07
- Original requirements from R01/R03
- Design principles checklist

## Prompt

```
TASK: Validate the specification for completeness, correctness, and implementability.

INPUT:
{complete_specification_from_R07}
{requirements_from_R01_R03}

═══════════════════════════════════════════════════════════════
REQUIREMENT TRACEABILITY
Is every requirement addressed?
═══════════════════════════════════════════════════════════════

For each requirement:

REQUIREMENT: {FR/NFR-id}
  Statement: {requirement}
  Addressed by:
    Class: {class}
    Feature: {feature}
    Contract: {which contract}
  Verification: {how to verify}
  TRACED: [YES | NO]

TRACEABILITY MATRIX:
| Requirement | Class.Feature | Contract | Verified |
|-------------|---------------|----------|----------|
| FR-001 | SIMPLE_X.op | ensure x | ✓ |

UNTRACED REQUIREMENTS:
  {FR-id}: Not addressed by any class/feature
    Action: {how to address}

COVERAGE: {traced}/{total} = {percent}%

═══════════════════════════════════════════════════════════════
OOSC2 PRINCIPLE COMPLIANCE
Are design principles followed?
═══════════════════════════════════════════════════════════════

PRINCIPLE: Single Responsibility
  For each class:
    {CLASS}: {responsibility}
    Multiple responsibilities: [YES | NO]
  VERDICT: [COMPLIANT | VIOLATION: {class}]

PRINCIPLE: Open/Closed
  Extension points: {where can be extended}
  Modification required for extension: [YES | NO]
  VERDICT: [COMPLIANT | VIOLATION: {issue}]

PRINCIPLE: Liskov Substitution
  For each inheritance:
    {CHILD} can replace {PARENT}: [YES | NO]
  VERDICT: [COMPLIANT | VIOLATION: {class}]

PRINCIPLE: Interface Segregation
  Fat interfaces: {classes with too many features}
  VERDICT: [COMPLIANT | VIOLATION: {class}]

PRINCIPLE: Dependency Inversion
  High-level depends on low-level: [YES | NO]
  VERDICT: [COMPLIANT | VIOLATION: {issue}]

OOSC2 SCORE: {compliant}/{total} principles

═══════════════════════════════════════════════════════════════
EIFFEL EXCELLENCE CHECK
Are Eiffel best practices followed?
═══════════════════════════════════════════════════════════════

COMMAND-QUERY SEPARATION:
  Violations: {features that both query and command}
  VERDICT: [COMPLIANT | VIOLATION: {features}]

UNIFORM ACCESS:
  Could be changed from attribute to function: [YES]
  VERDICT: [COMPLIANT | VIOLATION]

DESIGN BY CONTRACT:
  Features with preconditions: {percent}%
  Features with postconditions: {percent}%
  Classes with invariants: {percent}%
  VERDICT: [EXCELLENT | GOOD | NEEDS_WORK]

INFORMATION HIDING:
  Internal features properly hidden: [YES | NO]
  VERDICT: [COMPLIANT | VIOLATION: {exposed}]

GENERICITY:
  Appropriate use of generics: [YES | NO]
  Missed opportunities: {list}
  VERDICT: [COMPLIANT | COULD_IMPROVE]

═══════════════════════════════════════════════════════════════
CONTRACT VALIDATION
Are contracts implementable and correct?
═══════════════════════════════════════════════════════════════

For each contract:

CONTRACT: {class}.{feature}.{require|ensure|invariant}
  Expression: {the contract}

  Checks:
    Syntactically valid: [YES | NO]
    Semantically meaningful: [YES | NO]
    Implementable: [YES | NO]
    Not too strong: [YES | NO]
    Not too weak: [YES | NO]

  VERDICT: [VALID | ISSUE: {problem}]

CONTRACT ISSUES:
  {contract}: {issue}
    Fix: {how to fix}

CONTRACT CONFLICTS:
  {contract1} conflicts with {contract2}
    Resolution: {how to resolve}

═══════════════════════════════════════════════════════════════
COMPLETENESS CHECK
Is the specification complete?
═══════════════════════════════════════════════════════════════

CLASSES:
  All domain concepts have classes: [YES | NO]
  Missing: {concepts without classes}

FEATURES:
  All operations specified: [YES | NO]
  Missing: {operations without features}

CONTRACTS:
  All features have contracts: [YES | NO]
  Missing: {features without contracts}

DOCUMENTATION:
  All classes documented: [YES | NO]
  All features documented: [YES | NO]
  Missing: {undocumented items}

COMPLETENESS SCORE: {percent}%

═══════════════════════════════════════════════════════════════
CONSISTENCY CHECK
Are there contradictions?
═══════════════════════════════════════════════════════════════

Check for:
- Same concept with different definitions
- Conflicting contracts
- Impossible state combinations
- Circular dependencies

INCONSISTENCIES:
  {inconsistency}: {description}
    Resolution: {how to fix}

CONSISTENCY VERDICT: [CONSISTENT | INCONSISTENCIES_FOUND]

═══════════════════════════════════════════════════════════════
IMPLEMENTABILITY CHECK
Can this be implemented?
═══════════════════════════════════════════════════════════════

FEASIBILITY:

Technical feasibility:
  - All algorithms known: [YES | NO]
  - Dependencies available: [YES | NO]
  - Platform constraints met: [YES | NO]

Contract feasibility:
  - All preconditions checkable: [YES | NO]
  - All postconditions verifiable: [YES | NO]
  - All invariants maintainable: [YES | NO]

Resource feasibility:
  - Estimated complexity: {O(?) for key operations}
  - Memory requirements: {estimate}
  - External dependencies: {list}

IMPLEMENTABILITY ISSUES:
  {issue}: {description}
    Recommendation: {how to address}

IMPLEMENTABILITY VERDICT: [READY | NEEDS_WORK]

═══════════════════════════════════════════════════════════════
TEST SPECIFICATION
What tests are implied?
═══════════════════════════════════════════════════════════════

From contracts, derive test cases:

TEST: test_{feature}_{scenario}
  From contract: {class}.{feature}.{ensure x}
  Input: {test input}
  Expected: {expected outcome}
  Priority: [HIGH | MEDIUM | LOW]

IMPLIED TESTS:
| Feature | Scenario | Input | Expected |
|---------|----------|-------|----------|
| {feature} | normal | {input} | {output} |
| {feature} | boundary | {input} | {output} |
| {feature} | error | {input} | {error} |

TEST COVERAGE: {test_count} tests from {contract_count} contracts

═══════════════════════════════════════════════════════════════
QUALITY SCORE
Overall specification quality
═══════════════════════════════════════════════════════════════

QUALITY METRICS:

| Metric | Score | Target | Met |
|--------|-------|--------|-----|
| Requirement coverage | {%} | 100% | [Y/N] |
| Contract coverage | {%} | 95% | [Y/N] |
| OOSC2 compliance | {n}/5 | 5/5 | [Y/N] |
| Completeness | {%} | 95% | [Y/N] |
| Consistency | {%} | 100% | [Y/N] |

OVERALL GRADE: [A | B | C | D | F]

═══════════════════════════════════════════════════════════════
VALIDATION SUMMARY
═══════════════════════════════════════════════════════════════

OUTPUT FORMAT:

# SPECIFICATION VALIDATION: {project_name}

## Validation Summary

| Category | Score | Status |
|----------|-------|--------|
| Requirement Traceability | {%} | {PASS/FAIL} |
| OOSC2 Compliance | {n}/5 | {PASS/FAIL} |
| Eiffel Excellence | {score} | {PASS/FAIL} |
| Contract Validity | {%} | {PASS/FAIL} |
| Completeness | {%} | {PASS/FAIL} |
| Consistency | {%} | {PASS/FAIL} |
| Implementability | {score} | {PASS/FAIL} |

**Overall Grade: {grade}**

## Issues Found

### Critical (Must Fix)
{List of critical issues}

### Important (Should Fix)
{List of important issues}

### Minor (Nice to Fix)
{List of minor issues}

## Untraced Requirements
{Requirements not addressed}

## Contract Issues
{Contract problems}

## Inconsistencies
{Contradictions found}

## Implementability Concerns
{Feasibility issues}

## Implied Tests
{Test cases derived from contracts}

## Recommendations

### Before Implementation
{What must be fixed}

### During Implementation
{What to watch for}

### After Implementation
{Validation needed}

## Certification

This specification is **[APPROVED | CONDITIONALLY APPROVED | NOT APPROVED]** for implementation.

Conditions (if applicable):
{List of conditions}

Validated by: {AI model}
Date: {date}
```

## Success Criteria
- All requirements traced
- OOSC2 principles followed
- Contracts validated
- Specification complete
- No contradictions
- Implementable

## Workflow Complete

The Spec-from-Research workflow is now complete. The specification is ready for:
1. Implementation review
2. Team discussion
3. Implementation
4. Testing against contracts
