# S08: Validate Specification

## Context
Phase: **SYNTHESIS** (2 of 2)
Purpose: Cross-check specification completeness and accuracy

## Input Required
- Synthesized specification (S07)
- Original source code
- Test suite

## Prompt

```
TASK: Validate the extracted specification against the original codebase.

INPUT:
{synthesized_specification}
{original_source_files}
{test_suite}

═══════════════════════════════════════════════════════════════
COMPLETENESS CHECK
Is every code artifact represented in the spec?
═══════════════════════════════════════════════════════════════

For each class in codebase:
  CLASS: {name}
  In spec: [YES | NO]
  All features documented: [YES | NO]
  Missing: {list of undocumented features}

COMPLETENESS SCORE: {X}% of features documented

═══════════════════════════════════════════════════════════════
ACCURACY CHECK
Does the spec accurately describe the code?
═══════════════════════════════════════════════════════════════

For each feature in spec:
  FEATURE: {class}.{feature}
  Spec says: {what spec claims}
  Code does: {what code actually does}
  Match: [EXACT | CLOSE | MISMATCH]

ACCURACY ISSUES:
  - {Feature}: spec says X, code does Y
  - {Feature}: spec incomplete, missing {aspect}

═══════════════════════════════════════════════════════════════
CONTRACT VERIFICATION
Are all contracts captured correctly?
═══════════════════════════════════════════════════════════════

For each contract in code:
  CONTRACT: {class}.{feature}.{require|ensure|invariant}
  In spec: [YES | NO]
  Accurate: [YES | NO]

MISSING CONTRACTS IN SPEC:
  - {contract not documented}

INCORRECT CONTRACTS IN SPEC:
  - {contract documented incorrectly}

═══════════════════════════════════════════════════════════════
TEST CORRELATION
Does the spec match test expectations?
═══════════════════════════════════════════════════════════════

For each test:
  TEST: {test_name}
  Tests: {what behavior}
  Spec describes this: [YES | NO]
  Consistent: [YES | NO]

TEST-SPEC MISMATCHES:
  - {test expects X, spec says Y}

═══════════════════════════════════════════════════════════════
CONSISTENCY CHECK
Is the spec internally consistent?
═══════════════════════════════════════════════════════════════

Check for:
- Same term used with different meanings
- Conflicting constraints
- Circular dependencies
- Impossible preconditions
- Unachievable postconditions

CONSISTENCY ISSUES:
  - {inconsistency found}

═══════════════════════════════════════════════════════════════
TRACEABILITY MATRIX
Can every spec item be traced to code?
═══════════════════════════════════════════════════════════════

| Spec Item | Source | Evidence |
|-----------|--------|----------|
| {requirement} | {class.feature} | {where in code} |

ORPHAN SPEC ITEMS (no code evidence):
  - {spec item without implementation}

ORPHAN CODE (no spec coverage):
  - {code without specification}

═══════════════════════════════════════════════════════════════
GAP ANALYSIS
What's missing from the spec?
═══════════════════════════════════════════════════════════════

DOCUMENTATION GAPS:
  Priority HIGH:
    - {critical undocumented behavior}
  Priority MEDIUM:
    - {important undocumented behavior}
  Priority LOW:
    - {minor undocumented behavior}

CONTRACT GAPS:
  - {Feature}: needs {type} contract for {reason}

TEST GAPS:
  - {Behavior}: untested, needs test for {reason}

═══════════════════════════════════════════════════════════════
AMBIGUITY REVIEW
What remains unclear?
═══════════════════════════════════════════════════════════════

AMBIGUITIES REQUIRING HUMAN REVIEW:
  1. {Question about intent}
     - Spec says: {X}
     - Could also mean: {Y}
     - Need clarification: {specific question}

  2. {Unclear behavior}
     - Code does: {X}
     - Intent unclear: {why this might be wrong}
     - Need confirmation: {specific question}

═══════════════════════════════════════════════════════════════
VALIDATION SUMMARY
Overall specification quality
═══════════════════════════════════════════════════════════════

METRICS:
  Completeness: {X}%
  Accuracy: {X}%
  Contract coverage: {X}%
  Test correlation: {X}%
  Consistency: {X}%

OVERALL GRADE: [A | B | C | D | F]

READY FOR USE: [YES | NO | WITH CAVEATS]

REQUIRED ACTIONS BEFORE USE:
  1. {Critical fix needed}
  2. {Important clarification needed}

OUTPUT FORMAT:

# SPECIFICATION VALIDATION REPORT: {library_name}

## Validation Summary
- Completeness: {X}%
- Accuracy: {X}%
- Overall Grade: {grade}

## Completeness Issues
{List of undocumented items}

## Accuracy Issues
{List of inaccurate descriptions}

## Consistency Issues
{List of internal inconsistencies}

## Traceability Gaps
{Orphan specs and orphan code}

## Ambiguities for Human Review
{Questions requiring clarification}

## Recommended Actions
### Critical (must fix)
{List}

### Important (should fix)
{List}

### Nice to have (optional)
{List}

## Certification
This specification is [VALIDATED | CONDITIONALLY VALIDATED | NOT VALIDATED]
for use in maintenance and development activities.

Validated by: {AI model}
Date: {date}
```

## Success Criteria
- Every spec item traced to code
- All contracts verified
- Inconsistencies identified
- Ambiguities flagged for human review
- Clear validation grade assigned

## Output Location
Create: `specs/VALIDATION-REPORT.md`

## Workflow Complete
The spec-extraction workflow is now complete. Output files:
- `specs/DOMAIN-MODEL.md`
- `specs/CLASS-SPECS/*.md`
- `specs/CONSTRAINTS.md`
- `specs/BOUNDARIES.md`
- `specs/SPEC-SUMMARY.md`
- `specs/VALIDATION-REPORT.md`
