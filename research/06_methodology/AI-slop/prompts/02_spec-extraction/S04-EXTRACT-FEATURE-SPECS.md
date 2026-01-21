# S04: Extract Feature Specifications

## Context
Phase: **EXTRACTION** (2 of 4)
Purpose: Derive detailed behavioral specifications for key features

## Input Required
- Class specifications from S03
- Source code of key features
- Test cases that exercise features

## Prompt

```
TASK: Extract detailed behavioral specifications for public features.

INPUT:
{class_specifications_from_S03}
{feature_source_code}
{relevant_test_cases}

═══════════════════════════════════════════════════════════════
FEATURE SELECTION
Which features need detailed specs?
═══════════════════════════════════════════════════════════════

Priority features (require detailed specs):
1. Facade entry points (main API)
2. Engine core algorithms
3. Features with complex logic
4. Features with multiple code paths
5. Features called from tests

Skip (simple, self-documenting):
- Simple getters/setters
- Delegation-only features
- Standard inherited features

═══════════════════════════════════════════════════════════════
FEATURE BEHAVIOR ANALYSIS
What does this feature actually DO?
═══════════════════════════════════════════════════════════════

For each priority feature:

FEATURE: {class}.{feature_name}

SIGNATURE:
  {full_signature_with_types}

PURPOSE:
  {What problem does this solve?}

ALGORITHM (if applicable):
  1. {Step 1}
  2. {Step 2}
  ...

CODE PATHS:
  Path A: {condition} → {outcome}
  Path B: {condition} → {outcome}
  ...

═══════════════════════════════════════════════════════════════
CONTRACT DERIVATION
What contracts SHOULD exist?
═══════════════════════════════════════════════════════════════

EXISTING CONTRACTS:
  require
    {existing preconditions}
  ensure
    {existing postconditions}

DERIVED CONTRACTS (from implementation analysis):
  require
    {implicit assumptions in code}
  ensure
    {guaranteed outcomes from code}

RECOMMENDED ADDITIONS:
  - Add precondition: {what and why}
  - Add postcondition: {what and why}

═══════════════════════════════════════════════════════════════
STATE TRANSITION SPECIFICATION
How does this feature change state?
═══════════════════════════════════════════════════════════════

For commands (state-changing features):

STATE BEFORE:
  - {attribute1}: {expected state}
  - {attribute2}: {expected state}

STATE AFTER:
  - {attribute1}: {new state}
  - {attribute2}: {new state}

UNCHANGED:
  - {attributes that must not change}

═══════════════════════════════════════════════════════════════
INPUT/OUTPUT SPECIFICATION
What goes in, what comes out?
═══════════════════════════════════════════════════════════════

INPUTS:
  {param1}: {type}
    Valid: {what makes it valid}
    Invalid: {what would be invalid}

  {param2}: {type}
    Valid: {what makes it valid}
    Invalid: {what would be invalid}

OUTPUT:
  {return_type}
    Meaning: {what the return value represents}
    Range: {possible values}
    Relationship to input: {how output relates to input}

═══════════════════════════════════════════════════════════════
EDGE CASES (from code analysis)
What boundary conditions exist?
═══════════════════════════════════════════════════════════════

From implementation, identify:
- Empty collections
- Null/Void handling
- Boundary values (0, 1, max)
- Error conditions

EDGE CASE: {description}
  Input: {what triggers it}
  Behavior: {what happens}
  Contract: {how it's protected}

═══════════════════════════════════════════════════════════════
TEST CORRELATION
What tests verify this feature?
═══════════════════════════════════════════════════════════════

RELATED TESTS:
  - {test_name}: tests {what aspect}
  - {test_name}: tests {what aspect}

UNTESTED ASPECTS:
  - {code path or condition not covered by tests}

OUTPUT FORMAT:

# FEATURE SPECIFICATION: {class}.{feature_name}

## Signature
```eiffel
{full_signature}
```

## Purpose
{What this feature does and why}

## Behavior
{Algorithm or behavior description}

## Contracts
### Existing
{Current contracts}

### Recommended
{Contracts that should be added}

## State Changes
{For commands: before/after state}

## Input Validation
{Valid/invalid inputs}

## Output Specification
{What the output means}

## Edge Cases
{Boundary conditions}

## Test Coverage
{Related tests and gaps}
```

## Success Criteria
- All facade features have detailed specs
- Algorithm behavior documented
- Missing contracts identified
- Edge cases enumerated
- Test correlation established

## Output Location
Add to: `specs/CLASS-SPECS/{CLASS_NAME}.md` under each feature

## Next Step
→ S05-EXTRACT-CONSTRAINTS.md
