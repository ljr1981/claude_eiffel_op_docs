# D02: Smell Detection

## Context
Phase: **DETECTION** (2 of 4)
Purpose: Systematically identify design smells

## Mindset

**Design smells indicate deeper problems.**

Smells are symptoms, not diseases. Each smell suggests a structural problem that makes code harder to maintain, extend, or reuse.

## Input Required
- Structure analysis from D01
- All source files

## Prompt

```
TASK: Scan for design smells in the codebase.

INPUT:
{structure_analysis_from_D01}
{source_files}

═══════════════════════════════════════════════════════════════
SMELL: GOD CLASS
Class that does too much
═══════════════════════════════════════════════════════════════

Detection criteria:
- > 20 features
- > 300 lines of code
- Multiple distinct responsibilities
- Many dependencies on other classes

SCAN for God Classes:

GOD CLASS CANDIDATE: {class}
  Features: {count}
  Lines: {count}
  Responsibilities detected:
    1. {responsibility}: {features}
    2. {responsibility}: {features}
  Coupling: {count} other classes used
  SEVERITY: [HIGH | MEDIUM | LOW]
  RECOMMENDATION: Extract {suggested_class} for {responsibility}

═══════════════════════════════════════════════════════════════
SMELL: FEATURE ENVY
Feature uses another class's data more than its own
═══════════════════════════════════════════════════════════════

Detection criteria:
- Feature accesses > 3 features of another class
- Feature accesses more external data than own class data

SCAN for Feature Envy:

FEATURE ENVY: {class}.{feature}
  Uses from {other_class}: {list of features accessed}
  Uses from own class: {list of features accessed}
  External > Internal: [YES | NO]
  SEVERITY: [HIGH | MEDIUM | LOW]
  RECOMMENDATION: Move to {other_class} or extract intermediate class

═══════════════════════════════════════════════════════════════
SMELL: LONG PARAMETER LIST
Feature with too many parameters
═══════════════════════════════════════════════════════════════

Detection criteria:
- > 4 parameters
- Multiple parameters of same type
- Related parameters travel together

SCAN for Long Parameter Lists:

LONG PARAM LIST: {class}.{feature}
  Parameters: {count}
  Signature: {full signature}
  Parameter groups detected:
    - {param1, param2, param3}: appear related ({why})
  SEVERITY: [HIGH | MEDIUM | LOW]
  RECOMMENDATION: Introduce parameter object {suggested_name}

═══════════════════════════════════════════════════════════════
SMELL: DATA CLUMPS
Same data appears together in multiple places
═══════════════════════════════════════════════════════════════

Detection criteria:
- Same 3+ fields in multiple classes
- Same 3+ parameters in multiple features

SCAN for Data Clumps:

DATA CLUMP: {data1, data2, data3}
  Appears in:
    - {class1}.{feature/attribute}
    - {class2}.{feature/attribute}
  SEVERITY: [HIGH | MEDIUM | LOW]
  RECOMMENDATION: Extract class {suggested_name}

═══════════════════════════════════════════════════════════════
SMELL: PRIMITIVE OBSESSION
Using primitives where domain types should exist
═══════════════════════════════════════════════════════════════

Detection criteria:
- STRING used for emails, phones, URLs, IDs
- INTEGER used for money, quantities with units
- Type implies validation or formatting

SCAN for Primitive Obsession:

PRIMITIVE OBSESSION: {class}.{attribute/parameter}
  Type: STRING (or other primitive)
  Semantic: {what it represents}
  Should be: {domain type}
  Validation needed: {what rules apply}
  SEVERITY: [HIGH | MEDIUM | LOW]
  RECOMMENDATION: Create {domain_type} class

═══════════════════════════════════════════════════════════════
SMELL: DEAD CODE
Unreachable or unused code
═══════════════════════════════════════════════════════════════

Detection criteria:
- Features never called
- Branches never reached
- Classes never instantiated

SCAN for Dead Code:

DEAD CODE: {class}.{feature}
  Type: [NEVER_CALLED | NEVER_REACHED | OBSOLETE]
  Evidence: {how determined}
  SEVERITY: [HIGH | MEDIUM | LOW]
  RECOMMENDATION: Remove or verify intentional

═══════════════════════════════════════════════════════════════
SMELL: INAPPROPRIATE INTIMACY
Classes know too much about each other
═══════════════════════════════════════════════════════════════

Detection criteria:
- Accessing private/internal features
- Circular dependencies
- Changing behavior based on other class's internals

SCAN for Inappropriate Intimacy:

INAPPROPRIATE INTIMACY: {class1} ↔ {class2}
  {class1} accesses from {class2}: {features}
  {class2} accesses from {class1}: {features}
  Circular dependency: [YES | NO]
  SEVERITY: [HIGH | MEDIUM | LOW]
  RECOMMENDATION: {break the coupling}

═══════════════════════════════════════════════════════════════
SMELL: SPECULATIVE GENERALITY
Overly complex for current needs
═══════════════════════════════════════════════════════════════

Detection criteria:
- Abstract classes with only one concrete child
- Unused parameters "for future use"
- Overly generic when only one type used

SCAN for Speculative Generality:

SPECULATIVE GENERALITY: {class/feature}
  Pattern: {what was over-engineered}
  Usage: {how actually used}
  SEVERITY: [HIGH | MEDIUM | LOW]
  RECOMMENDATION: Simplify to actual needs

═══════════════════════════════════════════════════════════════
SMELL: COMMENTS AS DEODORANT
Comments explaining complex code instead of fixing it
═══════════════════════════════════════════════════════════════

Detection criteria:
- Long explanatory comments
- Comments that could be code (extract method)
- Apologetic comments ("this is a hack")

SCAN for Comment Smells:

COMMENT SMELL: {class}.{feature}:{line}
  Comment: "{text}"
  Problem: {what the comment is hiding}
  SEVERITY: [HIGH | MEDIUM | LOW]
  RECOMMENDATION: {refactoring to make comment unnecessary}

═══════════════════════════════════════════════════════════════
SMELL SUMMARY
Prioritized list of all smells
═══════════════════════════════════════════════════════════════

OUTPUT FORMAT:

# SMELL DETECTION REPORT: {library_name}

## Smell Summary
| Smell Type | Count | High | Medium | Low |
|------------|-------|------|--------|-----|
| God Class | {n} | {n} | {n} | {n} |
| Feature Envy | {n} | {n} | {n} | {n} |
| Long Param List | {n} | {n} | {n} | {n} |
| Data Clumps | {n} | {n} | {n} | {n} |
| Primitive Obsession | {n} | {n} | {n} | {n} |
| Dead Code | {n} | {n} | {n} | {n} |
| Inappropriate Intimacy | {n} | {n} | {n} | {n} |
| Speculative Generality | {n} | {n} | {n} | {n} |
| Comment Smells | {n} | {n} | {n} | {n} |

## High Severity Smells
{Detailed list with recommendations}

## Medium Severity Smells
{Detailed list}

## Low Severity Smells
{List}

## Top Refactoring Opportunities
1. {Most impactful improvement}
2. {Second most impactful}
...
```

## Success Criteria
- All smell categories scanned
- Each smell has severity and recommendation
- Prioritized refactoring opportunities identified

## Next Step
→ D03-INHERITANCE-AUDIT.md
