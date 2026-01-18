# D07: Apply Genericity

## Context
Phase: **REFACTORING** (3 of 4)
Purpose: Introduce generic classes where identified

## Mindset

**Replace duplication with parameterization.**

Genericity is Eiffel's most powerful reuse mechanism. This phase:
- Creates generic versions of duplicate classes
- Adds type parameters where appropriate
- Applies constrained genericity for type safety
- Removes redundant type-specific classes

## Input Required
- Refactoring plan from D05 (genericity sections)
- Genericity scan from D04
- Source files

## Prompt

```
TASK: Implement genericity improvements from the refactoring plan.

INPUT:
{refactoring_plan_D05}
{genericity_scan_D04}
{source_files}

RULE: Generify incrementally. Verify type safety at each step.

═══════════════════════════════════════════════════════════════
INTRODUCE TYPE PARAMETER
Add generic parameter to existing class
═══════════════════════════════════════════════════════════════

For each class to parameterize:

GENERIFY: {id}
  Class: {CLASS_NAME}
  Current: uses {SPECIFIC_TYPE} directly
  After: {CLASS_NAME} [G]

STEP 1: Add type parameter

BEFORE:
```eiffel
class SPECIFIC_PROCESSOR
feature
  item: STRING
  process (a_item: STRING)
    do
      -- work with STRING
    end
```

AFTER:
```eiffel
class PROCESSOR [G]
feature
  item: G
  process (a_item: G)
    do
      -- work with G
    end
```

STEP 2: Update all usages

BEFORE:
```eiffel
processor: SPECIFIC_PROCESSOR
```

AFTER:
```eiffel
processor: PROCESSOR [STRING]
```

STEP 3: Compile and verify type safety
  - Compiles: [YES | NO]
  - Type errors found: {list}

═══════════════════════════════════════════════════════════════
MERGE DUPLICATE CLASSES INTO GENERIC
Combine type-specific classes
═══════════════════════════════════════════════════════════════

For each set of duplicate classes:

MERGE: {id}
  Classes to merge: {CLASS_A}, {CLASS_B}, {CLASS_C}
  Generic result: {GENERIC_CLASS} [G]

STEP 1: Create generic class with union of features

```eiffel
class GENERIC_CLASS [G]
  -- Replaces {CLASS_A}, {CLASS_B}, {CLASS_C}

feature -- From all merged classes

  {common_features}

feature -- Specific to type (if needed)

  -- May need to use constrained genericity
  -- or move type-specific to descendants

end
```

STEP 2: Update all usages of old classes

BEFORE:
```eiffel
my_strings: STRING_LIST
my_integers: INTEGER_LIST
```

AFTER:
```eiffel
my_strings: GENERIC_LIST [STRING]
my_integers: GENERIC_LIST [INTEGER]
```

STEP 3: Delete old type-specific classes
  Files deleted: {list}

STEP 4: Verify no broken references
  Compile: [YES | NO]
  Missing references: {list}

═══════════════════════════════════════════════════════════════
ADD CONSTRAINED GENERICITY
Add constraints where needed
═══════════════════════════════════════════════════════════════

For generic classes that need type constraints:

CONSTRAIN: {id}
  Class: {CLASS_NAME} [G]
  Needs: G must support {operations}
  Constraint: G -> {CONSTRAINT_CLASS}

BEFORE:
```eiffel
class SORTABLE_LIST [G]
feature
  sort
    local
      l_temp: G
    do
      -- Needs to compare G items
      if item_a < item_b then  -- ERROR: G has no '<'
```

AFTER:
```eiffel
class SORTABLE_LIST [G -> COMPARABLE]
feature
  sort
    local
      l_temp: G
    do
      -- Now valid: G has '<' from COMPARABLE
      if item_a < item_b then
```

COMMON CONSTRAINTS:
  - G -> COMPARABLE: for sorting, min/max
  - G -> HASHABLE: for hash tables
  - G -> NUMERIC: for arithmetic
  - G -> ANY: no constraint (default)

═══════════════════════════════════════════════════════════════
GENERIFY FEATURES
Make individual features generic
═══════════════════════════════════════════════════════════════

For features that should be generic:

GENERIFY FEATURE: {id}
  Class: {CLASS_NAME}
  Feature: {feature_name}
  Current: works with {SPECIFIC_TYPE}
  After: works with any G

BEFORE:
```eiffel
process_string (a_item: STRING): STRING
  do
    -- specific to STRING
  end

process_integer (a_item: INTEGER): INTEGER
  do
    -- same algorithm, different type
  end
```

AFTER:
```eiffel
process [G] (a_item: G): G
  do
    -- works with any type
  end
