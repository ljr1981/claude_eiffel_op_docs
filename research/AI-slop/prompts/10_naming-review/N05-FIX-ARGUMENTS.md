# N05: Fix Arguments and Locals

## Context
Phase: **FIX** (3 of 4)
Purpose: Fix argument prefixes, local variables, and loop cursors

## Mindset

**Arguments and locals have limited scope but cause real bugs when named wrong.**

VOIT(2) errors from cursor conflicts. Shadowing bugs from locals matching attributes. The `a_` prefix prevents confusion between arguments and attributes.

## Input Required
- Classification report from N02
- List of argument/local/cursor fixes

## Prompt

```
TASK: Fix argument, local variable, and loop cursor naming.

INPUT:
{classification_from_N02}
Fixes needed:
{list of argument/local/cursor issues}

═══════════════════════════════════════════════════════════════
PART 1: FIX ARGUMENT NAMES
Convention: a_ prefix + lowercase
═══════════════════════════════════════════════════════════════

FOR EACH ARGUMENT WITHOUT a_ PREFIX:

ARGUMENT FIX: {Class}.{feature}({argument})

STEP 1: Identify the argument
  Feature: {feature_name}
  Argument: {current_name}
  Type: {type}
  Problem: {missing_a_prefix | CamelCase | shadows_attribute}

STEP 2: Determine correct name
  Current: {name}
  Correct: a_{name} (or a_{snake_case_name} if CamelCase)

STEP 3: Check for shadowing
  Does argument name match an attribute?
  Attribute: {attribute_name} (if any)

  If shadows:
  - MUST add a_ prefix
  - Prevents subtle bugs where arg assignment doesn't update attribute

STEP 4: Update signature and body
  File: {file}

  BEFORE:
  {feature_name} ({name}: {TYPE})
    do
      attribute := {name}
    end

  AFTER:
  {feature_name} (a_{name}: {TYPE})
    do
      attribute := a_{name}
    end

STEP 5: Update contracts using this argument
  BEFORE:
  require
      valid: {name}.is_valid

  AFTER:
  require
      valid: a_{name}.is_valid

═══════════════════════════════════════════════════════════════
PART 2: FIX LOCAL VARIABLE NAMES
Convention: lowercase, l_ prefix optional
═══════════════════════════════════════════════════════════════

FOR EACH LOCAL WITH NAMING ISSUE:

LOCAL FIX: {Class}.{feature} local {variable}

STEP 1: Identify the problem
  Local: {current_name}
  Problem: {CamelCase | shadows_feature | shadows_attribute}

STEP 2: Determine fix

  If CamelCase:
    myVariable → my_variable

  If shadows attribute/feature:
    Option A: Add l_ prefix
      count → l_count
    Option B: More descriptive name
      count → local_count, temp_count

STEP 3: Update declaration and all uses
  File: {file}

  BEFORE:
  local
      {oldName}: {TYPE}
  do
      {oldName} := ...
      if {oldName} > 0 then

  AFTER:
  local
      {new_name}: {TYPE}
  do
      {new_name} := ...
      if {new_name} > 0 then

═══════════════════════════════════════════════════════════════
PART 3: FIX LOOP CURSOR NAMES
Convention: ic or ic_ prefix
═══════════════════════════════════════════════════════════════

FOR EACH CURSOR WITHOUT ic_ PREFIX:

CURSOR FIX: {Class}.{feature} cursor {cursor}

STEP 1: Identify the problem
  Current: across {structure} as {cursor}
  Problem: {no_prefix | shadows_feature}

  If shadows feature:
  - CRITICAL: This is a VOIT(2) compile error
  - Feature shadowed: {feature_name}

STEP 2: Determine new cursor name
  Simple cursor: ic
  Named cursor: ic_{descriptive_name}

  Examples:
    across items as item → across items as ic
    across customers as c → across customers as ic
    across orders as order → across orders as ic_order

STEP 3: Update cursor and all uses within loop
  File: {file}

  BEFORE:
  across {structure} as {cursor} loop
      process ({cursor}.item)
      if {cursor}.item.is_valid then
          ...
      end
  end

  AFTER:
  across {structure} as ic loop
      process (ic.item)
      if ic.item.is_valid then
          ...
      end
  end

STEP 4: Handle nested loops
  For nested loops, use different cursor names:

  across outer_list as ic_outer loop
      across inner_list as ic_inner loop
          process (ic_outer.item, ic_inner.item)
      end
  end

═══════════════════════════════════════════════════════════════
BATCH OPERATIONS
═══════════════════════════════════════════════════════════════

BATCH 1: Add a_ prefix to all arguments

  This is often safe to batch because:
  - Argument scope is limited to feature
  - Pattern is consistent

  Find pattern: (\w+): (\w+) in feature signatures
  Replace pattern: a_$1: $2

  CAUTION: Manual review needed for:
  - Arguments already with a_ prefix
  - Arguments in inherited features (check parent)

BATCH 2: Fix all cursor names to ic

  Find pattern: across .* as (\w+) loop
  Replace pattern: across ... as ic loop

  Also update: $1.item → ic.item within loop body

  CAUTION: Nested loops need ic_outer, ic_inner

═══════════════════════════════════════════════════════════════
COMMON PATTERNS
═══════════════════════════════════════════════════════════════

Arguments:
  name → a_name
  value → a_value
  index → a_index
  other → a_other
  fileName → a_file_name (also fix CamelCase)

Locals (when shadowing):
  result → l_result
  item → l_item
  count → l_count

Cursors:
  item → ic
  element → ic
  customer → ic (or ic_customer in complex loops)
  i → ic
  c → ic

═══════════════════════════════════════════════════════════════
OUTPUT: ARGUMENT/LOCAL/CURSOR FIX LOG
═══════════════════════════════════════════════════════════════

# ARGUMENT/LOCAL/CURSOR FIXES: {library_name}

## Arguments Fixed
| Class | Feature | Old | New | Shadowed? |
|-------|---------|-----|-----|-----------|
| {cls} | {feat} | {old} | a_{new} | {yes/no} |

## Locals Fixed
| Class | Feature | Old | New | Reason |
|-------|---------|-----|-----|--------|
| {cls} | {feat} | {old} | {new} | {shadow/CamelCase} |

## Cursors Fixed
| Class | Feature | Old | New | Was VOIT(2)? |
|-------|---------|-----|-----|--------------|
| {cls} | {feat} | {old} | ic | {yes/no} |

## Compilation Status
- After fixes: {PASS/FAIL}

## VOIT(2) Errors Resolved
{count} cursor shadowing conflicts fixed
```

## Success Criteria
- All arguments have a_ prefix
- No locals shadow attributes/features
- All cursors use ic or ic_ prefix
- No VOIT(2) errors
- Library compiles

## Next Step
→ N06-FIX-CONTRACTS.md
