# S03: Extract Class Specifications

## Context
Phase: **EXTRACTION** (1 of 4)
Purpose: Derive per-class responsibility specifications

## Input Required
- Domain model from S02
- All class source files
- Existing contracts (preconditions, postconditions, invariants)

## Prompt

```
TASK: Extract formal class specifications from implementation.

INPUT:
{domain_model_from_S02}
{class_source_files}

═══════════════════════════════════════════════════════════════
CLASS SPECIFICATION TEMPLATE
For each non-test class, produce a specification
═══════════════════════════════════════════════════════════════

CLASS SPECIFICATION: {CLASS_NAME}

IDENTITY:
  Name: {CLASS_NAME}
  Domain Concept: {from domain model}
  Role: [FACADE | ENGINE | DATA | HELPER | RENDERER]

PURPOSE:
  {Extract from note clause, or infer from class structure}

  This class represents: {what it models}
  This class is responsible for: {what it does}
  This class guarantees: {what contracts enforce}

═══════════════════════════════════════════════════════════════
CREATION SPECIFICATION
How can instances be created?
═══════════════════════════════════════════════════════════════

For each creation procedure:

CREATION: {procedure_name}
  Signature: {parameters with types}
  Purpose: {what this creation mode provides}
  Preconditions:
    - {from require clause, or "NONE" if missing}
  Postconditions:
    - {from ensure clause, or "NONE" if missing}
  Initial State:
    - {infer from implementation what attributes are set}

═══════════════════════════════════════════════════════════════
QUERY SPECIFICATION
What information can be retrieved?
═══════════════════════════════════════════════════════════════

For each query (function returning value, no side effects):

QUERY: {feature_name}
  Signature: {parameters} → {return_type}
  Purpose: {what question does this answer}
  Preconditions:
    - {from require clause}
  Postconditions:
    - {from ensure clause}
  Pure: [YES | NO] -- does it modify state?

═══════════════════════════════════════════════════════════════
COMMAND SPECIFICATION
What operations can be performed?
═══════════════════════════════════════════════════════════════

For each command (procedure modifying state):

COMMAND: {feature_name}
  Signature: {parameters}
  Purpose: {what state change does this cause}
  Preconditions:
    - {from require clause}
  Postconditions:
    - {from ensure clause}
  Modifies:
    - {which attributes change}

═══════════════════════════════════════════════════════════════
INVARIANT SPECIFICATION
What must always be true?
═══════════════════════════════════════════════════════════════

For each invariant clause:

INVARIANT: {name_if_tagged}
  Expression: {the Eiffel expression}
  Meaning: {human-readable explanation}
  Enforces: {what rule this protects}

Missing invariants (inferred from usage):
  - {attribute} should be: {inferred constraint}

═══════════════════════════════════════════════════════════════
DEPENDENCY SPECIFICATION
What does this class depend on?
═══════════════════════════════════════════════════════════════

DEPENDENCIES:
  Inherits: {parent classes}
  Uses: {classes referenced in features}
  Creates: {classes instantiated}

  Coupling assessment: [LOW | MEDIUM | HIGH]

═══════════════════════════════════════════════════════════════
CONTRACT COVERAGE ANALYSIS
How well-specified is this class?
═══════════════════════════════════════════════════════════════

COVERAGE:
  Features with preconditions: {X}/{Y} ({percent}%)
  Features with postconditions: {X}/{Y} ({percent}%)
  Has class invariant: [YES | NO]

  Overall specification quality: [STRONG | MODERATE | WEAK]

GAPS IDENTIFIED:
  - {Feature X}: missing postcondition for {what}
  - {Feature Y}: missing precondition for {what}

OUTPUT FORMAT:

# CLASS SPECIFICATION: {CLASS_NAME}

## Identity
- Role: {role}
- Domain Concept: {concept}

## Purpose
{Purpose statement}

## Creation
{Creation procedures with contracts}

## Queries
{Query features with contracts}

## Commands
{Command features with contracts}

## Invariants
{Class invariants}

## Dependencies
{What this class depends on}

## Contract Coverage
{Quality assessment and gaps}
```

## Success Criteria
- Every public feature documented
- All existing contracts captured
- Missing contracts identified as gaps
- Role and purpose clearly stated
- Dependencies mapped

## Output Location
Create: `specs/CLASS-SPECS/{CLASS_NAME}.md` for each class

## Next Step
→ S04-EXTRACT-FEATURE-SPECS.md
