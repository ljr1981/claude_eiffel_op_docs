# 03: Write Preconditions

## Context
Working Hat: **SPECIFICATION** (Phase 1 of 4)

## Input Required
- Class skeletons from 02-DEFINE-CLASS-STRUCTURE.md
- Constraints from 01-ANALYZE-REQUIREMENTS.md

## Prompt

```
TASK: Add preconditions (require clauses) to each feature.

INPUT:
{class_skeletons}
{constraints}

OUTPUT FORMAT (per feature):
feature_name (arg1: TYPE1; arg2: TYPE2)
    require
        tag_name: boolean_expression
        another_tag: another_expression
    do
        -- TBD
    end

PRECONDITION CATEGORIES - check each:
1. ARGUMENT VALIDITY
   - not_void: arg /= Void (for detachable types)
   - positive: arg > 0 (for counts, indices)
   - valid_range: arg >= min and arg <= max
   - not_empty: not arg.is_empty (for strings, collections)

2. STATE VALIDITY
   - ready: is_initialized
   - not_closed: not is_closed
   - has_capacity: count < capacity

3. RELATIONSHIP VALIDITY
   - exists: has_item (key)
   - not_duplicate: not has_item (key)

RULES:
- Every precondition MUST have a tag (name)
- Tags should be readable: valid_index NOT v_idx
- Only reference queries, not commands
- If no preconditions needed, omit require block entirely
- Weaker is better: don't over-constrain
```

## Success Criteria
- All features analyzed for preconditions
- Every precondition tagged
- No unnecessary constraints added
- Argument validity covered

## Next Step
→ 04-WRITE-POSTCONDITIONS.md
