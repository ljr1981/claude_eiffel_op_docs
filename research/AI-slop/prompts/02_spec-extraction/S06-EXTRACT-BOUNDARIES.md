# S06: Extract Boundaries

## Context
Phase: **EXTRACTION** (4 of 4)
Purpose: Derive edge cases and limits from tests and error handling

## Input Required
- All test classes and test cases
- Error handling code
- Boundary-related preconditions

## Prompt

```
TASK: Extract edge cases, limits, and error conditions from tests and code.

INPUT:
{test_source_files}
{error_handling_code}
{boundary_preconditions}

═══════════════════════════════════════════════════════════════
TEST CASE ANALYSIS
What do tests reveal about boundaries?
═══════════════════════════════════════════════════════════════

For each test:

TEST: {test_name}
  Tests: {what feature}
  Input: {test input}
  Expected: {expected outcome}
  Boundary aspect: {what edge case this covers}

BOUNDARY CATEGORIES FROM TESTS:
  - Empty inputs: {tests that use empty collections/strings}
  - Single elements: {tests with count = 1}
  - Large inputs: {stress tests if any}
  - Invalid inputs: {tests expecting errors}

═══════════════════════════════════════════════════════════════
EDGE CASE EXTRACTION
What boundary conditions exist?
═══════════════════════════════════════════════════════════════

For each identified edge case:

EDGE CASE: {name}
  Description: {what makes this an edge case}
  Valid input: [YES | NO]
  Expected behavior: {what should happen}
  Currently tested: [YES | NO]
  Contract protection: {precondition that guards this}

═══════════════════════════════════════════════════════════════
LIMIT ANALYSIS
What are the practical limits?
═══════════════════════════════════════════════════════════════

NUMERIC LIMITS:
  - {Parameter}: min={X}, max={Y}, typical={Z}
  - {Counter}: minimum={X}, maximum={Y}

COLLECTION LIMITS:
  - {Collection}: min_size={X}, max_practical={Y}
  - Empty allowed: [YES | NO]

STRING LIMITS:
  - {String param}: min_length={X}, max_length={Y}
  - Empty allowed: [YES | NO]
  - Newlines allowed: [YES | NO]

═══════════════════════════════════════════════════════════════
ERROR CONDITION INVENTORY
What can go wrong?
═══════════════════════════════════════════════════════════════

From code analysis, identify:
- File operations: file not found, permission denied
- Network operations: connection failed, timeout
- Resource operations: out of memory, limit exceeded
- Validation failures: invalid format, out of range

ERROR: {error_type}
  Trigger: {what causes this}
  Handling: {how code handles it}
  User notification: {how user learns of error}
  Recovery: {how to recover}

═══════════════════════════════════════════════════════════════
FAILURE MODE ANALYSIS
How does the system fail?
═══════════════════════════════════════════════════════════════

For each failure mode:

FAILURE: {description}
  Cause: {what triggers this failure}
  Symptoms: {observable effects}
  State after: {what state is the system in}
  Recovery: {how to return to valid state}
  Contract: {precondition that should prevent this}

═══════════════════════════════════════════════════════════════
PRECONDITION BOUNDARY ANALYSIS
What do preconditions reveal about limits?
═══════════════════════════════════════════════════════════════

For each precondition:

PRECONDITION: {expression}
  Feature: {where it appears}
  Boundary implied: {what limit this enforces}
  Just-valid input: {smallest/largest valid value}
  Just-invalid input: {smallest/largest invalid value}

═══════════════════════════════════════════════════════════════
UNTESTED BOUNDARIES
What edge cases lack test coverage?
═══════════════════════════════════════════════════════════════

Compare test coverage to identified boundaries:

MISSING TEST: {boundary_case}
  Should test: {what behavior}
  Why risky: {what could go wrong}
  Priority: [HIGH | MEDIUM | LOW]

═══════════════════════════════════════════════════════════════
DEFENSIVE PROGRAMMING ANALYSIS
How defensive is the code?
═══════════════════════════════════════════════════════════════

Check for:
- Input validation beyond preconditions
- Defensive copies
- Bounds checking
- Null guards (beyond void safety)

DEFENSIVE MEASURE: {description}
  Location: {where in code}
  Protects against: {what invalid state}
  Necessary: [YES | NO | UNCLEAR]

OUTPUT FORMAT:

# BOUNDARY SPECIFICATIONS: {library_name}

## Edge Cases

### Empty Input Cases
{List of empty input handling}

### Single Element Cases
{List of single-element handling}

### Boundary Value Cases
{List of min/max value handling}

## Limits

### Numeric Limits
{Table of numeric parameter limits}

### Collection Limits
{Table of collection size limits}

### String Limits
{Table of string length limits}

## Error Conditions

### Recoverable Errors
{List of errors with recovery}

### Fatal Errors
{List of unrecoverable errors}

## Failure Modes
{List of ways the system can fail}

## Test Coverage Gaps

### High Priority (need tests)
{Untested boundaries that are risky}

### Medium Priority (should test)
{Untested boundaries that could cause issues}

### Low Priority (nice to test)
{Untested boundaries unlikely to cause issues}
```

## Success Criteria
- All edge cases from tests documented
- Limits and boundaries identified
- Error conditions inventoried
- Failure modes analyzed
- Test coverage gaps identified

## Output Location
Create: `specs/BOUNDARIES.md`

## Next Step
→ S07-SYNTHESIZE-SPEC.md
