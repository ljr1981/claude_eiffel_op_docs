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

═══════════════════════════════════════════════════════════════
MODEL-BASED POSTCONDITIONS (PREFERRED - complete specification)
═══════════════════════════════════════════════════════════════

If class has a model query, use MODEL-BASED postconditions:

For COMMANDS that modify state:
   - model_extended: model |=| old model & x        -- append
   - model_reduced: model |=| old model / x         -- remove one
   - model_cleared: model.is_empty                  -- clear all
   - model_updated: model |=| old model.replaced_at (i, x)

For QUERIES that access state:
   - model_consistent: Result = model [i]           -- indexed access
   - model_contains: Result = model.has (x)         -- membership
   - model_count: Result = model.count              -- size

WHY MODEL-BASED: "model |=| old model & x" specifies the COMPLETE
transformation. Primitive "count = old count + 1" only checks ONE property.

═══════════════════════════════════════════════════════════════
PRIMITIVE POSTCONDITIONS (use when no model applies)
═══════════════════════════════════════════════════════════════

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
- **PREFER model-based postconditions when class has model query**
- Use 'old' keyword for before-values
- Use 'Result' for return value
- Use '|=|' for model equality comparison
- Queries: focus on Result properties OR model consistency
- Commands: focus on model transformation OR state changes
- Be specific: "model |=| old model & x" NOT "model changed"
- Every postcondition MUST be verifiable
```

## Success Criteria
- All features have postconditions
- Commands specify state changes
- Queries specify Result properties
- 'old' used correctly for delta checks

## Next Step
→ 05-WRITE-INVARIANTS.md
