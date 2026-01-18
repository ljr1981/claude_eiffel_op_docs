# 04: Write Postconditions

## Context
Working Hat: **SPECIFICATION** (Phase 1 of 4)

## Input Required
- Features with preconditions from 03-WRITE-PRECONDITIONS.md

## Prompt

```
TASK: Add postconditions (ensure clauses) to each feature.

INPUT:
{features_with_preconditions}

OUTPUT FORMAT (per feature):
feature_name (arg: TYPE): RESULT_TYPE
    require
        -- existing preconditions
    do
        -- TBD
    ensure
        tag_name: boolean_expression
        uses_old: attribute = old attribute + delta
    end

POSTCONDITION CATEGORIES - check each:
1. RESULT CONDITIONS (for queries)
   - result_valid: Result >= 0
   - result_consistent: Result = some_query
   - result_not_void: Result /= Void

2. STATE CHANGES (for commands)
   - attribute_set: attribute = expected_value
   - count_changed: count = old count + 1
   - item_added: has_item (new_item)

3. FRAME CONDITIONS (what didn't change)
   - other_unchanged: other_attr = old other_attr
   - count_unchanged: count = old count

4. RELATIONSHIP TO INPUT
   - reflects_argument: stored_value = arg
   - contains_input: has_substring (arg)

RULES:
- Use 'old' keyword for before-values
- Use 'Result' for return value
- Queries: focus on Result properties
- Commands: focus on state changes
- Be specific: "count = old count + 1" NOT "count changed"
- Every postcondition MUST be verifiable
```

## Success Criteria
- All features have postconditions
- Commands specify state changes
- Queries specify Result properties
- 'old' used correctly for delta checks

## Next Step
→ 05-WRITE-INVARIANTS.md
