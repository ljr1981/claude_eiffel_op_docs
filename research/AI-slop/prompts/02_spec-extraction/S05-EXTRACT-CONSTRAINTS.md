# S05: Extract Constraints

## Context
Phase: **EXTRACTION** (3 of 4)
Purpose: Derive system-wide rules and invariants

## Input Required
- All class specifications from S03
- All class invariants
- Preconditions that appear in multiple places

## Prompt

```
TASK: Extract system-wide constraints from class invariants and repeated preconditions.

INPUT:
{all_class_specifications}
{all_invariant_clauses}
{all_precondition_clauses}

═══════════════════════════════════════════════════════════════
INVARIANT COLLECTION
Gather all class invariants
═══════════════════════════════════════════════════════════════

For each class with an invariant clause:

CLASS: {class_name}
INVARIANTS:
  {tag}: {expression}
    Meaning: {human-readable}
    Protects: {what invalid state this prevents}

═══════════════════════════════════════════════════════════════
CONSTRAINT CATEGORIZATION
What types of constraints exist?
═══════════════════════════════════════════════════════════════

CATEGORY: Data Integrity
  - {constraint}: {where enforced}

CATEGORY: State Validity
  - {constraint}: {where enforced}

CATEGORY: Relationship Consistency
  - {constraint}: {where enforced}

CATEGORY: Business Rules
  - {constraint}: {where enforced}

═══════════════════════════════════════════════════════════════
CROSS-CLASS CONSTRAINTS
What rules span multiple classes?
═══════════════════════════════════════════════════════════════

Look for:
- Same precondition in multiple features
- Invariant in one class about another class's state
- Type constraints that imply relationships

CROSS-CLASS RULE: {description}
  Enforced in: {class1}.{feature}, {class2}.{feature}
  Meaning: {what system-wide property this maintains}

═══════════════════════════════════════════════════════════════
IMPLICIT CONSTRAINTS
What rules are assumed but not stated?
═══════════════════════════════════════════════════════════════

Analyze code for:
- Unchecked assumptions (missing preconditions)
- Implied relationships
- Convention-based constraints

IMPLICIT CONSTRAINT: {description}
  Evidence: {where in code this is assumed}
  Should be: {how it should be formalized}
  Risk: {what happens if violated}

═══════════════════════════════════════════════════════════════
VOID SAFETY CONSTRAINTS
What void-safety rules exist?
═══════════════════════════════════════════════════════════════

For detachable attributes:
- When can they be Void?
- When must they be attached?
- What operations require attachment checks?

VOID CONSTRAINT: {attribute}
  Detachable: [YES | NO]
  Null allowed: {when}
  Must be attached: {for what operations}

═══════════════════════════════════════════════════════════════
TEMPORAL CONSTRAINTS
What ordering requirements exist?
═══════════════════════════════════════════════════════════════

Look for:
- Setup-before-use patterns
- Initialization requirements
- Sequential dependencies

TEMPORAL RULE: {description}
  Before: {what must happen first}
  After: {what can happen after}
  Enforced by: {precondition or convention}

═══════════════════════════════════════════════════════════════
CONSTRAINT COMPLETENESS
Are all critical constraints specified?
═══════════════════════════════════════════════════════════════

MISSING CONSTRAINTS (identified from code patterns):
  - {Class}: should have invariant for {what}
  - {Feature}: should have precondition for {what}
  - {Relationship}: no constraint ensures {what}

OUTPUT FORMAT:

# SYSTEM CONSTRAINTS: {library_name}

## Data Integrity Rules
{List of data validity constraints}

## State Validity Rules
{List of valid state constraints}

## Relationship Rules
{List of cross-class consistency rules}

## Business Rules
{List of domain-specific rules}

## Void Safety Rules
{List of null/void constraints}

## Temporal Rules
{List of ordering/sequence requirements}

## Implicit Constraints (NEEDS FORMALIZATION)
{List of assumed-but-unstated rules}

## Constraint Gaps (NEEDS CONTRACTS)
{List of missing constraints that should exist}
```

## Success Criteria
- All invariants collected and categorized
- Cross-class rules identified
- Implicit constraints surfaced
- Temporal requirements documented
- Gaps in constraint coverage identified

## Output Location
Create: `specs/CONSTRAINTS.md`

## Next Step
→ S06-EXTRACT-BOUNDARIES.md
