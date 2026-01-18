# D03: Inheritance Audit

## Context
Phase: **DETECTION** (3 of 4)
Purpose: Validate inheritance decisions (is-a vs has-a)

## Mindset

**Inheritance is the strongest coupling. Use it only when correct.**

Inheritance should only be used for true IS-A relationships that satisfy:
1. **Liskov Substitution**: Child can always be used where parent expected
2. **Semantic correctness**: Child truly IS a kind of parent
3. **Interface inheritance**: Child wants parent's interface
4. **Implementation reuse**: Child uses parent's implementation

When any of these fail, composition (HAS-A) is better.

## Input Required
- Structure analysis from D01
- All source files with inheritance

## Prompt

```
TASK: Audit all inheritance relationships for correctness.

INPUT:
{structure_analysis_from_D01}
{source_files}

═══════════════════════════════════════════════════════════════
LISKOV SUBSTITUTION PRINCIPLE (LSP)
Can child always replace parent?
═══════════════════════════════════════════════════════════════

For each inheritance relationship:

INHERITANCE: {child} inherit {parent}

LSP CHECK:
  - Can {child} be used everywhere {parent} is expected? [YES | NO | PARTIAL]
  - Does {child} strengthen preconditions? [YES = VIOLATION]
  - Does {child} weaken postconditions? [YES = VIOLATION]
  - Does {child} throw unexpected exceptions? [YES = VIOLATION]

LSP VIOLATION: {child} cannot replace {parent}
  Reason: {why substitution fails}
  Example: {scenario where it breaks}
  RECOMMENDATION: Use composition instead

═══════════════════════════════════════════════════════════════
SEMANTIC IS-A CHECK
Is this truly an is-a relationship?
═══════════════════════════════════════════════════════════════

For each inheritance:

SEMANTIC CHECK: {child} inherit {parent}
  Question: Is a {child} truly a kind of {parent}?
  Answer: [YES | NO | QUESTIONABLE]

WRONG IS-A:
  STACK inherit ARRAY
    - A stack is NOT an array
    - It's a restriction of array behavior
    - Should be: STACK has-a ARRAY

CORRECT IS-A:
  SAVINGS_ACCOUNT inherit ACCOUNT
    - A savings account IS an account
    - Semantic relationship is correct

═══════════════════════════════════════════════════════════════
REFUSED BEQUEST
Does child actually use parent's features?
═══════════════════════════════════════════════════════════════

For each inheritance:

FEATURE USAGE: {child} inherit {parent}
  Parent features: {list}
  Used by child: {list}
  Redefined: {list}
  Ignored: {list}
  Undefine'd: {list}

REFUSED BEQUEST: {child}
  Ignores: {unused_parent_features}
  Undefines: {undefined_features}
  Percentage inherited but unused: {percent}%
  VERDICT: Child doesn't want parent's interface
  RECOMMENDATION: {alternative design}

═══════════════════════════════════════════════════════════════
INHERITANCE FOR IMPLEMENTATION ONLY
Using inheritance just to reuse code?
═══════════════════════════════════════════════════════════════

IMPLEMENTATION INHERITANCE: {child} inherit {parent}
  Public parent features used: {list}
  Private parent features used: {list}
  Interface wanted: [YES | NO]

WRONG: Inheriting just for implementation
  - Child doesn't want to be seen as parent
  - Child just wants parent's code
  - Should use: Composition or delegation

═══════════════════════════════════════════════════════════════
MULTIPLE INHERITANCE AUDIT
Is multiple inheritance appropriate?
═══════════════════════════════════════════════════════════════

For classes with multiple parents:

MULTIPLE INHERITANCE: {class}
  Parents: {list}
  Reason for each:
    - {parent1}: {why}
    - {parent2}: {why}

PROBLEMS:
  - Diamond problem: [YES | NO]
  - Feature conflicts: {list}
  - Semantic confusion: {does class have clear identity}

EVALUATION:
  Appropriate multiple inheritance: [YES | NO]
  Recommendation: {if NO, what instead}

═══════════════════════════════════════════════════════════════
INHERITANCE DEPTH AUDIT
Is the hierarchy too deep?
═══════════════════════════════════════════════════════════════

DEEP HIERARCHIES:
  Maximum depth found: {number}
  Deepest chain: {A → B → C → D → ...}

PROBLEMS with deep hierarchies:
  - Hard to understand behavior
  - Fragile base class problem
  - Difficulty in testing

RECOMMENDATION:
  - Depth > 4: Consider flattening
  - Depth > 6: Definitely refactor

═══════════════════════════════════════════════════════════════
COMPOSITION OPPORTUNITY IDENTIFICATION
Where should has-a replace is-a?
═══════════════════════════════════════════════════════════════

For each problematic inheritance:

REPLACE: {child} inherit {parent}
WITH: {child} has {parent_or_new_class}

REFACTORING:
  BEFORE:
```eiffel
class CAR
inherit
  VEHICLE
  RADIO
  GPS
```

  AFTER:
```eiffel
class CAR
inherit
  VEHICLE
feature
  radio: RADIO
  gps: GPS
```

REASON: {why composition is better}
EFFORT: [LOW | MEDIUM | HIGH]

═══════════════════════════════════════════════════════════════
DEFERRED CLASS USAGE
Are abstract classes used correctly?
═══════════════════════════════════════════════════════════════

DEFERRED CLASSES:
  {class}: {deferred features}
  Implementations: {concrete children}

AUDIT:
  - Is abstraction at right level? [YES | NO]
  - Could be interface instead? [YES | NO]
  - Missing implementations? [YES | NO]

═══════════════════════════════════════════════════════════════
INHERITANCE CORRECTNESS SUMMARY
═══════════════════════════════════════════════════════════════

OUTPUT FORMAT:

# INHERITANCE AUDIT: {library_name}

## Summary
- Total inheritance relationships: {count}
- Correct: {count}
- Questionable: {count}
- Incorrect: {count}

## LSP Violations
| Child | Parent | Problem | Severity |
|-------|--------|---------|----------|
| {child} | {parent} | {problem} | HIGH |

## Refused Bequest
| Child | Parent | Unused Features | Recommendation |
|-------|--------|-----------------|----------------|
| {child} | {parent} | {features} | {action} |

## Implementation-Only Inheritance
| Child | Parent | Recommendation |
|-------|--------|----------------|
| {child} | {parent} | Composition |

## Multiple Inheritance Issues
| Class | Parents | Issue | Fix |
|-------|---------|-------|-----|
| {class} | {parents} | {issue} | {fix} |

## Deep Hierarchy Chains
| Chain | Depth | Problem | Fix |
|-------|-------|---------|-----|
| A→B→C→D→E | 5 | Too deep | Flatten |

## Recommended Changes
1. {highest priority change}
2. {second priority}
...

## Correct Inheritance (No Changes Needed)
{List of properly designed inheritance}
```

## Success Criteria
- All inheritance relationships audited
- LSP violations identified
- Refused bequest cases found
- Composition opportunities identified
- Prioritized recommendations

## Next Step
→ D04-GENERICITY-SCAN.md
