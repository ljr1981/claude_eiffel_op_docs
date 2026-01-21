# D04: Genericity Scan

## Context
Phase: **DETECTION** (4 of 4)
Purpose: Find opportunities for generic classes

## Mindset

**Same code + different types = opportunity for genericity.**

Eiffel's genericity allows writing code once that works with multiple types. Missing genericity leads to:
- Code duplication
- Type-unsafe workarounds
- Maintenance burden
- Missed reuse opportunities

## Input Required
- Structure analysis from D01
- All source files
- Similar class patterns to compare

## Prompt

```
TASK: Scan for genericity opportunities in the codebase.

INPUT:
{structure_analysis_from_D01}
{source_files}

═══════════════════════════════════════════════════════════════
DUPLICATE STRUCTURE DETECTION
Find classes with same structure, different types
═══════════════════════════════════════════════════════════════

Look for:
- CLASS_X and CLASS_Y with same feature signatures but different types
- STRING_LIST, INTEGER_LIST, PERSON_LIST patterns
- Similar containers for different element types

DUPLICATES FOUND:

DUPLICATE SET: {id}
  Classes: {class1}, {class2}, {class3}
  Structural similarity: {percent}%
  Type differences:
    - {class1} uses TYPE_A
    - {class2} uses TYPE_B
  GENERICITY OPPORTUNITY: [HIGH | MEDIUM | LOW]
  SUGGESTED GENERIC: {CLASS_NAME} [G]

═══════════════════════════════════════════════════════════════
PARALLEL HIERARCHY DETECTION
Find type-specific class hierarchies
═══════════════════════════════════════════════════════════════

Look for:
```
STRING_PROCESSOR ←→ INTEGER_PROCESSOR ←→ DATE_PROCESSOR
      ↓                   ↓                   ↓
STRING_RESULT    ←→ INTEGER_RESULT    ←→ DATE_RESULT
```

PARALLEL HIERARCHIES:

HIERARCHY SET: {id}
  Type A: {class1, class2, ...}
  Type B: {class3, class4, ...}
  Type C: {class5, class6, ...}
  Pattern: {what makes them parallel}
  COLLAPSE TO: {GENERIC_CLASS} [G]

═══════════════════════════════════════════════════════════════
TYPE-SPECIFIC FEATURE DETECTION
Find features duplicated for different types
═══════════════════════════════════════════════════════════════

Look for:
- process_string, process_integer, process_date
- convert_to_string, convert_to_integer
- Same algorithm, different types

TYPE-SPECIFIC FEATURES:

FEATURE SET: {id}
  Features: {feature1}, {feature2}, {feature3}
  Location: {class}
  Algorithm: {same}
  Type difference: {what varies}
  GENERIFY AS: {feature_name} [G] (a_param: G)

═══════════════════════════════════════════════════════════════
UNCONSTRAINED ANY DETECTION
Find features using ANY where specific type needed
═══════════════════════════════════════════════════════════════

Look for:
- Collections of ANY
- Parameters typed as ANY
- Return types of ANY

ANY USAGE:

UNCONSTRAINED ANY: {class}.{feature}
  Uses: ANY (or like ANY)
  Actual types stored/processed: {observed_types}
  Type safety lost: [YES | NO]
  GENERIFY AS: {G} or {G -> CONSTRAINT}

═══════════════════════════════════════════════════════════════
TYPE CASTING DETECTION
Find code that casts or checks types
═══════════════════════════════════════════════════════════════

Look for:
- attached {TYPE} as l_typed
- object_id comparison
- type checking patterns

TYPE CASTS:

CAST: {class}.{feature}:{line}
  Code: attached {SPECIFIC_TYPE} {object} as l_typed
  Reason: {why casting needed}
  INDICATES: Missing genericity or wrong design
  FIX: {how to make type-safe}

═══════════════════════════════════════════════════════════════
COLLECTION TYPE ANALYSIS
Are collections properly typed?
═══════════════════════════════════════════════════════════════

For each collection:

COLLECTION: {class}.{attribute}
  Declared type: ARRAYED_LIST [{G}]
  Element type: {what G is}
  Type-safe: [YES | NO]

UNTYPED COLLECTIONS:
  - {class}.{list}: ARRAYED_LIST [ANY] → should be {proper_type}
  - {class}.{table}: HASH_TABLE [ANY, ANY] → should be {K, V}

═══════════════════════════════════════════════════════════════
CONSTRAINED GENERICITY OPPORTUNITIES
Where should type parameters be constrained?
═══════════════════════════════════════════════════════════════

Look for:
- Generic classes that call specific features on G
- Need for {G -> COMPARABLE} or similar

CONSTRAINT NEEDS:

CLASS: {class_name} [G]
  Uses on G: {features called on G}
  Constraint needed: G -> {CONSTRAINT}
  Currently constrained: [YES | NO]

═══════════════════════════════════════════════════════════════
ALGORITHM GENERALIZATION
Find algorithms that could be generic
═══════════════════════════════════════════════════════════════

For algorithms that operate on specific types:

ALGORITHM: {class}.{feature}
  Current type: {specific_type}
  Could work with: {broader_types}
  Dependencies on type: {what type-specific operations}
  GENERIFY: {how to parameterize}

═══════════════════════════════════════════════════════════════
GENERICITY IMPACT ASSESSMENT
What's the benefit of adding genericity?
═══════════════════════════════════════════════════════════════

For each opportunity:

OPPORTUNITY: {id}
  Current code: {duplicated_classes/features}
  After genericity: {single_generic}
  Lines saved: {estimate}
  Type safety improved: [YES | NO]
  Reusability improved: [YES | NO]
  PRIORITY: [HIGH | MEDIUM | LOW]

═══════════════════════════════════════════════════════════════
GENERICITY SUMMARY
═══════════════════════════════════════════════════════════════

OUTPUT FORMAT:

# GENERICITY SCAN: {library_name}

## Summary
- Generic classes currently: {count}
- Genericity opportunities found: {count}
- Estimated code reduction: {percent}%

## Duplicate Structures (High Value)
| Classes | Similarity | Generic Solution |
|---------|------------|------------------|
| A, B, C | 95% | COMBINED [G] |

## Parallel Hierarchies
| Hierarchy | Classes | Solution |
|-----------|---------|----------|
| {type}_{X} | {list} | {GENERIC [G]} |

## Type-Specific Features
| Class | Features | Solution |
|-------|----------|----------|
| {class} | f_a, f_b, f_c | f [G] |

## Unsafe ANY Usage
| Location | Fix |
|----------|-----|
| {class}.{attr} | Use [G] |

## Type Casts to Remove
| Location | Cast | Fix |
|----------|------|-----|
| {location} | {cast} | {generic} |

## Recommended Generic Classes

### NEW: {SUGGESTED_CLASS} [G]
```eiffel
class SUGGESTED_CLASS [G]
  -- Replaces: {list of replaced classes}
feature
  {features}
end
```

Replaces:
- {class1}
- {class2}

Lines saved: {estimate}

## Implementation Priority
1. {Highest impact change}
2. {Second highest}
...
```

## Success Criteria
- All duplication patterns identified
- Genericity opportunities prioritized
- Unsafe type usage flagged
- Generic class designs sketched

## Next Step
→ D05-REFACTOR-PLAN.md