```

NOTE: Eiffel feature-level genericity is limited.
Consider making the class generic instead.

═══════════════════════════════════════════════════════════════
REPLACE ANY WITH PROPER GENERICS
Fix unsafe ANY usage
═══════════════════════════════════════════════════════════════

For each ANY-typed element:

FIX ANY: {id}
  Location: {class}.{feature/attribute}
  Current: ANY (or ARRAYED_LIST [ANY])
  Should be: {PROPER_TYPE} or [G]

BEFORE:
```eiffel
items: ARRAYED_LIST [ANY]

add_item (a_item: ANY)
  do
    items.extend (a_item)
  end

get_item (i: INTEGER): ANY
  do
    Result := items [i]
  end
```

AFTER:
```eiffel
class CONTAINER [G]

items: ARRAYED_LIST [G]

add_item (a_item: G)
  do
    items.extend (a_item)
  end

get_item (i: INTEGER): G
  do
    Result := items [i]
  end
```

TYPE SAFETY GAINED:
  - Compile-time type checking
  - No runtime casts needed
  - Clear API contract

═══════════════════════════════════════════════════════════════
REMOVE TYPE CASTS
Eliminate unnecessary casting
═══════════════════════════════════════════════════════════════

For each type cast in the code:

REMOVE CAST: {id}
  Location: {class}.{feature}:{line}
  Cast: attached {TYPE} {expr} as l_typed

ANALYZE:
  Why is cast needed?
  - [ ] Using ANY where should be generic → Fix with genericity
  - [ ] Checking subtype → May be necessary
  - [ ] Wrong type in container → Fix container type

BEFORE:
```eiffel
item := items.item (i)
if attached {STRING} item as l_string then
  process_string (l_string)
end
```

AFTER (with proper generics):
```eiffel
-- items is now ARRAYED_LIST [STRING]
process_string (items.item (i))  -- No cast needed
```

═══════════════════════════════════════════════════════════════
COVARIANCE CHECK
Verify generic covariance is correct
═══════════════════════════════════════════════════════════════

For generic hierarchies:

COVARIANCE CHECK: {id}
  Parent: PARENT_CLASS [G]
  Child: CHILD_CLASS [G] inherit PARENT_CLASS [G]

VERIFY:
  - Child doesn't restrict G inappropriately
  - Redefined features maintain type safety
  - No CAT calls (changing argument type)

═══════════════════════════════════════════════════════════════
GENERICITY VERIFICATION
Test the generic classes
═══════════════════════════════════════════════════════════════

For each generic class:

TEST GENERIC: {CLASS_NAME} [G]

  Instantiate with different types:
```eiffel
test_generic_with_string
  local
    l_instance: {CLASS_NAME} [STRING]
  do
    create l_instance.make
    -- test with STRING
  end

test_generic_with_integer
  local
    l_instance: {CLASS_NAME} [INTEGER]
  do
    create l_instance.make
    -- test with INTEGER
  end
```

  All types work: [YES | NO]
  Type-specific issues: {list}

OUTPUT FORMAT:

# GENERICITY REPORT: {library_name}

## Summary
- Generic classes created: {count}
- Classes merged into generics: {count}
- Type parameters added: {count}
- ANY usages removed: {count}
- Type casts removed: {count}

## New Generic Classes

### {GENERIC_CLASS} [G]
- Replaces: {list of old classes}
- Constraint: {if any}
- Features: {count}

### {GENERIC_CLASS_2} [G, H]
...

## Classes Deleted (Merged)
- {OLD_CLASS_1}: replaced by {GENERIC [TYPE]}
- {OLD_CLASS_2}: replaced by {GENERIC [TYPE]}

## Usages Updated
| Location | Before | After |
|----------|--------|-------|
| {class.attr} | {OLD_TYPE} | {GENERIC [T]} |

## Type Safety Improvements
- Type casts removed: {count}
- ANY usages fixed: {count}
- Compile-time checks added: {count}

## Tests
- Compile: PASS
- Type safety: VERIFIED
- Generic tests: {pass}/{total}

## Code Reduction
- Classes before: {n}
- Classes after: {n}
- Lines removed: {estimate}
```

## Implementation Notes

1. Generify one class at a time
2. Update all usages before deleting old classes
3. Add constraints as needed
4. Test with multiple type instantiations
5. Verify no regression in existing functionality

## Success Criteria
- All planned genericity applied
- Type safety improved (no casts)
- Duplicate classes eliminated
- All tests pass
- Code compiles without warnings

## Next Step
→ D08-VERIFY-DESIGN.md
