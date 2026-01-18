# D05: Refactor Plan

## Context
Phase: **REFACTORING** (1 of 4)
Purpose: Design a comprehensive improvement plan

## Mindset

**Plan before refactoring. Refactoring without a plan is dangerous.**

Consolidate all findings from D01-D04 into a prioritized, executable refactoring plan that:
- Addresses the most impactful issues first
- Maintains working code throughout
- Can be done incrementally
- Has clear validation criteria

## Input Required
- Structure analysis (D01)
- Smell detection (D02)
- Inheritance audit (D03)
- Genericity scan (D04)

## Prompt

```
TASK: Create a comprehensive refactoring plan from all detection findings.

INPUT:
{structure_analysis_D01}
{smell_detection_D02}
{inheritance_audit_D03}
{genericity_scan_D04}

═══════════════════════════════════════════════════════════════
FINDING CONSOLIDATION
Gather all issues in one place
═══════════════════════════════════════════════════════════════

From D01 Structure Analysis:
  - {issue}: {description}

From D02 Smell Detection:
  - {smell}: {location} - {severity}

From D03 Inheritance Audit:
  - {issue}: {classes involved}

From D04 Genericity Scan:
  - {opportunity}: {classes affected}

TOTAL ISSUES: {count}

═══════════════════════════════════════════════════════════════
ISSUE PRIORITIZATION
Which issues to fix first?
═══════════════════════════════════════════════════════════════

PRIORITY FACTORS:
  - Impact: How much code affected?
  - Risk: How likely to cause bugs?
  - Effort: How hard to fix?
  - Dependency: Does other work depend on this?

For each issue:

ISSUE: {id}
  Description: {what}
  Impact: [HIGH | MEDIUM | LOW]
  Risk: [HIGH | MEDIUM | LOW]
  Effort: [XS | S | M | L | XL]
  Blocked by: {other issues}
  Blocks: {other issues}
  PRIORITY: {calculated priority}

═══════════════════════════════════════════════════════════════
REFACTORING DEPENDENCY GRAPH
What order must changes happen?
═══════════════════════════════════════════════════════════════

Some refactorings depend on others:

DEPENDENCIES:
  {refactoring_A} → {refactoring_B}
    Reason: {why A must happen first}

  {refactoring_C} → {refactoring_D}
    Reason: {why C must happen first}

DEPENDENCY GRAPH:
```
[EXTRACT_BASE] ──┬──> [APPLY_GENERIC]
                 │
[FIX_INHERIT]  ──┴──> [SIMPLIFY_CLASS]
                           │
                           └──> [CLEANUP]
```

═══════════════════════════════════════════════════════════════
REFACTORING BATCHES
Group related changes
═══════════════════════════════════════════════════════════════

BATCH 1: {theme}
  Refactorings:
    - {refactoring_1}
    - {refactoring_2}
  Total effort: {sum}
  Dependencies: None (can start)

BATCH 2: {theme}
  Refactorings:
    - {refactoring_3}
    - {refactoring_4}
  Total effort: {sum}
  Dependencies: Batch 1

═══════════════════════════════════════════════════════════════
REFACTORING SPECIFICATIONS
Detailed plan for each refactoring
═══════════════════════════════════════════════════════════════

For each refactoring:

REFACTORING: {id} - {name}
  Type: [EXTRACT_CLASS | INTRODUCE_GENERIC | REPLACE_INHERITANCE |
         MOVE_FEATURE | RENAME | INLINE | EXTRACT_METHOD]

  Problem: {what design issue this fixes}
  Solution: {what the refactoring does}

  Before:
```eiffel
{current code structure}
```

  After:
```eiffel
{target code structure}
```

  Steps:
    1. {specific step}
    2. {specific step}
    ...

  Validation:
    - {how to verify refactoring correct}
    - {tests that must pass}

  Risk:
    - {what could go wrong}
    - {mitigation}

═══════════════════════════════════════════════════════════════
NEW CLASS DESIGNS
Design classes to be created
═══════════════════════════════════════════════════════════════

For each new class:

NEW CLASS: {NAME}
  Purpose: {why this class}
  Extracted from: {source class(es)}

  Class outline:
```eiffel
class {NAME} [G -> CONSTRAINT]

inherit
  {PARENT}

create
  make

feature -- Access

  {queries}

feature -- Operations

  {commands}

invariant
  {invariants}

end
```

  Migration:
    - {class1} will use {NAME} for {what}
    - {class2} will use {NAME} for {what}

═══════════════════════════════════════════════════════════════
CLASS CHANGES
Design changes to existing classes
═══════════════════════════════════════════════════════════════

For each class modification:

CLASS CHANGE: {CLASS_NAME}

  Inheritance changes:
    Remove: {parents to remove}
    Add: {parents to add}

  Features to remove:
    - {feature}: moved to {new_location}

  Features to add:
    - {feature}: from {source}

  Attributes to change:
    - {attribute}: {old_type} → {new_type}

═══════════════════════════════════════════════════════════════
MIGRATION STRATEGY
How to transition safely
═══════════════════════════════════════════════════════════════

STRATEGY: [BIG_BANG | INCREMENTAL | STRANGLER]

If INCREMENTAL:
  Phase 1: {what changes}
    - Code compiles: [YES]
    - Tests pass: [YES]
    - Old interface: [STILL WORKS]

  Phase 2: {what changes}
    ...

  Phase N: Remove deprecated code

DEPRECATION:
  Features to deprecate: {list}
  Deprecation period: {how long}
  Migration notes: {for clients}

═══════════════════════════════════════════════════════════════
TEST IMPACT
What tests need updating?
═══════════════════════════════════════════════════════════════

EXISTING TESTS:
  - {test}: [STILL_VALID | NEEDS_UPDATE | OBSOLETE]

NEW TESTS NEEDED:
  - test_{what}: verify {refactoring}

TEST UPDATE EFFORT: {estimate}

═══════════════════════════════════════════════════════════════
REFACTORING SCHEDULE
Execution order
═══════════════════════════════════════════════════════════════

EXECUTION ORDER:

1. {refactoring_id}: {description}
   Effort: {XS-XL}
   Validates: {how to verify}

2. {refactoring_id}: {description}
   Effort: {XS-XL}
   Depends on: #1
   Validates: {how to verify}

...

TOTAL ESTIMATED EFFORT: {sum}

OUTPUT FORMAT:

# REFACTORING PLAN: {library_name}

## Executive Summary
- Issues identified: {count}
- Refactorings planned: {count}
- New classes to create: {count}
- Classes to modify: {count}
- Estimated improvement: {description}

## Priority Matrix
| ID | Issue | Impact | Effort | Priority |
|----|-------|--------|--------|----------|
| R01 | {desc} | HIGH | M | 1 |

## Dependency Graph
{Visual showing execution order}

## Refactoring Batches

### Batch 1: {theme}
{List of refactorings}

### Batch 2: {theme}
{List of refactorings}

## Detailed Specifications

### R01: {Name}
{Full specification}

### R02: {Name}
{Full specification}

## New Classes
{Design for each new class}

## Migration Strategy
{How to transition}

## Test Impact
{What tests change}

## Execution Schedule
{Ordered list of work}
```

## Success Criteria
- All issues prioritized
- Dependencies mapped
- Detailed specifications written
- New classes designed
- Migration strategy defined
- Test impact assessed

## Next Step
→ D06-EXTRACT-ABSTRACTIONS.md
