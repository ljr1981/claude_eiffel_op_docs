# N06: Fix Contract Tag Names

## Context
Phase: **FIX** (4 of 4)
Purpose: Standardize contract assertion tag names

## Mindset

**Contract tags are documentation. They explain WHY the assertion matters.**

A tag like `valid` tells nothing. A tag like `index_in_range` tells the reader exactly what's being checked and why it might fail.

## Input Required
- Classification report from N02
- List of contract tag fixes

## Prompt

```
TASK: Fix contract assertion tag names to follow Eiffel conventions.

INPUT:
{classification_from_N02}
Contract tag fixes needed:
{list of contract tag issues}

═══════════════════════════════════════════════════════════════
CONTRACT TAG CONVENTIONS
═══════════════════════════════════════════════════════════════

PRECONDITIONS (require):
| Pattern | Use For | Example |
|---------|---------|---------|
| X_valid | Validation | index_valid: valid_index (a_index) |
| X_not_empty | Non-empty | name_not_empty: not a_name.is_empty |
| X_not_void | Non-void | item_not_void: a_item /= Void |
| X_positive | Positive number | count_positive: a_count > 0 |
| X_not_negative | Non-negative | amount_not_negative: a_amount >= 0 |
| X_in_range | Range check | index_in_range: a_index >= 1 and a_index <= count |
| has_X | State present | has_connection: is_connected |
| not_X | State absent | not_closed: not is_closed |
| X_exists | Existence | file_exists: a_path.exists |

POSTCONDITIONS (ensure):
| Pattern | Use For | Example |
|---------|---------|---------|
| X_set | Assignment | name_set: name = a_name |
| X_assigned | Assignment | result_assigned: Result /= Void |
| X_increased | Increment | count_increased: count = old count + 1 |
| X_decreased | Decrement | balance_decreased: balance = old balance - a_amount |
| X_unchanged | No change | count_unchanged: count = old count |
| X_added | Element added | item_added: has (a_item) |
| X_removed | Element removed | item_removed: not has (a_item) |
| result_X | Result property | result_positive: Result > 0 |
| result_not_void | Non-void result | result_not_void: Result /= Void |

INVARIANTS:
| Pattern | Use For | Example |
|---------|---------|---------|
| X_valid | Always valid | count_valid: count >= 0 |
| X_not_negative | Non-negative | balance_not_negative: balance >= 0 |
| X_consistent | Related values | bounds_consistent: lower <= upper |
| X_implies_Y | Logical relation | empty_means_zero: is_empty implies count = 0 |

═══════════════════════════════════════════════════════════════
FOR EACH CONTRACT TAG FIX
═══════════════════════════════════════════════════════════════

CONTRACT TAG FIX: {Class}.{feature} {require|ensure|invariant}

STEP 1: Identify the problem
  Current tag: {current_tag}
  Expression: {expression}
  Problem: {missing_tag | non_descriptive | wrong_pattern}

STEP 2: Analyze the assertion
  What is being checked?
  - Subject: {what entity}
  - Property: {what property}
  - Condition: {what must be true}

STEP 3: Choose appropriate tag pattern
  Based on assertion type:
  - Validation → X_valid
  - Non-empty → X_not_empty
  - Range → X_in_range
  - State → has_X / not_X
  - Assignment → X_set
  - Delta → X_increased / X_decreased

STEP 4: Apply the fix
  File: {file}
  Line: {line}

  BEFORE:
  require
      {old_tag}: {expression}

  AFTER:
  require
      {new_tag}: {expression}

═══════════════════════════════════════════════════════════════
COMMON TAG FIXES
═══════════════════════════════════════════════════════════════

Problem: No tag (just expression)
  BEFORE:
  require
      a_name /= Void

  AFTER:
  require
      name_not_void: a_name /= Void

Problem: Generic tag
  BEFORE:
  require
      valid: a_index >= 1 and a_index <= count

  AFTER:
  require
      index_in_range: a_index >= 1 and a_index <= count

Problem: Tag doesn't match expression
  BEFORE:
  require
      positive: a_amount >= 0  -- Actually checking non-negative!

  AFTER:
  require
      amount_not_negative: a_amount >= 0

Problem: CamelCase tag
  BEFORE:
  require
      nameNotEmpty: not a_name.is_empty

  AFTER:
  require
      name_not_empty: not a_name.is_empty

Problem: Tag too long/verbose
  BEFORE:
  require
      the_index_parameter_must_be_within_valid_bounds: valid_index (a_index)

  AFTER:
  require
      index_valid: valid_index (a_index)

═══════════════════════════════════════════════════════════════
FEATURE CLAUSE LABEL FIXES
═══════════════════════════════════════════════════════════════

While fixing contracts, also standardize feature clause labels:

Standard Labels:
| Label | Contents |
|-------|----------|
| Initialization | Creation procedures |
| Access | Public queries returning values |
| Measurement | count, capacity, size queries |
| Status report | Boolean status queries (is_empty, is_valid) |
| Comparison | Comparison features |
| Element change | Commands that modify elements |
| Removal | Commands that remove elements |
| Transformation | Commands that transform state |
| Conversion | to_X, as_X features |
| Duplication | copy, twin features |
| Basic operations | Core functionality |
| Implementation | Private helpers |

CLAUSE LABEL FIX:

  BEFORE:
  feature -- Queries

  AFTER:
  feature -- Access

  BEFORE:
  feature -- Setters

  AFTER:
  feature -- Element change

  BEFORE:
  feature {NONE} -- Private

  AFTER:
  feature {NONE} -- Implementation

═══════════════════════════════════════════════════════════════
BATCH OPERATIONS
═══════════════════════════════════════════════════════════════

Some contract tag fixes can be batched:

BATCH: Add tags to untagged assertions
  Find: require\n\s+([^:]+)$  (assertion without colon)
  Fix: Add appropriate tag based on expression analysis

BATCH: Fix CamelCase tags
  Find: ([A-Z][a-z]+)+:
  Replace: Convert to snake_case

Note: Contract tag fixes have no compile impact but improve readability.

═══════════════════════════════════════════════════════════════
OUTPUT: CONTRACT TAG FIX LOG
═══════════════════════════════════════════════════════════════

# CONTRACT TAG FIXES: {library_name}

## Precondition Tags Fixed
| Class | Feature | Old Tag | New Tag |
|-------|---------|---------|---------|
| {cls} | {feat} | {old} | {new} |

## Postcondition Tags Fixed
| Class | Feature | Old Tag | New Tag |
|-------|---------|---------|---------|
| {cls} | {feat} | {old} | {new} |

## Invariant Tags Fixed
| Class | Old Tag | New Tag |
|-------|---------|---------|
| {cls} | {old} | {new} |

## Feature Clause Labels Fixed
| Class | Old Label | New Label |
|-------|-----------|-----------|
| {cls} | {old} | {new} |

## Summary
- Precondition tags fixed: {n}
- Postcondition tags fixed: {n}
- Invariant tags fixed: {n}
- Clause labels fixed: {n}
```

## Success Criteria
- All contract assertions have descriptive tags
- Tags follow naming patterns
- Feature clause labels are standard
- No CamelCase in tags

## Next Step
→ N07-VERIFY-NAMING.md
